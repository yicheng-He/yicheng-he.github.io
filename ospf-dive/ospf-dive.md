---
theme: seriph
layout: cover
background: https://source.unsplash.com/1600x900/?nature,water
---
# OSPF Deep Dive
Presentator: Tom He

---
## Agenda

- OSPF Theory and Terminology
- Designated Routers
- Neighbor Formation
- Network Types
- LSA/Area/Route Types
- Demo

---

### OSPF Fundamentals

- OSPF is an open standard
- OSPF is a Link State routing protocol
- Every router has a "map" of the network, called a Link State Database
- OSPF runs the Dijkstra Algorithm to determine the shortest path to a network

---

## OSPF Theory and Terminology

- Router: A level three Internet Protocol packet switch. Formerly called a gateway in much of the IP literature.
- Autonomous System: A group of routers exchanging routing information via a common routing protocol.          
- Neighboring routers: Two routers that have interfaces to a common network. Neighbor relationships are maintained by, and usually
dynamically discovered by, OSPF's Hello Protocol.
- Adjacency: A relationship formed between selected neighboring routers for the purpose of exchanging routing information.  Not
every pair of neighboring routers become adjacent.
- Link state advertisement: Unit of data describing the local state of a router or network. For a router, this includes the state of the
router's interfaces and adjacencies.  Each link state advertisement is flooded throughout the routing domain. The collected link state advertisements of all routers and networks forms the protocol's link state database.        
- Designated Router:
The Designated Router generates an LSA for the network and has other special responsibilities in the running of the protocol. The Designated Router is elected by the Hello Protocol.
The Designated Router concept enables a reduction in the number of adjacencies required on a broadcast or NBMA network.  This in turn reduces the amount of routing protocol traffic and the size of the link-state database.

---

## Designated Routers

On the broadcast segment, there are n*(n-1)/2 neighbor relations, it will be a huge amount of Link State Updates and Acknowledgements sent over the subnet if the OSPF router will try to synchronize with each OSPF router on the subnet.

<img src="/resource/Ospf-adjacency.jpg" class="h-60"/>

This problem is solved by electing one **Designated Router** and one **Backup Designated Router** for each broadcast subnet. All other routers are synchronizing and forming adjacencies only with those two elected routers. This approach reduces the number of adjacencies from n*(n-1)/2 to only 2n-3.

---

## OSPF Neighbor Relationships

OSPF is a link-state protocol that assumes that the interface of the router is considered an OSPF link. Whenever OSPF is started, it adds the state of all the links in the local **link-state database**.

There are several steps before the OSPF network becomes fully functional:

- Neighbors discovery
- Database Synchronization
- Routing calculation

---

## OSPF Packets

- OSPF operates over the IP network layer using protocol number 89.

- A destination IP address is set to the neighbor's IP address or to one of the OSPF multicast addresses AllSPFRouters (224.0.0.5) or AllDRRouters (224.0.0.6). 

- Every OSPF packet begins with a standard 24-byte header.

--- 

<img src="/resource/OSPF_Header.png" class="h-40"/>

  | Field                     | Description                                                  |
  | ------------------------- | ------------------------------------------------------------ |
  | **Packet type**           | There are several types of OSPF packets: Hello packet, Database Description (DD) packet, Link state request packet, Link State Update packet, and Link State Acknowledgement packet. All of these packets except the Hello packet are used in link-state database synchronization. |
  | **Router ID**             | one of the router's IP addresses unless configured manually  |
  | **Area ID**               | Allows OSPF router to associate the packet to the proper OSPF area. |
  | **Checksum**              | Allows receiving router to determine if a packet was damaged in transit. |
  | **Authentication fields** | These fields allow the receiving router to verify that the packet's contents were not modified and that packet really came from the OSPF router whose Router ID appears in the packet. |

--- 

  There are five different OSPF packet types used to ensure proper LSA flooding over the OSPF network.

  - **Hello packet** - used to discover OSPF neighbors and build adjacencies.
  - **Database Description (DD)** - check for Database synchronization between routers. Exchanged after adjacencies are built.
  - **Link-State Request (LSR)** - used to request up-to-date pieces of the neighbor's database. Out-of-date parts of the routing database are determined after the DD exchange.
  - **Link-State Update (LSU)** - carries a collection of specifically requested link-state records.
  - **Link-State Acknowledgment (LSack)** - is used to acknowledge other packet types that way introducing reliable communication.

---

### OSPF Neighbor States
- Down
  - Hello packets have not been received by any neighbor
  - If the dead interval timer expires
- Attempt - Applies to only NBMA network types
- Init 
  - Hello packets are now being sent/received
  - Receiving router ID is not included in the hello packet
- 2-way - Indicates bi-directional communication is established
- Exstart - Master/slave relationship is established based on highest RID
- Exchange - Database descriptors are exchanged which contain the LSA headers
- Loading - LSR and LSU's are transmitted to propagate the LSDB
- Full

---

<img src="/resource/OSPF_neighbor_states.png"/>

---

## OSPF Areas

Backbone Area (Area 0) = Hub
<img src="/resource/OSPF_AREAS.png" class="h-80"/>

--- 

## OSPF Network Types
- Broadcast
  - DR/BDR election process
  - Discovery hellos sent to 224.0.0.5/DR traffic uses 244.0.0.6
- Non-Broadcast
  - DR/BDR election process
  - Requires neighbor command to form adjacency
  - Requires full mesh between OSPF neighbors
- Point-to-Point
  - Default on Serial and NBMA network types
  - Njo DR/BDR election
  - Hello packets sent to 224.0.0.5
- Point-to-Multipoint
  - Neighbors advertise next hop as a host route
  - No DR/BDR election
--- 

| Network Type                      | Hold Time | Dead Interval |
| --------------------------------- | --------- | ------------- |
| Broadcast                         | 10        | 40            |
| Non-Broadcast                     | 30        | 120           |
| Point-to-Point                    | 10        | 40            |
| Point-to-Multipoint               | 30        | 120           |
| Point-to-Multipoint Non-Broadcast | 30        | 120           |

---

## OSPF LSA Types

- <font color = black >LSA Type 1: Route LSA </font>
  - Sent by every OSPF enabled device within an area
  - Advertises its presence on the LAN segment within an area
  - Contains directly connected links on the device and the link type
  - Type 1 LSA's include the link-state ID as the RID of the advertising router
- <font color = green>LSA Type 2: Network LSA </font>
  - Generated by the Designated Router on a multi-access network, within an area
  - Contains a list of all the devices connected to the segment
  - The link-state ID of the type 2 LSA is the IP address of the DR
- <font color = orange>LSA Type 3: Summary LSA</font>
  -  Generated by the ABR of each area and sent into the backbone area
  - Summarizes detailed are information found in type 1 and type 2 LSA's
  - Summary LSA's use the network address as the link-state ID
  - Listed as O IA routes in the RIB

<img src="/resource/LSA_types.png" class="h-40"/>

--- 

- LSA Type 4: ASBR Summary
  - Generated by the ABR and is flooded within an area, to all areas
  - Contains information about the ASBR and sets the next-hop to the area ABR
  - The link-state ID is set to the RID of the ASBR
- LSA Type 5: External
  - Generated by the ASBR
  - Flooded throughout the entire OSPF domain unaltered (except NSSA & Stub areas)
  - Contains a list of all redistributed routes
  - Listed as E1/E2 routes in the RIB
- LSA Type 7: NSSA
  - Equivalent to type 5 LSA's and are used in a NSSA area
  - Type 7 LSA's are translated to type 5 LSA's as the ABR of an area

---

## OSPF Route Types

| LSA    | RIB               |
| ------ | ----------------- |
| Type 1 | O, = Intra-area   |
| Type 3 | OIA, = Inter-area |
| Type 5 | E1, E2            |
| Type 7 | N1, N2 (NSSA)     |

---

## OSPF Path Preference
1. Most specific route
2. Route type: O, OIA, E1, E2, N1, N2
3. Metric: BW, Cost

---

## Demo

Tools: GNS3

Topology:

<img src="/resource/OSPF_demo_topo.jpg" class="h-80"/>


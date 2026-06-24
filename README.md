# High-Availability Multi-Service Enterprise Network

A production-grade, highly available enterprise network architecture designed to eliminate single points of failure (SPOFs). This infrastructure leverages a collapsed core/distribution layer featuring robust Layer 2/3 redundancy, dynamic routing, enterprise security boundaries, and segmented environments for multi-service traffic (Voice, Data, IoT, and Centralized Server Services).

## 🏗️ Project Origins & Architecture Context

This infrastructure design is modeled closely after a production enterprise campus network observed during an industry internship. While experiencing the day-to-day operations of a corporate environment—specifically focusing on administrative mapping and automated workflows—I analyzed and reverse-engineered the core networking topology to study how modern mid-sized enterprises balance high-availability, scalability, and strict security segmentation.

The design implements a high-availability triangle topology at the distribution and core layers to ensure near-zero downtime. It is split into four distinct, functional departments (VLANs), an isolated server farm infrastructure, and a secure edge WAN connection.

![Network Topology](images/topology.png)

---

## 🔑 Technical Stack & Implemented Protocols

* **Layer 3 Routing:** Single-Area OSPF (Area 0) for sub-second route convergence, optimal path selection, and scalable interior gateway routing.
* **Gateway Redundancy:** HSRP (Hot Standby Router Protocol) providing seamless first-hop virtual gateway redundancy across all end-user subnets.
* **Link Aggregation:** LACP EtherChannel for backbone bandwidth aggregation and link-level failover between distribution switches.
* **Logical Segmentation:** IEEE 802.1Q VLAN tagging and Inter-VLAN routing configured via Switch Virtual Interfaces (SVIs).
* **Core Infrastructure Services:** Centralized DHCP Relay (`ip helper-address`) architecture routing dynamic allocations across routed layer-3 boundaries.
* **Edge Isolation:** Static Network Address Translation (NAT/PAT) and perimeter routing separating internal infrastructure from the ISP.

---

## 📂 Network Segmentation (VLAN Design)

The campus network is segmented into isolated logical broadcast domains to optimize traffic patterns, contain broadcasts, and apply granular security policies:

| VLAN ID | Department / Segment | Primary Elements | Gateway / HSRP VIP |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | Corporate Office / Data | PC1, Laptop2, Network Printer, IP Phone | `192.168.10.1` |
| **VLAN 20** | Engineering & Workstations | PC2, PC3, PC4, Laptops, Mobile Devices | `192.168.20.1` |
| **VLAN 30** | Centralized Server Farm | Enterprise Application & Storage Servers | `192.168.30.1` |
| **VLAN 60** | IoT & Smart Infrastructure | RFID Readers, Smart Doors, Wireless Access Points | `192.168.60.1` |

---

## 🛠️ Configuration & Implementation Highlights

### 1. High Availability Gateway Redundancy (HSRP)
Configured on core SVIs to ensure that if the primary distribution switch goes down, the standby switch takes over the virtual IP gateway seamlessly without client packet loss or manual intervention.

```text
! Configuration sample for Core Switch A (Primary for VLAN 10)
interface Vlan10
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 150
 standby 10 preempt

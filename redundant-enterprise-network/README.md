```markdown
# High-Availability Multi-Service Enterprise Network

A production-grade, highly available enterprise network architecture designed to eliminate single points of failure (SPOFs). This infrastructure leverages a collapsed core/distribution layer featuring robust Layer 2/3 redundancy, dynamic routing, enterprise security boundaries, and segmented environments for multi-service traffic (Voice, Data, IoT, and Centralized Server Services).

## 🏗️ Project Origins & Architecture Context

This infrastructure design is modeled closely after a production enterprise campus network observed during an industry internship. While experiencing the day-to-day operations of a corporate environment—specifically focusing on administrative mapping and automated workflows—I analyzed and reverse-engineered the core networking topology to study how modern mid-sized enterprises balance high-availability, scalability, and strict security segmentation.

The design implements a high-availability triangle topology at the distribution and core layers to ensure near-zero downtime. It is split into distinct, functional departments (VLANs), an isolated server farm infrastructure, and a secure edge WAN connection.

![Network Topology](./images/topology.png)

---

## 🔑 Technical Stack & Implemented Protocols

* **Layer 3 Routing:** Single-Area OSPF (Area 0) for sub-second route convergence, optimal path selection, and scalable interior gateway routing.
* **Gateway Redundancy:** HSRP (Hot Standby Router Protocol) providing seamless first-hop virtual gateway redundancy across all end-user subnets.
* **Link Aggregation:** LACP EtherChannel for backbone bandwidth aggregation and link-level failover between distribution switches.
* **Logical Segmentation:** IEEE 802.1Q VLAN tagging and auxiliary Voice VLAN mapping to separate deterministic real-time voice traffic from standard data planes.
* **Core Infrastructure Services:** Centralized DHCP Relay (`ip helper-address`) architecture routing dynamic allocations across routed layer-3 boundaries.
* **Edge Isolation:** Static Network Address Translation (NAT/PAT) and perimeter routing separating internal infrastructure from the ISP.

---

## 📂 Network Segmentation (VLAN Design)

The campus network is segmented into isolated logical broadcast domains to optimize traffic patterns, contain broadcasts, and apply granular security policies:

| VLAN ID | Department / Segment | Primary Elements | Gateway / HSRP VIP |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | Corporate Office / Data | PC1, Laptop2, Network Printer | `192.168.10.1` |
| **VLAN 20** | Engineering & Workstations | PC2, PC3, PC4, Laptops, Mobile Devices | `192.168.20.1` |
| **VLAN 30** | Centralized Server Farm | Enterprise Application & Storage Servers | `192.168.30.1` |
| **VLAN 60** | IoT & Smart Infrastructure | RFID Readers, Smart Doors, Wireless Access Points | `192.168.60.1` |
| **VLAN 70** | IP Voice Infrastructure | Cisco 7960 IP Phones (Auxiliary/Voice VLAN) | `192.168.70.1` |

---

## 🛠️ Configuration & Implementation Highlights

### 1. Auxiliary Voice VLAN Configuration
To guarantee packet prioritization for enterprise voice collaboration, access-layer ports are configured to separate standard untagged data traffic from tagged voice traffic on the same physical drop.

```text
! Access Switch Port Configuration hosting a PC and an IP Phone
interface FastEthernet 0/4
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 70
 spanning-tree portfast

```

### 2. High Availability Gateway Redundancy (HSRP)

Configured on core SVIs to ensure that if the primary distribution switch goes down, the standby switch takes over the virtual IP gateway seamlessly without dropping ongoing voice calls or active data sessions.

```text
! Configuration sample for Core Switch A (Primary for Voice Gateway)
interface Vlan70
 ip address 192.168.70.2 255.255.255.0
 standby 70 ip 192.168.70.1
 standby 70 priority 150
 standby 70 preempt

```

### 3. Dynamic Core Routing (OSPF)

Configured on Core Switches and the Edge Router (`Router0`) to automatically propagate internal networks, adapt to link failures, and compute optimal data paths.

```text
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.60.0 0.0.0.255 area 0
 network 192.168.70.0 0.0.0.255 area 0

```

---

## ✅ Verification & Validation Commands

The stability, routing tables, and failover mechanics of this infrastructure are verified via the following Cisco IOS CLI commands:

* **Verify Neighbor Adjacencies:** `show ip ospf neighbor` validates that full `DR/BDR` synchronization is established between Layer 3 switches and the perimeter router.
* **Verify HSRP State:** `show standby brief` monitors active/standby designations and verifies preempt operations during simulated link failures.
* **Verify Link Bundling:** `show etherchannel summary` ensures the logical interface displays a state of `SU` (Layer 2/In Use) and ports are marked as `P` (Bundled in Port-Channel).
* **Verify End-to-End Connectivity:** Extended `ping` and `traceroute` executions validate data plane reachability from isolated access-layer VLANs up to the simulated ISP edge (`Router2`).

---

## 📂 Repository Layout

You can explore the underlying scripts, configurations, and topology files using the links below:

* [Configs Directory](https://www.google.com/search?q=./configs): Contains complete production running configurations for the Edge Router, ISP platform, and Core switches.
* [Images Directory](https://www.google.com/search?q=./images): Topology diagrams, flowcharts, and system metrics documentation.
* [Cisco Packet Tracer File](https://www.google.com/search?q=./enterprise_topology.pkt): Packet Tracer master environment file *(Designed for Cisco Packet Tracer v8.2+)*.

```

```

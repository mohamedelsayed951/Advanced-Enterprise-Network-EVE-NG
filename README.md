# Multi-AS Enterprise Infrastructure: BGP, OSPF, EIGRP, MPLS Core & FortiGate IPsec VPN

A comprehensive, deployment-ready documentation for simulating a multi-tenant Service Provider and Enterprise-grade infrastructure within EVE-NG. This architecture validates boundary routing dynamics, unified label switching, and firewall security matrices across three independent Autonomous Systems.

---

## 🌐 1. Architectural Overview & Design
The infrastructure simulates three distinct Service Providers (SPs) interconnected via external border gateway interfaces, alongside distinct interior gateway protocol (IGP) frameworks:

- SP1 (AS 65001): Deploys Multi-Area OSPF (Area 0 & Area 1) across R1, R2, R3, and R4. R2 functions as the Area Border Router (ABR).
- SP2 (AS 65002): Deploys EIGRP Autonomous System 100 across R5, R6, and R7.
- SP3 (AS 65003): Implements an MPLS/LDP Core Network integrated with dual interior structures running OSPF and EIGRP 200, utilizing mutual redistribution on node R10.
- Security Matrix: Network edges are protected via virtualized FortiGate Next-Generation Firewalls (v7.6.2) processing secure data planes over IPsec VPN tunnels.

### Boundary Configuration Rules
- eBGP Peering: Established solely over the direct physical links connecting border nodes R1, R5, and R8.
- iBGP Mesh: Full-Mesh iBGP structure established internally within each autonomous domain using stable /32 Loopback addresses.
- Redistribution Strategy: Absent across BGP and internal IGPs to prevent core routing injection anomalies, with a strict exception on R10 executing mutual redistribution between OSPF and EIGRP 200 within SP3.

---

## 🛠️ 2. Step-by-Step Implementation Sequence

To ensure stable convergence without routing loops, execution must strictly follow this order:
1. Layer 3 Interface Mapping: Assign IP addressing schemas to physical and Loopback interfaces on all routers.
2. IGP Initialization: Enable OSPF/EIGRP within each respective autonomous block and verify neighborhood adjacencies.
3. MPLS Backbone Activation: Enable LDP label switching across the SP3 infrastructure.
4. iBGP Full-Mesh Peering: Bind internal BGP neighbors over converged loopback systems.
5. eBGP Boundary Peering: Initialize exterior peering across physical mesh links (R1 - R5 - R8).
6. Network Advertisement & Verification: Advertise edge WAN blocks via BGP and execute end-to-end trace validations.
7. Access-Layer Setup: Provision downstream switching components, VTP server structures, and Router-on-a-Stick sub-interfaces.
8. Security Enforcement: Hardening parameters, FortiGate CLI provisioning, and structural IPsec Phase 1/2 deployment.

---

## 📊 3. Network Addressing & Interface Layout

### Router Loopback 0 Interfaces (Subnet Mask: /32)

| Router Node | Provider Domain | Loopback 0 Address |
| :--- | :--- | :--- |
| R1 (ASBR) | SP1 (AS 65001) | 1.1.4.1 |
| R2 (ABR)  | SP1 (AS 65001) | 1.1.3.1 |
| R3 (Edge)  | SP1 (AS 65001) | 1.1.1.1 |
| R4 (Core)  | SP1 (AS 65001) | 1.1.2.1 |
| R5 (ASBR) | SP2 (AS 65002) | 3.3.1.1 |
| R6 (Core)  | SP2 (AS 65002) | 3.3.2.1 |
| R7 (Edge)  | SP2 (AS 65002) | 3.3.3.1 |
| R8 (ASBR) | SP3 (AS 65003) | 2.2.2.1 |
| R9 (P)     | SP3 (AS 65003) | 2.2.1.1 |
| R10 (P/Redist)| SP3 (AS 65003) | 2.2.3.1 |
| R11 (PE)    | SP3 (AS 65003) | 2.2.4.1 |

### Internal Topology Connectors

| Link Path | Addressing Schema | Active Routing Protocol |
| :--- | :--- | :--- |
| R1 fa0/0 <-> R2 fa1/0 | 10.0.0.0/29 | OSPF Area 0 |
| R2 fa0/0 <-> R4 fa1/0 | 10.0.1.0/29 | OSPF Area 1 |
| R4 fa0/0 <-> R3 fa1/0 | 10.0.2.0/29 | OSPF Area 1 |
| R3 fa0/0 <-> FW1 port2 | 10.0.3.0/24 | WAN Edge (BGP Advertisement Only) |
| R5 fa0/0 <-> R6 fa1/0 | 172.16.1.0/29 | EIGRP Autonomous System 100 |
| R6 fa0/0 <-> R7 fa1/0 | 172.16.2.0/29 | EIGRP Autonomous System 100 |
| R7 fa0/0 <-> FW2 port2 | 172.16.3.0/24 | WAN Edge (BGP Advertisement Only) |
| R8 fa0/0 <-> R9 fa1/0 | 192.168.4.0/29 | OSPF Area 0 + MPLS IP Core |
| R9 fa0/0 <-> R10 fa1/0 | 192.168.1.0/24 | OSPF Area 0 + MPLS IP Core |
| R10 fa0/0 <-> R11 fa1/0 | 192.168.2.0/24 | EIGRP 200 + MPLS IP Core |
| R11 fa0/0 <-> SW10 e0/3 | Trunking (No IP) | Router-on-a-Stick (VLANs 5, 10, 100) |

### Inter-AS Core eBGP Fabrics
- R1 fa1/0 <-> R5 fa3/0: 10.1.2.0/29
- R1 fa3/0 <-> R8 fa3/0: 10.1.3.0/29
- R5 fa1/0 <-> R8 fa1/0: 10.1.1.0/29

---

## 💻 4. Core Router Configuration Baselines

### SP1 Domain: Router R1 Sample Baseline
enable
configure terminal
hostname R1
no ip domain-lookup
!
interface Loopback0
 ip address 1.1.4.1 255.255.255.255
!
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.248
 no shutdown
!
interface FastEthernet1/0
 ip address 10.1.2.1 255.255.255.248
 no shutdown
!
interface FastEthernet3/0
 ip address 10.1.3.1 255.255.255.248
 no shutdown
!
router ospf 1
 router-id 1.1.4.1
 network 1.1.4.1 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.7 area 0
!
router bgp 65001
 bgp router-id 1.1.4.1
 neighbor 1.1.3.1 remote-as 65001
 neighbor 1.1.3.1 update-source Loopback0
 neighbor 1.1.3.1 next-hop-self
 neighbor 1.1.2.1 remote-as 65001
 neighbor 1.1.2.1 update-source Loopback0
 neighbor 1.1.2.1 next-hop-self
 neighbor 1.1.1.1 remote-as 65001
 neighbor 1.1.1.1 update-source Loopback0
 neighbor 1.1.1.1 next-hop-self
end
write memory
### SP3 Core Domain: Router R10 Mutual Redistribution
enable
configure terminal
hostname R10
no ip domain-lookup
ip cef
mpls label protocol ldp
mpls ldp router-id Loopback0 force
!
interface Loopback0
 ip address 2.2.3.1 255.255.255.255
!
interface FastEthernet1/0
 ip address 192.168.1.2 255.255.255.0
 mpls ip
 no shutdown
!
interface FastEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 mpls ip
 no shutdown
!
router ospf 1
 router-id 2.2.3.1
 network 2.2.3.1 0.0.0.0 area 0
 network 192.168.1.0 0.0.0.255 area 0
 redistribute eigrp 200 subnets
!
router eigrp 200
 eigrp router-id 2.2.3.1
 network 192.168.2.0 0.0.0.255
 redistribute ospf 1 metric 10000 100 255 1 1500
 no auto-summary
end
write memory
---

## 🎛️ 5. Access Layer Infrastructure (SP3 Block)

Downstream configurations integrate Layer-2 trunk aggregates, multi-chassis channel processing, and localized security parameters:

### Sub-Interface Gateway Allocations (R11)
- VLAN 5 (User Matrix): IP 192.168.5.1/24 -> encapsulation dot1Q 5
- VLAN 10 (User Matrix): IP 192.168.10.1/24 -> encapsulation dot1Q 10
- VLAN 100 (Native Management): IP 192.168.100.1/24 -> encapsulation dot1Q 100 native

### Switch Access Security Deployment Sample (SW3)
configure terminal
interface range e0/1 - 2
 switchport mode access
 switchport access vlan 5
 switchport nonegotiate
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
---

## 🛡️ 6. Security Perimeter (FortiGate Appliance Provisioning)

FortiGate firewall appliances handle border isolation using exact static assignment scripts via the integrated security terminal interface:

### Firewall Appliance FW1 (CLI Automation Profile)
config system global
    set hostname FW1
end
config system interface
    edit "port2"
        set mode static
        set ip 10.0.3.2 255.255.255.0
        set allowaccess ping https ssh
        set alias "WAN"
    next
    edit "port1"
        set mode static
        set ip 10.0.5.1 255.255.255.0
        set allowaccess ping
        set alias "LAN"
    next
end
config router static
    edit 1
        set gateway 10.0.3.1
        set device "port2"
    next
end
---

## 🔍 7. Verification & Diagnostic Runbooks
Execute these testing vectors sequentially to validate operational status:

### Interior Core Diagnostics
- show ip ospf neighbor -> Verify adjacent states converge to FULL.
- show ip mpls ldp neighbor -> Validate stable LDP sessions across internal SP3 boundaries.
- show mpls forwarding-table -> Verify label mapping strings are fully populated.

### Boundary Plane Diagnostics
- show ip bgp summary -> Verify both internal and external peer matrices show Established status.
- traceroute 2.2.2.1 source 2.2.4.1 -> Trace structural pathing to observe functional MPLS label tagging.
- execute ping 172.16.3.2 -> Execute from FortiGate CLI consoles to test data path connectivity across external networks.

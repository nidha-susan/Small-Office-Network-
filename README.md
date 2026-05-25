# Small Office Network — VLAN, Inter-VLAN Routing, DHCP & ACLs

A three-department small office network designed and configured in **Cisco Packet Tracer**. Demonstrates VLAN segmentation, inter-VLAN routing on a Layer 3 switch, centralized DHCP, and ACL-based traffic restriction between departments.


## Project overview

This lab simulates a small office with three departments : **Sales**, **Engineering**, and **HR** — each isolated in its own VLAN and subnet. A multilayer Cisco 3650 switch handles routing between the VLANs, hosts DHCP pools for all three subnets, and enforces an access control policy that prevents the Sales department from reaching Management resources.

### Key skills demonstrated

- VLAN design and configuration (Layer 2 segmentation)
- 802.1Q trunking between switches
- Inter-VLAN routing using SVIs on a Layer 3 switch
- Centralized DHCP with multiple pools and excluded address ranges
- Extended ACLs applied to SVIs for departmental traffic control
- Structured troubleshooting using `show` commands and `ping`


## Topology

| Device | Role | Model |
|--------|------|-------|
| L3-SW | Layer 3 switch — routing, DHCP, ACL enforcement | Cisco 3650-24PS |
| SW-Sales | Access switch — Sales VLAN | Cisco 2960-24TT |
| SW-Eng | Access switch — Engineering VLAN | Cisco 2960-24TT |
| SW-HR | Access switch — Management VLAN | Cisco 2960-24TT |
| PCs (6) | End hosts — 2 per department | Generic PC |

---

## IP addressing plan

| VLAN | Name | Subnet | Gateway (SVI) | DHCP range |
|------|------|--------|---------------|------------|
| 10 | SALES | 192.168.10.0/24 | 192.168.10.1 | .10 – .100 |
| 20 | ENGINEERING | 192.168.20.0/24 | 192.168.20.1 | .10 – .100 |
| 30 | HR | 192.168.30.0/24 | 192.168.30.1 | .10 – .100 |

Addresses .1 through .9 of each subnet are excluded from DHCP and reserved for the gateway and any future static assignments.

---

## Configuration highlights

### Layer 3 switch (L3-SW)

- Three VLANs created and corresponding SVIs configured as default gateways
- ip routing enabled for inter-VLAN routing
- Trunk interfaces use 802.1Q encapsulation with allowed VLAN list explicitly set
- DHCP pools **SALES_POOL**, **ENG_POOL**, **HR_POOL** serving all three subnets

### Access switches

- Each switch carries only its own local VLAN
- PC ports configured as access ports in the appropriate VLAN
- Uplink to L3-SW configured as 802.1Q trunk allowing VLANs 10, 20, 30

### Security — Sales to Management restriction

Extended ACL applied inbound on the Sales SVI:
access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 permit ip any any
interface vlan 10
ip access-group 100 in

Result: Sales PCs cannot reach Management PCs, but Sales-to-Engineering and all return traffic still works.


## Verification commands used

| Command | Verifies |
|---------|----------|
| `show ip route` | All three subnets present as directly connected |
| `show vlan brief` | VLANs created and access ports correctly assigned |
| `show interfaces trunk` | Trunk links up, allowed VLAN list correct |
| `show ip interface brief` | SVIs are up/up with correct addresses |
| `show ip dhcp binding` | DHCP leases issued across all VLANs |
| `show access-lists` | ACL match counters incrementing as expected |
<img width="1920" height="1017" alt="small office network" src="https://github.com/user-attachments/assets/8dfbd441-8808-4c3b-bc5e-9cd99127c1dc" />


## How to open this project

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a NetAcad account)
2. Open `small network office.pkt` in Packet Tracer
3. Wait ~30 seconds for Spanning Tree to converge
4. Open any PC → Desktop → Command Prompt → test connectivity with `ping`
   

## What I learned

- The 3650 multilayer switch requires `switchport trunk encapsulation dot1q` before `switchport mode trunk` will take effect — a common gotcha that silently breaks trunks.
- An access switch only needs to know about its own local VLAN; the trunk forwards tagged frames for other VLANs without the switch needing to "have" those VLANs locally.
- ACLs need an explicit `permit ip any any` at the end — otherwise the implicit deny blocks all remaining traffic and breaks the network.
- Always exclude the gateway address range from DHCP pools before assigning SVI IPs.

---

📦 EVPN Route Types Overview
Route Type	Name	Description	Common Use

Type 1	Ethernet Auto-Discovery (EAD)	Signals that a device is part of a broadcast domain (VNI) or multihoming ES	MLAG / ESI multihoming

Type 2	MAC/IP Advertisement	Advertises a MAC address (optionally with IP)	Host MAC + IP mobility

Type 3	Inclusive Multicast Ethernet Tag (IMET)	Advertises BUM (Broadcast, Unknown, Multicast) membership for a VNI	VXLAN flood list (BUM replication)

Type 4	Ethernet Segment Route	Identifies Ethernet Segments (ESI) for multihomed CE detection	MLAG peer detection in EVPN

Type 5	IP Prefix Route	Advertises IP prefixes from L3 VNIs (IRB)	IRB / L3VNI routing

📘 VXLAN EVPN Configuration: Arista EOS vs Cisco NX-OS

| Configuration Component      | Arista EOS Example                                      | Cisco NX-OS Example                                      |
|-----------------------------|----------------------------------------------------------|----------------------------------------------------------|
| **VTEP Loopback**           | `interface Loopback0`<br>`ip address 10.0.0.1/32`       | `interface loopback0`<br>`ip address 10.0.0.1/32`        |
| **VXLAN Tunnel Interface**  | `interface Vxlan1`<br>`vxlan source-interface Loopback0`| `interface nve1`<br>`source-interface loopback0`         |
| **L2 VNI Mapping**          | `vxlan vlan 10 vni 10010`                               | `member vni 10010 associate-vrf` or `evpn instance`      |
| **L3 VNI Mapping**          | `vxlan vrf Tenant-A vni 50010`                          | `vrf context Tenant-A`<br>`vni 50010`                    |
| **VRF Definition**          | `vrf instance Tenant-A`                                 | `vrf context Tenant-A`                                   |
| **SVI for IRB Gateway**     | `interface Vlan10`<br>`ip address virtual 10.1.10.1/24`<br>`vrf Tenant-A` | `interface Vlan10`<br>`ip address 10.1.10.1/24`<br>`fabric forwarding mode anycast-gateway`<br>`vrf member Tenant-A` |
| **Anycast Gateway**         | `ip address virtual <gateway>`                          | `fabric forwarding mode anycast-gateway`                 |
| **BGP Router Setup**        | `router bgp 65001`<br>`address-family evpn`             | `router bgp 65001`<br>`l2vpn evpn`                       |
| **EVPN Route Activation**   | `neighbor <ip> activate` under `address-family evpn`    | `evpn` <br>`neighbor <ip> activate` under `l2vpn evpn`   |
| **Route Distinguisher / RT**| `vlan-aware-bundle` or per-VLAN RT                      | `evpn instance` or `vrf` with `rd` and `route-target`    |
| **L3 Route Injection**      | `redistribute connected` under `vrf` in BGP             | `redistribute direct` under `vrf` in BGP                 |
| **MAC/IP Advertisement**    | Auto with `ip address virtual`                          | Auto with `fabric forwarding anycast-gateway`            |
| **Show VNI Status**         | `show vxlan vni`                                        | `show nve vni`                                           |
| **Show EVPN Type-2 Routes** | `show bgp evpn route-type mac-ip`                       | `show bgp l2vpn evpn route-type 2`                       |
| **Show EVPN Type-5 Routes** | `show bgp evpn route-type ip-prefix`                    | `show bgp l2vpn evpn route-type 5`                       |
| **Show BGP Routes in VRF**  | `show ip bgp vrf Tenant-A`                              | `show bgp vrf Tenant-A`                                  |
| **Show IP Route in VRF**    | `show ip route vrf Tenant-A`                            | `show ip route vrf Tenant-A`                             |

📡 gNMI Telemetry Validation: Arista EOS vs Cisco NX-OS

| Task / Area                      | Arista EOS (EOS ≥ 4.23.2F)                            | Cisco NX-OS (9.3(5)+ for N9K)                          |
|----------------------------------|--------------------------------------------------------|--------------------------------------------------------|
| ✅ Check gNMI Service Status     | `show management api gnmi`                             | `show grpc gnmi service`                               |
| 🔌 Verify Port is Listening      | `show management api gnmi | json` <br> (`:8080` or `:6030`) | `show grpc internal gnmi`                              |
| 🔐 Certificate/SSL Check         | `show management api gnmi certificate`                 | `show run | include gnmi server`                        |
| ⚙️ Enable gNMI Server            | `management api gnmi` <br> `transport grpc default`    | `feature gnmi` <br> `gnmi server`                      |
| 🧠 Check Subscriptions           | `show management api gnmi subscriptions`               | `show telemetry internal subscription all`             |
| 📬 Streaming Data to Collector   | Validate via external collector (e.g. Telegraf, InfluxDB, Prometheus) | Same — collector must decode JSON/gRPC correctly       |
| 📦 Supported YANG Models         | `show management api capabilities`                     | `show telemetry model` or `show gnmi capabilities`     |
| 🛠 Debug gNMI Stream             | Packet capture on TCP 6030 or 57400                    | Same (Wireshark decode as gRPC + gNMI)                 |
| 🧾 gNMI Path Query (external)    | Use `gnmi_cli` or `pygnmi` to `get / subscribe / set`  | Same (gNMI clients work cross-vendor)                  |
| 🧰 Example gNMI Path             | `/interfaces/interface[name=Ethernet1]/state/counters/in-octets` | `/interfaces/interface[name=Ethernet1]/state/counters/in-octets` |

🔗 What is ESI Multihoming in EVPN?
ESI (Ethernet Segment Identifier) multihoming allows a host or device to connect to multiple leaf switches for resiliency and active-active traffic in an EVPN VXLAN fabric.

In this model:

The same MAC/IP is reachable via multiple VTEPs.

A unique ESI (e.g., 0000:0000:0000:0001) identifies the multihomed segment.

EVPN Route Types 1 and 4 are used for DF (Designated Forwarder) election, loop prevention, and MAC synchronization.

📦 Use Cases
Use Case	Benefit
Dual-homed servers or appliances	Active/active or active/standby
Dual-homed top-of-rack switches	Fabric redundancy
Interconnecting MLAG domains	Multi-site VXLAN EVPN fabrics

📘 How ESI Multihoming Works
Each leaf connected to the same host advertises:

Type-1: Ethernet Auto-Discovery with ESI

Type-4: Ethernet Segment Route with ESI

DF Election (Designated Forwarder):

One leaf is elected to forward BUM traffic for that segment.

Prevents duplication and loops.

Type-2 routes (MAC/IP) may be advertised with ESI attached.

🧰 Arista EOS ESI Multihoming Example
🔧 Step 1: Define Ethernet Segment
```bash

interface Ethernet1
  switchport
  ethernet-segment identifier 0000:0000:0000:0001
  ethernet-segment redundancy single-active
```
🔧 Step 2: Enable under Port-Channel
```bash
interface Port-Channel10
  switchport
  ethernet-segment identifier 0000:0000:0000:0001
  ethernet-segment redundancy single-active
```
Can also use multi-active if the downstream supports it.

🧰 Cisco NX-OS ESI Multihoming Example
🔧 Step 1: Enable Feature
```bash
feature evpn
```
🔧 Step 2: Configure Interface with ESI
```bash
interface Ethernet1/1
  no switchport
  evpn ethernet-segment
    identifier 0000.0000.0000.0001.00
    redundancy mode single-active
```
ESI format in NX-OS = 0000.0000.0000.0001.00 (5 x 4 hex digits + 1 byte)

🔧 Step 3: Enable EVPN Multihoming
```bash
evpn
  esi multihoming
```
🧪 Validation Commands
|Check	                 |Arista EOS	                              |Cisco NX-OS
|------------------------|-----------------------------------------|--------------------------------------|
|ESI Route Advertised	|show bgp evpn route-type ethernet-segment|	show bgp l2vpn evpn route-type 4     |
|EAD Route (Type-1)	|show bgp evpn route-type ethernet-auto-discovery	|show bgp l2vpn evpn route-type 1 |
|MAC Advertisement with ESI	|show bgp evpn route-type mac-ip	|show bgp l2vpn evpn route-type 2 |
|DF Election Status	|show ethernet-segment	|show evpn ethernet-segment |

🧠 Notes
ESI must be consistent across all participating leafs for a segment.

Works with or without MLAG — MLAG syncs MACs automatically, ESI makes it BGP-visible.

BGP Route Reflector must reflect Type 1/2/4 routes correctly.

🧪 VXLAN EVPN Troubleshooting Commands: Arista EOS vs Cisco NX-OS

| Troubleshooting Area        | Arista EOS Command                            | Cisco NX-OS Command                                |
|-----------------------------|-----------------------------------------------|----------------------------------------------------|
| 🔄 BGP EVPN Summary         | `show bgp evpn summary`                       | `show bgp l2vpn evpn summary`                      |
| 👥 BGP Neighbor in VRF      | `show ip bgp neighbors vrf <VRF>`            | `show bgp vrf <VRF> neighbors`                     |
| 📦 Show L2 VNI Info         | `show vxlan vni`                              | `show nve vni`                                     |
| 🌐 VXLAN Tunnel Interface   | `show vxlan interface`                        | `show interface nve1`                              |
| 🧠 BGP Routes in VRF        | `show ip bgp vrf <VRF>`                       | `show bgp vrf <VRF>`                               |
| 📡 IP Routes in VRF         | `show ip route vrf <VRF>`                     | `show ip route vrf <VRF>`                          |
| 📬 EVPN Type-2 (MAC+IP)     | `show bgp evpn route-type mac-ip`            | `show bgp l2vpn evpn route-type 2`                 |
| 🌍 EVPN Type-5 (IP Prefix)  | `show bgp evpn route-type ip-prefix`         | `show bgp l2vpn evpn route-type 5`                 |
| 📣 EVPN Type-3 (BUM)        | `show bgp evpn route-type inclusive-multicast` | `show bgp l2vpn evpn route-type 3`               |
| 🔧 MAC Table (VXLAN)        | `show mac address-table dynamic vxlan`        | `show mac address-table vlan <vlan>`               |
| 🌐 ARP Table in VRF         | `show ip arp vrf <VRF>`                       | `show ip arp vrf <VRF>`                            |
| ⚠ Interface Counters        | `show interfaces counters errors`            | `show interface counters errors`                   |
| 📊 MTU Check                | `show interfaces | include MTU`               | `show interface <intf> | include MTU`             |
| 🔗 Ping VTEP Loopback       | `ping <remote> source <loopback>`            | `ping <remote> source <loopback>`                  |
| 🧪 Traceroute               | `traceroute <remote>`                         | `traceroute <remote>`                              |

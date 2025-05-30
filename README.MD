##📘 VXLAN EVPN Configuration: Arista EOS vs Cisco NX-OS

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

# Домашнее задание №6
## VxLAN. L3 VNI
### Цели:
- Настроить маршрутизацию в рамках Overlay между клиентами
------------------------------
### Решение:
#### IP PLAN LEAF-SPINE:
 
| Hostname   |Interface | IP/Mask  | Description |
| ----------- | ----------- |-----------|-----------|
| DC1-Leaf-01   | Lo1   |10.1.1.0/32    |Loopback0|
| DC1-Leaf-01   | Eth1   |10.2.1.0/31  |Link to DC1-Spine-01|
| DC1-Leaf-01   | Eth2   |10.2.2.0/31  |Link to DC1-Spine-02|
| DC1-Leaf-02   | Lo1   |10.1.2.0/32    |Loopback0|
| DC1-Leaf-02   | Eth1   |10.2.1.2/31  |Link to DC1-Spine-01|
| DC1-Leaf-02   | Eth2   |10.2.2.2/31  |Link to DC1-Spine-02|
| DC1-Leaf-03   | Lo1   |10.1.3.0/32    |Loopback0|
| DC1-Leaf-03   | Eth1   |10.2.1.4/31  |Link to DC1-Spine-01|
| DC1-Leaf-03   | Eth2   |10.2.2.4/31  |Link to DC1-Spine-02|
| DC1-Spine-01   | Lo0   |10.0.1.0/32    |Loopback0|
| DC1-Spine-01   | Eth1   |10.2.1.1/31  |Link to DC1-Leaf-01|
| DC1-Spine-01   | Eth2   |10.2.1.3/31  |Link to DC1-Leaf-02|
| DC1-Spine-01   | Eth3   |10.2.1.5/31  |Link to DC1-Leaf-03|
| DC1-Spine-02   | Lo0   |10.0.2.0/32    |Loopback0|
| DC1-Spine-02   | Eth1   |10.2.2.1/31  |Link to DC1-Leaf-01|
| DC1-Spine-02   | Eth2   |10.2.2.3/31  |Link to DC1-Leaf-02|
| DC1-Spine-02   | Eth3   |10.2.2.5/31  |Link to DC1-Leaf-03|

#### IP PLAN Clients:
| Network   |Vlan | Description | vrf|
| ----------- | ----------- |-----------|---------|
| 10.0.10.0/24 | Vlan 10    |net-10| PROD |
| 10.0.20.0/24 | Vlan 20    | net-20| PROD |

| Hostname   |Vlan | IP/Mask  | Description | GW|
| ----------- | ----------- |-----------|-----------|----------|
| Client-1  | Vlan 10   |10.0.10.101/24    |Client net-10|10.0.10.1|
| Client-2  | Vlan 20   |10.0.20.102/24  |Client net-20|10.0.20.1|
| Client-3   |Vlan 10   |10.0.10.103/24  |Client net-10|10.0.10.1|
| Client-4  | Vlan 20    |10.0.20.104/24    |Client net-20|10.0.20.1|


#### Структурная схема сети:
![123](/lesson7/DC-Topology_Multihoming.png)


#### Конфигурация устройств:
----------------------------
 
- ##### DC1-Leaf-01:
 
```
! Command: show running-config
! device: DC1-Leaf-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-Leaf-01
!
spanning-tree mode mstp
!
vlan 10
   name net-10
!
vlan 20
   name net-20
!
vrf instance PROD
   description ### PROD SERVERS ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0010:aaaa:aaaa:aaaa:aaaa
      route-target import 00:10:aa:aa:aa:aa
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0020:aaaa:aaaa:aaaa:aaaa
      route-target import 00:20:aa:aa:aa:aa
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC1-Spine-01 int Eth1 ###
   no switchport
   ip address 10.2.1.0/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 JARL+7iYPO16qHVlDBzu4cpOF/JUAIGn
!
interface Ethernet2
   description ### Link To DC1-Spine2 int Eth2 ###
   no switchport
   ip address 10.2.2.0/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   channel-group 20 mode active
!
interface Ethernet8
   channel-group 10 mode active
!
interface Ethernet9
!
interface Loopback1
   ip address 10.1.1.0/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan20
   description ### GW net-20 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 10001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
!
router bgp 65001
   router-id 10.1.1.0
   timers bgp 3 9
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 10.0.1.0 peer group SPINES
   neighbor 10.0.2.0 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 20
      rd 65001:20
      route-target both 65001:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf PROD
      rd 10.1.1.0:10001
      route-target import 65001:10001
      route-target export 65001:10001
      redistribute connected
!
router ospf 1
   router-id 10.1.1.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
!
end

```
----------------------------
- ##### DC1-Leaf-02:
```
! Command: show running-config
! device: DC1-Leaf-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-Leaf-02
!
spanning-tree mode mstp
!
vlan 10
   name net-10
!
vlan 20
   name net-20
!
vrf instance PROD
!
interface Port-Channel10
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0010:aaaa:aaaa:aaaa:aaaa
      route-target import 00:10:aa:aa:aa:aa
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0020:aaaa:aaaa:aaaa:aaaa
      route-target import 00:20:aa:aa:aa:aa
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.2/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 JARL+7iYPO16qHVlDBzu4cpOF/JUAIGn
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.2/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   channel-group 10 mode active
!
interface Ethernet8
   channel-group 20 mode active
!
interface Ethernet9
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.2.0/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan20
   description ### GW net-20 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 10001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
!
router bgp 65001
   router-id 10.1.2.0
   timers bgp 3 9
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 10.0.1.0 peer group SPINES
   neighbor 10.0.2.0 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 20
      rd 65001:20
      route-target both 65001:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf PROD
      rd 10.1.3.0:10001
      route-target import 65001:10001
      route-target export 65001:10001
      redistribute connected
!
router ospf 1
   router-id 10.1.2.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
   log-adjacency-changes detail
!
end

```  
----------------------------
- ##### DC1-Leaf-03:
```
! Command: show running-config
! device: DC1-Leaf-03 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-Leaf-03
!
spanning-tree mode mstp
!
vlan 10
   name net-10
!
vlan 20
   name net-20
!
vrf instance PROD
!
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.4/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 JARL+7iYPO16qHVlDBzu4cpOF/JUAIGn
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.4/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   description ### Link to Client-4 int e0/0 ###
   switchport trunk allowed vlan 20
   switchport mode trunk
!
interface Ethernet8
   description ### Link to Client-3 int e0/0 ###
   switchport trunk allowed vlan 10
   switchport mode trunk
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.3.0/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan20
   description ### GW net-20 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 10001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
!
router bgp 65001
   router-id 10.1.3.0
   timers bgp 3 9
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 10.0.1.0 peer group SPINES
   neighbor 10.0.2.0 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 20
      rd 65001:20
      route-target both 65001:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINES activate
   !
   address-family ipv4
      no neighbor SPINES activate
   !
   vrf PROD
      rd 10.1.3.0:10001
      route-target import 65001:10001
      route-target export 65001:10001
      redistribute connected
!
router ospf 1
   router-id 10.1.3.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
   log-adjacency-changes detail
!
end

```  
----------------------------
- ##### DC1-Spine-01:
``` 
! Command: show running-config
  ! device: DC1-Spine-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-Spine-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Leaf-01 int Eth1 ###
   no switchport
   ip address 10.2.1.1/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 JARL+7iYPO16qHVlDBzu4cpOF/JUAIGn
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth1 ###
   no switchport
   ip address 10.2.1.3/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth1 ###
   no switchport
   ip address 10.2.1.5/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Loopback0
   description ### Lo0 ###
   ip address 10.0.1.0/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 10.0.1.0
   timers bgp 3 9
   bgp cluster-id 0.0.0.1
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback0
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback0
   neighbor 10.0.2.0 peer group RR
   neighbor 10.1.1.0 peer group LEAFS
   neighbor 10.1.2.0 peer group LEAFS
   neighbor 10.1.3.0 peer group LEAFS
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor RR activate
!
router ospf 1
   router-id 10.0.1.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
   log-adjacency-changes detail
!
end


```  
----------------------------
- ##### DC1-Spine-02:
```
! Command: show running-config
 ! device: DC1-Spine-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-Spine-02
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Leaf-01 int Eth2 ###
   no switchport
   ip address 10.2.2.1/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 JARL+7iYPO16qHVlDBzu4cpOF/JUAIGn
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth2 ###
   no switchport
   ip address 10.2.2.3/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth2 ###
   no switchport
   ip address 10.2.2.5/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf message-digest-key 1 sha512 7 Oo8j5zYaEmUGCPBnpnrbWRi8SWTSbhlB
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Ethernet9
!
interface Loopback0
   description ### Lo0 ###
   ip address 10.0.2.0/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 10.0.2.0
   timers bgp 3 9
   bgp cluster-id 0.0.0.1
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback0
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback0
   neighbor 10.0.1.0 peer group RR
   neighbor 10.1.1.0 peer group LEAFS
   neighbor 10.1.2.0 peer group LEAFS
   neighbor 10.1.3.0 peer group LEAFS
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor RR activate
!
router ospf 1
   router-id 10.0.2.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
   log-adjacency-changes detail
!
end


```

----------------------------
#### Проверка BGP EVPN таблицы, ARP, таблицы маршрутизации VRF:
- ##### DC1-Leaf-01
- ###### EVPN Route-Type 4
```
BGP routing table information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.1.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.2.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.2.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
```
- ###### EVPN Route-Type 1
```
BGP routing table information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 *        RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.2.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 *        RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.2.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1

```
- ###### ARP
```

```

- ##### DC1-Leaf-02
- ###### EVPN Route-Type 4
```
BGP routing table information for VRF default
Router identifier 10.1.2.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.1.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.2.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.1.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.2.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 -                     -       -       0       i
```
- ###### EVPN Route-Type 1
```
BGP routing table information for VRF default
Router identifier 10.1.2.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.1.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.2.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 * >      RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.1.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.2.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 -                     -       -       0       i

```
- ###### ARP
```

```
- ###### Таблица маршрутизации VRF PROD
```

```
- ##### DC1-Leaf-03
- ###### EVPN Route-Type 4
```
BGP routing table information for VRF default
Router identifier 10.1.3.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.1.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.2.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 ethernet-segment 0010:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.1.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.2.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 ethernet-segment 0020:aaaa:aaaa:aaaa:aaaa 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
```
- ###### EVPN Route-Type 1
```
BGP routing table information for VRF default
Router identifier 10.1.3.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 auto-discovery 0 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.1.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.2.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 auto-discovery 0010:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 auto-discovery 0 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.1.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.1.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 10.1.2.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 10.1.2.0:1 auto-discovery 0020:aaaa:aaaa:aaaa:aaaa
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
```
- ###### ARP
```

```
- ###### Таблица маршрутизации VRF PROD
```

```

#### Проверка IP связности клиентов:
- ##### ICMP Clinet-1 to Clinet-2
```

```
- ##### ICMP Clinet-1 to Clinet-4
```

```
- ##### ICMP Clinet-2 to Clinet-1
```

```
- ##### ICMP Clinet-2 to Clinet-3
```

```
- ##### ICMP Clinet-4 to Clinet-1
```

```
- ##### ICMP Clinet-4 to Clinet-3
```

```

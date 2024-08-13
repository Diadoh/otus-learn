# Домашнее задание №7
## VxLAN. Routing.
### Цели:
- Реализовать передачу суммарных префиксов через EVPN route-type 5
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
| 10.0.30.0/24 | Vlan 30    | net-30| SERVICE |
| 10.100.1.0/30 | Vlan 1020    | net-20| PROD |
| 10.100.2.0/30 | Vlan 3030    | net-20| SERVICE |

| Hostname   |Vlan | IP/Mask  | Description | GW|
| ----------- | ----------- |-----------|-----------|----------|
| Client-1  | Vlan 10   |10.0.10.101/24    |Client net-10|10.0.10.1|
| Client-2  | Vlan 20   |10.0.20.102/24  |Client net-20|10.0.20.1|
| Client-3   |Vlan 10   |10.0.10.103/24  |Client net-10|10.0.10.1|
| Client-4  | Vlan 20    |10.0.20.104/24    |Client net-20|10.0.20.1|
| Client-5  | Vlan 30    |10.0.30.105/24    |Client net-30|10.0.30.1|
| Client-6  | Vlan 30    |10.0.30.106/24    |Client net-30|10.0.30.1|


#### Структурная схема сети:

![123](/lesson8/DC-Topology_Routing.png)\


#### Конфигурация устройств:

<details>
  <summary>DC1-Leaf-01</summary>
  
  ```
 DC1-Leaf-01#sh run
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
vlan 30
   name net-30
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance SERVICE
   description ### SERVICE SERVERS ###
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
interface Port-Channel11
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
   switchport trunk allowed vlan 30
   switchport mode trunk
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
interface Vlan30
   description ### GW net-30 ###
   vrf SERVICE
   ip address virtual 10.0.30.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf PROD vni 10001
   vxlan vrf SERVICE vni 10002
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf SERVICE
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
   vlan 30
      rd 65001:30
      route-target both 65001:10030
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
   vrf SERVICE
      rd 10.1.1.0:10002
      route-target import 65001:10002
      route-target export 65001:10002
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
  
</details>

<details>
  <summary>DC1-Leaf-02</summary>
  
  ```
  DC1-Leaf-02#sh run
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
interface Port-Channel11
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
  
</details>

<details>
  <summary>DC1-Leaf-03</summary>
  
  ```
  DC1-Leaf-03#sh run
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
vlan 30
   name net-30
!
vlan 1020
   name ospf-vrf-prod
!
vlan 3030
   name ospf-vrf-service
!
vrf instance PROD
!
vrf instance SERVICE
   description ### SERVICE SERVERS ###
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
   switchport trunk allowed vlan 1020,3030
   switchport mode trunk
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
   switchport trunk allowed vlan 30
   switchport mode trunk
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
interface Vlan30
   description ### GW net-30 ###
   vrf SERVICE
   ip address virtual 10.0.30.1/24
!
interface Vlan1020
   description ospf-vrf-prod
   vrf PROD
   ip address 10.100.1.1/30
!
interface Vlan3030
   description ospf-vrf-service
   vrf SERVICE
   ip address 10.100.2.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf PROD vni 10001
   vxlan vrf SERVICE vni 10002
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf SERVICE
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
   vlan 30
      rd 65001:30
      route-target both 65001:10030
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
      redistribute ospf
      redistribute ospf match external
   !
   vrf SERVICE
      rd 10.1.1.0:10002
      route-target import 65001:10002
      route-target export 65001:10002
      redistribute connected
      redistribute ospf
      redistribute ospf match external
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
router ospf 1020 vrf PROD
   router-id 10.100.1.1
   redistribute bgp
   network 10.0.0.0/8 area 0.0.0.0
   max-lsa 12000
!
router ospf 3030 vrf SERVICE
   router-id 10.100.2.1
   redistribute bgp
   network 10.0.0.0/8 area 0.0.0.0
   max-lsa 12000
!
end

  ```
  
</details>

<details>
  <summary>DC1-Spine-01</summary>
  
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
  
</details>

<details>
  <summary>DC1-Spine-02</summary>
  
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
  
</details>

<details>
  <summary>DC1-Router-01</summary>
  
  ```
  DC1-Router-01#sh run
Building configuration...

Current configuration : 1332 bytes
!
version 15.7
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname DC1-Router-01
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
clock timezone EET 2 0
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!


!
!
!
!
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Ethernet0/0
 no ip address
 duplex auto
!
interface Ethernet0/0.1020
 description ### Link DC1-Leaf-03 int vlan 1020 ###
 encapsulation dot1Q 1020
 ip address 10.100.1.2 255.255.255.252
!
interface Ethernet0/0.3030
 description ### Link DC1-Leaf-03 int vlan 3030 ###
 encapsulation dot1Q 3030
 ip address 10.100.2.2 255.255.255.252
!
interface Ethernet0/1
 no ip address
 shutdown
 duplex auto
!
interface Ethernet0/2
 no ip address
 shutdown
 duplex auto
!
interface Ethernet0/3
 no ip address
 shutdown
 duplex auto
!
router ospf 1
 router-id 10.100.1.2
 network 10.0.0.0 0.255.255.255 area 0
 default-information originate
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 ioam timestamp
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

  ```
  
</details>


### Проверки

<details>
  <summary>ICMP Client-1</summary>
  
  ```
  Client-1#ping 10.0.20.102
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.102, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/17/21 ms
Client-1#ping 10.0.10.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.10.103, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 54/56/60 ms
Client-1#ping 10.0.20.104
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.104, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 39/42/46 ms
Client-1#ping 10.0.30.105
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.30.105, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 67/77/103 ms
Client-1#ping 10.0.30.106
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.30.106, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 45/54/64 ms
  ```
  
</details>

<details>
  <summary>DC1-Leaf-01</summary>
  
  ```
  DC1-Leaf-01#sh ip route vrf PROD

VRF: PROD
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        10.0.10.0/24 is directly connected, Vlan10
 B I      10.0.20.104/32 [200/0] via VTEP 10.1.3.0 VNI 10001 router-mac 50:00:00:d5:5d:c0 local-inteface Vxlan1                                                                                                                                        
 C        10.0.20.0/24 is directly connected, Vlan20
 B I      10.0.30.105/32 [200/0] via VTEP 10.1.3.0 VNI 10001 router-mac 50:00:00:d5:5d:c0 local-inteface Vxlan1                                                                                                                                        
 B I      10.0.30.0/24 [200/0] via VTEP 10.1.3.0 VNI 10001 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.100.1.0/30 [200/0] via VTEP 10.1.3.0 VNI 10001 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                       
 B I      10.100.2.0/30 [200/0] via VTEP 10.1.3.0 VNI 10001 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                        
  ```
```
DC1-Leaf-01#sh ip route vrf SERVICE

VRF: SERVICE
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B I      10.0.10.101/32 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.0.10.0/24 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.0.20.102/32 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.0.20.0/24 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.0.30.106/32 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 C        10.0.30.0/24 is directly connected, Vlan30
 B I      10.100.1.0/30 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1                                                                                                                                         
 B I      10.100.2.0/30 [200/0] via VTEP 10.1.3.0 VNI 10002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1   
```
```
DC1-Leaf-01#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.1.0:10001 ip-prefix 10.0.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.10.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.10.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.10.101/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.10.101/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10001 ip-prefix 10.0.20.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.20.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.20.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.20.102/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.20.102/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.30.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.30.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.30.0/24
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.30.105/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.30.105/32
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.100.1.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.100.1.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.100.1.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.100.1.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.100.2.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.100.2.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.100.2.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.100.2.0/30
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
```
 
</details>
<details>
  <summary>DC1-Leaf-03</summary>
  
  ```
  DC1-Leaf-03(config)#show ip route vrf PROD

VRF: PROD
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B I      10.0.10.101/32 [200/0] via VTEP 10.1.1.0 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        10.0.10.0/24 is directly connected, Vlan10
 B I      10.0.20.102/32 [200/0] via VTEP 10.1.2.0 VNI 10001 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 C        10.0.20.0/24 is directly connected, Vlan20
 O        10.0.30.0/24 [110/30] via 10.100.1.2, Vlan1020
 C        10.100.1.0/30 is directly connected, Vlan1020
 O        10.100.2.0/30 [110/20] via 10.100.1.2, Vlan1020
  ```
```
DC1-Leaf-03(config)#show ip route vrf SERVICE

VRF: SERVICE
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 O E2     10.0.10.101/32 [110/1] via 10.100.2.2, Vlan3030
 O        10.0.10.0/24 [110/30] via 10.100.2.2, Vlan3030
 O E2     10.0.20.102/32 [110/1] via 10.100.2.2, Vlan3030
 O        10.0.20.0/24 [110/30] via 10.100.2.2, Vlan3030
 C        10.0.30.0/24 is directly connected, Vlan30
 O        10.100.1.0/30 [110/20] via 10.100.2.2, Vlan3030
 C        10.100.2.0/30 is directly connected, Vlan3030

```

```
DC1-Leaf-03(config)#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.3.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.1.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.10.0/24
                                 -                     -       -       0       i
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.10.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.10.101/32
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.20.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.20.0/24
                                 -                     -       -       0       i
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *        RD: 10.1.3.0:10001 ip-prefix 10.0.20.0/24
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.20.102/32
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10002 ip-prefix 10.0.30.0/24
                                 -                     -       -       0       i
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.30.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:10002 ip-prefix 10.0.30.0/24
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 10.1.3.0:10001 ip-prefix 10.0.30.0/24
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10002 ip-prefix 10.100.1.0/30
                                 -                     -       -       0       i
 * >      RD: 10.1.3.0:10001 ip-prefix 10.100.1.0/30
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:10002 ip-prefix 10.100.2.0/30
                                 -                     -       -       0       i
 * >      RD: 10.1.3.0:10001 ip-prefix 10.100.2.0/30
                                 -                     -       -       0       i
```

</details>


# Домашнее задание №5
## VxLAN. L2 VNI
### Цели:
- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
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
| Network   |Vlan | Description |
| ----------- | ----------- |-----------|
| 10.0.10.0/24 | Vlan 10    |net-10|
| 10.0.20.0/24 | Vlan 20    | net-20|

| Hostname   |Vlan | IP/Mask  | Description |
| ----------- | ----------- |-----------|-----------|
| Client-1  | Vlan 10   |10.0.10.101/24    |Client net-10|
| Client-2  | Vlan 20   |10.0.20.102/24  |Client net-20|
| Client-3   |Vlan 10   |10.0.10.103/24  |Client net-10|
| Client-4  | Vlan 20    |10.0.20.104/24    |Client net-20|


#### Структурная схема сети:
![123](/lesson5/DC-Topology_L2VXLAN.png)

#### Конфигурация устройств:
----------------------------
- ##### DC1-Leaf-01:
>! ```
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
!
interface Ethernet8
   description ### To Client-1 int e 0/0 ###
   switchport trunk allowed vlan 10
   switchport mode trunk
!
interface Ethernet9
!
interface Loopback1
   ip address 10.1.1.0/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
!
interface Ethernet8
   switchport trunk allowed vlan 20
   switchport mode trunk
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.2.0/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
   ip address 10.0.10.11/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
##### Client-1:
```
Client-1#sh run
Building configuration...

Current configuration : 1163 bytes
!
! Last configuration change at 14:26:42 EET Mon Jun 17 2024
!
version 15.7
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Client-1
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
interface Ethernet0/0.10
 description ### Link to DC1-Leaf-01 int Eth9 ###
 encapsulation dot1Q 10
 ip address 10.0.10.101 255.255.255.0
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
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.0.10.1
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
##### Client-2:
```
Client-2#sh run
Building configuration...

Current configuration : 1163 bytes
!
! Last configuration change at 21:42:51 EET Sun Jun 16 2024
!
version 15.7
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Client-2
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
interface Ethernet0/0.20
 description ### Link to DC1-Leaf-02 int Eth8 ###
 encapsulation dot1Q 20
 ip address 10.0.20.102 255.255.255.0
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
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.0.20.1
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
##### Client-3:
```
Client-3#sh run
Building configuration...

Current configuration : 1163 bytes
!
! Last configuration change at 14:29:38 EET Mon Jun 17 2024
!
version 15.7
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Client-3
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
interface Ethernet0/0.10
 description ### Link to DC1-Leaf-02 int Eth8 ###
 encapsulation dot1Q 10
 ip address 10.0.10.103 255.255.255.0
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
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.0.10.1
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
##### Client-4:
```
Client-4#sh run
Building configuration...

Current configuration : 1163 bytes
!
! Last configuration change at 21:36:00 EET Sun Jun 16 2024
!
version 15.7
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Client-4
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
interface Ethernet0/0.20
 description ### Link to DC1-Leaf-03 int Eth7 ###
 encapsulation dot1Q 20
 ip address 10.0.20.104 255.255.255.0
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
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.0.20.1
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

----------------------------
#### Проверка iBGP EVPN соседства:
- ##### DC1-Spine-01:
```
DC1-Spine-01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.2.0 4 65001          42152     42135    0    0    1d05h Estab   0      0
  10.1.1.0 4 65001          42504     42539    0    0 23:16:25 Estab   2      2
  10.1.2.0 4 65001          42475     42507    0    0 23:05:35 Estab   2      2
  10.1.3.0 4 65001          42520     42529    0    0 23:05:51 Estab   2      2

```
- ##### DC1-Spine-02:
```
DC1-Spine-02#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.0 4 65001          42188     42206    0    0    1d05h Estab   0      0
  10.1.1.0 4 65001          42094     42160    0    0 23:18:40 Estab   2      2
  10.1.2.0 4 65001          42100     42167    0    0 23:07:50 Estab   2      2
  10.1.3.0 4 65001          42157     42173    0    0 23:08:06 Estab   2      2
```

#### Проверка IP связности клиентов:
- ##### ICMP Clinet-1 to Clinet-3
```
  Client-1#ping 10.0.10.103
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.10.103, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 51/98/262 ms
```
- ##### ICMP Clinet-2 to Clinet-4
```
Client-2#ping 10.0.20.104
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.104, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 29/67/148 ms

```
- ##### DC1-Leaf-01 EVPN Route-Type 2/3
```
DC1-Leaf-01#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10 imet 10.1.1.0
                                 -                     -       -       0       i
 * >      RD: 65001:20 imet 10.1.1.0
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
DC1-Leaf-01#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10 mac-ip aabb.cc00.8000
                                 -                     -       -       0       i
 * >Ec    RD: 65001:20 mac-ip aabb.cc00.9000
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc00.9000
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.a000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.a000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.a000 10.0.10.103
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.a000 10.0.10.103
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc00.b000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc00.b000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1

```
- ##### DC1-Leaf-02 EVPN Route-Type 2/3
```
DC1-Leaf-02#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.2.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 65001:10 imet 10.1.2.0
                                 -                     -       -       0       i
 * >      RD: 65001:20 imet 10.1.2.0
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.3.0
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
DC1-Leaf-02#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.2.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.8000
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.8000
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >      RD: 65001:20 mac-ip aabb.cc00.9000
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.a000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.a000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.a000 10.0.10.103
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.a000 10.0.10.103
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc00.b000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc00.b000
                                 10.1.3.0              -       100     0       i Or-ID: 10.1.3.0 C-LST: 0.0.0.1

```
- ##### DC1-Leaf-03 EVPN Route-Type 2/3
```
DC1-Leaf-03#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.3.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.1.0
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 imet 10.1.2.0
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 65001:10 imet 10.1.3.0
                                 -                     -       -       0       i
 * >      RD: 65001:20 imet 10.1.3.0
                                 -                     -       -       0       i
DC1-Leaf-03#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.3.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 mac-ip aabb.cc00.8000
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc00.8000
                                 10.1.1.0              -       100     0       i Or-ID: 10.1.1.0 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc00.9000
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc00.9000
                                 10.1.2.0              -       100     0       i Or-ID: 10.1.2.0 C-LST: 0.0.0.1
 * >      RD: 65001:10 mac-ip aabb.cc00.a000
                                 -                     -       -       0       i
 * >      RD: 65001:10 mac-ip aabb.cc00.a000 10.0.10.103
                                 -                     -       -       0       i
 * >      RD: 65001:20 mac-ip aabb.cc00.b000
                                 -                     -       -       0       i
```
- ##### DC1-Leaf-01 VXLAN VTEP Peers
```
DC1-Leaf-01#show vxlan vtep detail
Remote VTEPS for Vxlan1:

VTEP           Learned Via         MAC Address Learning       Tunnel Type(s)
-------------- ------------------- -------------------------- --------------
10.1.2.0       control plane       control plane              flood
10.1.3.0       control plane       control plane              flood

```
- ##### DC1-Leaf-02 VXLAN VTEP Peers
```
DC1-Leaf-02#show vxlan vtep detail
Remote VTEPS for Vxlan1:

VTEP           Learned Via         MAC Address Learning       Tunnel Type(s)
-------------- ------------------- -------------------------- --------------
10.1.1.0       control plane       control plane              flood
10.1.3.0       control plane       control plane              flood

Total number of remote VTEPS:  2

```
- ##### DC1-Leaf-03 VXLAN VTEP Peers
```
DC1-Leaf-03#show vxlan vtep detail
Remote VTEPS for Vxlan1:

VTEP           Learned Via         MAC Address Learning       Tunnel Type(s)
-------------- ------------------- -------------------------- --------------
10.1.1.0       control plane       control plane              flood
10.1.2.0       control plane       control plane              flood

Total number of remote VTEPS:  2

```

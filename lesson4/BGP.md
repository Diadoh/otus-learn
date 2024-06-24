# Домашнее задание №3
## Underlay. IS-IS
### Цели:
- Настроить IS-IS для Underlay сети
------------------------------
### Решение:
#### IP PLAN:
 
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

#### IS-IS - Zone|System ID|Level|:
| Hostname   |Zone | System ID  | Level |
| ----------- | ----------- |-----------|-----------|
| DC1-Leaf-01   | 49.0001   |0100.1000.1000    |L1|
| DC1-Leaf-02   | 49.0001   |0100.0100.2000    |L1|
| DC1-Leaf-03   | 49.0001   |0100.0100.3000    |L1|
| DC1-Spine-01   | 49.0001  |0100.0000.1000    |L1/L2|
| DC1-Spine-02   | 49.0001  |0100.0000.2000   |L1/L2|

#### Структурная схема сети:
![123](/lesson4/DC-Topology_BGP.png)

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
service routing protocols model ribd
!
hostname DC1-Leaf-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC1-Spine-01 int Eth1 ###
   no switchport
   ip address 10.2.1.0/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf1
   isis authentication mode md5
   isis authentication key 7 6BVVvCxtZO2/zgQ+MhA3Nw==
!
interface Ethernet2
   description ### Link to DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.0/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf1
   isis authentication mode md5
   isis authentication key 7 6BVVvCxtZO2/zgQ+MhA3Nw==
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
!
interface Ethernet9
!
interface Loopback1
   ip address 10.1.1.0/32
!
interface Management1
!
ip routing
!
route-map con-in-isis permit 10
   set isis level level-1
!
router isis Leaf1
   net 49.0001.0100.1001.0000.00
   is-hostname DC1-Leaf-01
   is-type level-1
   log-adjacency-changes
   redistribute connected route-map con-in-isis
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
service routing protocols model ribd
!
hostname DC1-Leaf-02
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.2/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf2
   isis authentication mode md5
   isis authentication key 7 ZSmEUsO0NGUHpr/QZURdbA==
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.2/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf2
   isis authentication mode md5
   isis authentication key 7 ZSmEUsO0NGUHpr/QZURdbA==
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
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.2.0/32
!
interface Management1
!
ip routing
!
route-map con-in-isis permit 10
   set isis level level-1
!
router isis Leaf2
   net 49.0001.0100.0100.2000.00
   is-hostname DC1-Leaf-02
   is-type level-1
   log-adjacency-changes
   redistribute connected route-map con-in-isis
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
service routing protocols model ribd
!
hostname DC1-Leaf-03
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.4/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf3
   isis authentication mode md5
   isis authentication key 7 ZSmEUsO0NGUHpr/QZURdbA==
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.4/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Leaf3
   isis authentication mode md5
   isis authentication key 7 ZSmEUsO0NGUHpr/QZURdbA==
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
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.3.0/32
!
interface Management1
!
ip routing
!
route-map con-in-isis permit 10
   set isis level level-1
!
router isis Leaf3
   net 49.0001.0100.0103.0000.00
   is-hostname DC1-Leaf-03
   is-type level-1
   redistribute connected route-map con-in-isis
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
service routing protocols model ribd
!
hostname DC1-Spine-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Leaf-01 int Eth1 ###
   no switchport
   ip address 10.2.1.1/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine1
   isis authentication mode md5
   isis authentication key 7 0lsBFyXym7AI8YbJRi2oyg==
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth1 ###
   no switchport
   ip address 10.2.1.3/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine1
   isis authentication mode md5
   isis authentication key 7 0lsBFyXym7AI8YbJRi2oyg==
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth1 ###
   no switchport
   ip address 10.2.1.5/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine1
   isis authentication mode md5
   isis authentication key 7 0lsBFyXym7AI8YbJRi2oyg==
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
route-map con-in-isis permit 10
   set isis level level-1
!
router isis Spine1
   net 49.0001.0100.0000.1000.00
   is-hostname DC1-Spine-01
   redistribute connected route-map con-in-isis
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
service routing protocols model ribd
!
hostname DC1-Spine-02
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link To DC1-Leaf-01 int Eth2 ###
   no switchport
   ip address 10.2.2.1/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine2
   isis authentication mode md5
   isis authentication key 7 zRl/G6jQcW7Jv01kszaCBw==
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth2 ###
   no switchport
   ip address 10.2.2.3/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine2
   isis authentication mode md5
   isis authentication key 7 zRl/G6jQcW7Jv01kszaCBw==
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth2 ###
   no switchport
   ip address 10.2.2.5/31
   bfd interval 200 min-rx 200 multiplier 4
   isis enable Spine2
   isis authentication mode md5
   isis authentication key 7 zRl/G6jQcW7Jv01kszaCBw==
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
route-map con-in-isis permit 10
   set isis level level-1
!
router isis Spine2
   net 49.0001.0100.0002.0000.00
   is-hostname DC1-Spine-02
   redistribute connected route-map con-in-isis
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end

```  
----------------------------
#### Проверка IS-IS соседства, содержимое таблиц машрутизации и LSDB:
- ##### DC1-Spine-01:
- ###### IS-IS соседство:
 ```
DC1-Spine-01#sh isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
Spine1    default  DC1-Leaf-02      L1   Ethernet2          50:0:0:cb:38:c2   UP    9           DC1-Leaf-02.0d
Spine1    default  DC1-Leaf-03      L1   Ethernet3          50:0:0:d5:5d:c0   UP    8           DC1-Leaf-03.0d
Spine1    default  DC1-Leaf-01      L1   Ethernet1          50:0:0:d7:ee:b    UP    9           DC1-Leaf-01.0e

```
- ###### Таблица маршрутизации:
```
DC1-Spine-01#sh ip route

VRF: default
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

 C        10.0.1.0/32 is directly connected, Loopback0
 I L1     10.0.2.0/32 [115/21] via 10.2.1.0, Ethernet1
                               via 10.2.1.2, Ethernet2
                               via 10.2.1.4, Ethernet3
 I L1     10.1.1.0/32 [115/11] via 10.2.1.0, Ethernet1
 I L1     10.1.2.0/32 [115/11] via 10.2.1.2, Ethernet2
 I L1     10.1.3.0/32 [115/11] via 10.2.1.4, Ethernet3
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3
 I L1     10.2.2.0/31 [115/20] via 10.2.1.0, Ethernet1
 I L1     10.2.2.2/31 [115/20] via 10.2.1.2, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.1.4, Ethernet3
```
- ###### LSDB:

```
DC1-Spine-01#show isis database

IS-IS Instance: Spine1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    DC1-Spine-01.00-00           39  25621   624    150 L2 <>
    DC1-Spine-02.00-00           36   9561  1170    150 L2 <>
    DC1-Leaf-02.00-00            30  46509   473    125 L1 <>
    DC1-Leaf-02.0d-00            21  47011   510     51 L1 <>
    DC1-Leaf-02.0e-00            20  16425   819     51 L1 <>
    DC1-Leaf-03.00-00            27  20323   866    125 L1 <>
    DC1-Leaf-03.0d-00            17  38403   801     51 L1 <>
    DC1-Leaf-03.0e-00            18   7050   388     51 L1 <>
    DC1-Leaf-01.00-00            39  49107   999    125 L1 <>
    DC1-Leaf-01.0e-00            22  53161   575     51 L1 <>
    DC1-Leaf-01.0f-00            15  25641   897     51 L1 <>
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    DC1-Spine-01.00-00           53  40900   510    169 L2 <>

```
- ##### DC1-Spine-02:
```
DC1-Spine-02#sh isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
Spine2    default  DC1-Leaf-02      L1   Ethernet2          50:0:0:cb:38:c2   UP    7           DC1-Leaf-02.0e
Spine2    default  DC1-Leaf-03      L1   Ethernet3          50:0:0:d5:5d:c0   UP    8           DC1-Leaf-03.0e
Spine2    default  DC1-Leaf-01      L1   Ethernet1          50:0:0:d7:ee:b    UP    8           DC1-Leaf-01.0f

```
- ###### Таблица маршрутизации:

```
DC1-Spine-02#sh ip route

VRF: default
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

 I L1     10.0.1.0/32 [115/21] via 10.2.2.0, Ethernet1
                               via 10.2.2.2, Ethernet2
                               via 10.2.2.4, Ethernet3
 C        10.0.2.0/32 is directly connected, Loopback0
 I L1     10.1.1.0/32 [115/11] via 10.2.2.0, Ethernet1
 I L1     10.1.2.0/32 [115/11] via 10.2.2.2, Ethernet2
 I L1     10.1.3.0/32 [115/11] via 10.2.2.4, Ethernet3
 I L1     10.2.1.0/31 [115/20] via 10.2.2.0, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.2.2, Ethernet2
 I L1     10.2.1.4/31 [115/20] via 10.2.2.4, Ethernet3
 C        10.2.2.0/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet3

```
- ###### LSDB:
```
DC1-Spine-02#show isis database

IS-IS Instance: Spine2 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    DC1-Spine-01.00-00           39  25621   528    150 L2 <>
    DC1-Spine-02.00-00           36   9561  1072    150 L2 <>
    DC1-Leaf-02.00-00            30  46509   376    125 L1 <>
    DC1-Leaf-02.0d-00            21  47011   412     51 L1 <>
    DC1-Leaf-02.0e-00            20  16425   721     51 L1 <>
    DC1-Leaf-03.00-00            27  20323   769    125 L1 <>
    DC1-Leaf-03.0d-00            17  38403   703     51 L1 <>
    DC1-Leaf-03.0e-00            19   6539  1160     51 L1 <>
    DC1-Leaf-01.00-00            39  49107   902    125 L1 <>
    DC1-Leaf-01.0e-00            22  53161   478     51 L1 <>
    DC1-Leaf-01.0f-00            15  25641   800     51 L1 <>
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    DC1-Spine-02.00-00           50  59273   469    169 L2 <>

```
- ##### DC1-Leaf-01:
###### Ping DC1-Leaf-02:
```
DC1-Leaf-01#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=96.1 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=88.5 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=86.7 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=81.0 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=82.3 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 81.000/86.959/96.107/5.361 ms, pipe 5, ipg/ewma 12.604/91.210 ms

```
###### Ping DC1-Leaf-03:
```
DC1-Leaf-01#ping 10.1.3.0
PING 10.1.3.0 (10.1.3.0) 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=32.7 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=30.8 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=23.2 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=26.6 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=21.5 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 21.535/27.014/32.785/4.287 ms, pipe 3, ipg/ewma 21.095/29.633 ms

```
- ##### DC1-Leaf-02:
###### Ping DC1-Leaf-01: 
 ```
DC1-Leaf-02#ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=39.6 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=29.6 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=40.3 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=48.3 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=26.0 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 81ms
rtt min/avg/max/mdev = 26.039/36.800/48.332/8.008 ms, pipe 4, ipg/ewma 20.427/38.131 ms

```
###### Ping DC1-Leaf-03:
```
DC1-Leaf-02#ping 10.1.3.0
PING 10.1.3.0 (10.1.3.0) 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=32.8 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=30.3 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=34.5 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=20.7 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=17.4 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 94ms
rtt min/avg/max/mdev = 17.484/27.199/34.537/6.824 ms, pipe 3, ipg/ewma 23.686/29.581 ms

```
- ##### DC1-Leaf-03:
###### Ping DC1-Leaf-01:
```
DC1-Leaf-03#ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=45.7 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=37.6 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=43.7 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=38.8 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=23.4 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 23.461/37.904/45.730/7.812 ms, pipe 4, ipg/ewma 21.059/41.335 ms

```
###### Ping DC1-Leaf-02:
```
DC1-Leaf-03#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=38.9 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=24.4 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=25.7 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=36.6 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=26.3 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 100ms
rtt min/avg/max/mdev = 24.423/30.410/38.922/6.093 ms, pipe 3, ipg/ewma 25.226/34.621 ms
DC1-Leaf-03#
```


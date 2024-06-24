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
service routing protocols model multi-agent
!
hostname DC1-Leaf-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC1-Spine-01 int Eth1 ###
   no switchport
   ip address 10.2.1.0/31
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet2
   description ### Link To DC1-Spine2 int Eth2 ###
   no switchport
   ip address 10.2.2.0/31
   bfd interval 50 min-rx 50 multiplier 3
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
ip prefix-list loopback
   seq 10 permit 10.1.0.0/16 ge 16
!
route-map loopback permit 10
   match ip address prefix-list loopback
!
router bgp 65001
   router-id 10.1.1.0
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES allowas-in 1
   neighbor SPINES password 7 hTM54+XaNuhrPT95b9S4xg==
   neighbor SPINES send-community
   neighbor SPINES maximum-routes 12000
   neighbor 10.2.1.1 peer group SPINES
   neighbor 10.2.2.1 peer group SPINES
   redistribute connected route-map loopback
   !
   address-family ipv4
      neighbor SPINES activate
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
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.2/31
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.2/31
   bfd interval 50 min-rx 50 multiplier 3
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
ip prefix-list loopback
   seq 10 permit 10.1.0.0/16 ge 16
!
route-map loopback permit 10
   match ip address prefix-list loopback
!
router bgp 65002
   router-id 10.1.2.0
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES allowas-in 1
   neighbor SPINES password 7 hTM54+XaNuhrPT95b9S4xg==
   neighbor SPINES send-community
   neighbor SPINES maximum-routes 12000
   neighbor 10.2.1.3 peer group SPINES
   neighbor 10.2.2.3 peer group SPINES
   redistribute connected route-map loopback
   !
   address-family ipv4
      neighbor SPINES activate
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
interface Ethernet1
   description ### Link To DC1-Spine-01 int Eth2 ###
   no switchport
   ip address 10.2.1.4/31
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet2
   description ### Link To DC1-Spine-02 int Eth2 ###
   no switchport
   ip address 10.2.2.4/31
   bfd interval 50 min-rx 50 multiplier 3
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
ip prefix-list loopback
   seq 10 permit 10.1.0.0/16 ge 16
!
route-map loopback permit 10
   match ip address prefix-list loopback
!
router bgp 65003
   router-id 10.1.3.0
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES allowas-in 1
   neighbor SPINES password 7 hTM54+XaNuhrPT95b9S4xg==
   neighbor SPINES send-community
   neighbor SPINES maximum-routes 12000
   neighbor 10.2.1.5 peer group SPINES
   neighbor 10.2.2.5 peer group SPINES
   redistribute connected route-map loopback
   !
   address-family ipv4
      neighbor SPINES activate
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
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth1 ###
   no switchport
   ip address 10.2.1.3/31
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth1 ###
   no switchport
   ip address 10.2.1.5/31
   bfd interval 50 min-rx 50 multiplier 3
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
route-map loopback permit 10
   match interface Loopback0
!
router bgp 65000
   neighbor 10.2.1.0 remote-as 65001
   neighbor 10.2.1.0 bfd
   neighbor 10.2.1.0 password 7 xKpH9TKUNMWUDKMAlOV8PQ==
   neighbor 10.2.1.2 remote-as 65002
   neighbor 10.2.1.2 bfd
   neighbor 10.2.1.2 password 7 UsRpRE+61x2NeyMRcL2aog==
   neighbor 10.2.1.4 remote-as 65003
   neighbor 10.2.1.4 bfd
   neighbor 10.2.1.4 password 7 IfEQGA02F3azK08/0FfFZA==
   redistribute connected route-map loopback
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
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet2
   description ### Link To DC1-Leaf-02 int Eth2 ###
   no switchport
   ip address 10.2.2.3/31
   bfd interval 50 min-rx 50 multiplier 3
!
interface Ethernet3
   description ### Link To DC1-Leaf-03 int Eth2 ###
   no switchport
   ip address 10.2.2.5/31
   bfd interval 50 min-rx 50 multiplier 3
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
route-map loopback permit 10
   match interface Loopback0
!
router bgp 65000
   neighbor 10.2.2.0 remote-as 65001
   neighbor 10.2.2.0 bfd
   neighbor 10.2.2.0 password 7 fPGf8Fh3w7RAB0kEZSeSTQ==
   neighbor 10.2.2.2 remote-as 65002
   neighbor 10.2.2.2 bfd
   neighbor 10.2.2.2 password 7 am7s9DNpBwwL/oURqvNM6w==
   neighbor 10.2.2.4 remote-as 65003
   neighbor 10.2.2.4 bfd
   neighbor 10.2.2.4 password 7 vbRJbblAXEZufc+oT/yH5w==
   redistribute connected route-map loopback
!
end

```  
----------------------------
#### Проверка BGP соседства, содержимое таблиц машрутизации:
- ##### DC1-Leaf-01:
- ###### BGP соседство:
 ```
DC1-Leaf-01#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1 4 65000           6938      8270    0    0 00:03:17 Estab   3      3
  10.2.2.1 4 65000           6817      8347    0    0 00:03:17 Estab   3      3

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
 B E      10.1.1.0/32 [200/0] via 10.2.1.0, Ethernet1
 B E      10.1.2.0/32 [200/0] via 10.2.1.2, Ethernet2
 B E      10.1.3.0/32 [200/0] via 10.2.1.4, Ethernet3
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3

```
- ###### Проверка связности с другими Leaf:

```
DC1-Leaf-01#ping 10.1.2.0 source loopback 1
PING 10.1.2.0 (10.1.2.0) from 10.1.1.0 : 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=35.6 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=28.7 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=36.8 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=32.3 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=26.0 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 69ms
rtt min/avg/max/mdev = 26.059/31.927/36.839/4.087 ms, pipe 4, ipg/ewma 17.300/33.642 ms
DC1-Leaf-01#ping 10.1.3.0 source loopback 1
PING 10.1.3.0 (10.1.3.0) from 10.1.1.0 : 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=21.3 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=17.0 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=33.4 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=19.3 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=14.2 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 79ms
rtt min/avg/max/mdev = 14.263/21.092/33.454/6.620 ms, pipe 2, ipg/ewma 19.894/21.040 ms

```
- ##### DC1-Leaf-02:
- ###### BGP соседство:
```
DC1-Leaf-02#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.2.0, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.3 4 65000           6944      8283    0    0 00:04:19 Estab   3      3
  10.2.2.3 4 65000           6905      8496    0    0 00:04:19 Estab   3      3

```
- ###### Таблица маршрутизации:
```
DC1-Leaf-02#sh ip route

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

 B E      10.0.1.0/32 [200/0] via 10.2.1.3, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.3, Ethernet2
 B E      10.1.1.0/32 [200/0] via 10.2.1.3, Ethernet1
                              via 10.2.2.3, Ethernet2
 C        10.1.2.0/32 is directly connected, Loopback1
 B E      10.1.3.0/32 [200/0] via 10.2.1.3, Ethernet1
                              via 10.2.2.3, Ethernet2
 C        10.2.1.2/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2

```
- ###### Проверка связности с другими Leaf:

```
DC1-Leaf-02#ping 10.1.1.0 source loopback 1
PING 10.1.1.0 (10.1.1.0) from 10.1.2.0 : 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=19.7 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=24.2 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=21.1 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=22.7 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=13.6 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 79ms
rtt min/avg/max/mdev = 13.662/20.310/24.279/3.659 ms, pipe 2, ipg/ewma 19.960/19.812 ms
DC1-Leaf-02#ping 10.1.3.0 source loopback 1
PING 10.1.3.0 (10.1.3.0) from 10.1.2.0 : 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=21.0 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=24.4 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=29.5 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=21.4 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=30.1 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 21.007/25.331/30.142/3.879 ms, pipe 2, ipg/ewma 19.131/23.309 ms


```
- ##### DC1-Leaf-03:
- ###### BGP соседство:
 ```
DC1-Leaf-03#sh ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.3.0, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.5 4 65000           6979      8424    0    0 00:00:59 Estab   3      3
  10.2.2.5 4 65000           6814      8351    0    0 00:05:55 Estab   3      3

```
- ###### Таблица маршрутизации:
```
DC1-Leaf-03#sh ip route

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

 B E      10.0.1.0/32 [200/0] via 10.2.1.5, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.5, Ethernet2
 B E      10.1.1.0/32 [200/0] via 10.2.1.5, Ethernet1
                              via 10.2.2.5, Ethernet2
 B E      10.1.2.0/32 [200/0] via 10.2.1.5, Ethernet1
                              via 10.2.2.5, Ethernet2
 C        10.1.3.0/32 is directly connected, Loopback1
 C        10.2.1.4/31 is directly connected, Ethernet1
 C        10.2.2.4/31 is directly connected, Ethernet2

```
- ###### Проверка связности с другими Leaf:

```
DC1-Leaf-03#ping 10.1.1.0 source loopback 1
PING 10.1.1.0 (10.1.1.0) from 10.1.3.0 : 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=29.6 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=23.8 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=21.7 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=25.4 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=16.7 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 16.730/23.497/29.684/4.267 ms, pipe 3, ipg/ewma 21.239/26.355 ms
DC1-Leaf-03#ping 10.1.2.0 source loopback 1
PING 10.1.2.0 (10.1.2.0) from 10.1.3.0 : 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=64.0 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=61.9 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=63.3 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=57.3 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=57.5 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 57.384/60.845/64.017/2.850 ms, pipe 5, ipg/ewma 10.762/62.243 ms


```

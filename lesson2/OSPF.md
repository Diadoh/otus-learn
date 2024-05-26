# Домашнее задание №2
## Underlay. OSPF
### Цели:
- Настроить OSPF для Underlay сети
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
![123](/lesson2/DC-Topology_OSPF.png)

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
   description ### Client ###
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
!
interface Loopback1
   description ### Lo1 ###
   ip address 10.1.2.0/32
!
interface Management1
!
ip routing
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
- ##### DC1-Spine-01:
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
#### Проверка OSPF соседства и IP связности:
- ##### DC1-Spine-01:
 ```
DC1-Spine-01#sh ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.1.1.0        1        default  0   FULL                   00:00:35    10.2.1.0        Ethernet1
10.1.2.0        1        default  0   FULL                   00:00:38    10.2.1.2        Ethernet2
10.1.3.0        1        default  0   FULL                   00:00:30    10.2.1.4        Ethernet3
```
- ##### DC1-Spine-02:
```
DC1-Spine-02#sh ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.1.1.0        1        default  0   FULL                   00:00:29    10.2.2.0        Ethernet1
10.1.2.0        1        default  0   FULL                   00:00:32    10.2.2.2        Ethernet2
10.1.3.0        1        default  0   FULL                   00:00:31    10.2.2.4        Ethernet3
```
- ##### DC1-Leaf-01:
###### Ping DC1-Leaf-02:
```
DC1-Leaf-01#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=83.6 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=77.1 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=91.3 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=90.5 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=100 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 77.162/88.535/100.035/7.718 ms, pipe 5, ipg/ewma 11.127/86.626 ms
```
###### Ping DC1-Leaf-03:
```
DC1-Leaf-01#ping 10.1.3.0
PING 10.1.3.0 (10.1.3.0) 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=44.7 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=33.2 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=42.1 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=36.5 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=21.3 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 78ms
rtt min/avg/max/mdev = 21.352/35.588/44.703/8.183 ms, pipe 4, ipg/ewma 19.551/39.683 ms
```
- ##### DC1-Leaf-02:
###### Ping DC1-Leaf-01: 
 ```
DC1-Leaf-02#ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=58.2 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=56.5 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=51.1 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=49.9 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=44.5 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 44.563/52.103/58.233/4.891 ms, pipe 5, ipg/ewma 10.900/54.807 ms
```
###### Ping DC1-Leaf-03:
```
DC1-Leaf-02#ping 10.1.3.0
PING 10.1.3.0 (10.1.3.0) 72(100) bytes of data.
80 bytes from 10.1.3.0: icmp_seq=1 ttl=63 time=75.5 ms
80 bytes from 10.1.3.0: icmp_seq=2 ttl=63 time=69.2 ms
80 bytes from 10.1.3.0: icmp_seq=3 ttl=63 time=66.6 ms
80 bytes from 10.1.3.0: icmp_seq=4 ttl=63 time=61.5 ms
80 bytes from 10.1.3.0: icmp_seq=5 ttl=63 time=56.1 ms

--- 10.1.3.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 56.166/65.834/75.553/6.611 ms, pipe 5, ipg/ewma 10.889/70.218 ms
```
- ##### DC1-Leaf-03:
###### Ping DC1-Leaf-01:
```
DC1-Leaf-03#ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=33.8 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=39.1 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=40.9 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=35.4 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=25.4 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 116ms
rtt min/avg/max/mdev = 25.471/34.980/40.989/5.385 ms, pipe 3, ipg/ewma 29.029/34.118 ms
```
###### Ping DC1-Leaf-02:
```
DC1-Leaf-03#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=89.7 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=83.0 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=132 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=130 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=130 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 83.059/113.140/132.578/21.941 ms, pipe 5, ipg/ewma 12.351/102.769 ms
```

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

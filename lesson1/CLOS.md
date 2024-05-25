### Цели:
- Собрать схему CLOS;
- Распределить адресное пространство.

### Решение:
#### IP PLAN:

|DC|Network|Description|
|-------|-----------|-----------|
|DC1  |10.0.0.0/16| Loopback0-Spine|
|DC1  |10.1.0.0/16| Loopback1-Leaf|
|DC1  |10.2.0.0/16| P2P links|
|DC1  |10.3.0.0/16| Reserve|
|DC1  |10.4.0.0/14| Services|

  
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
![123](/lesson1/DC-Topology.png)

#### Конфигурация устройств:
- ##### DC1-Leaf-01:
----------------------------
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
    !
    interface Ethernet2
       description ### Link To DC1-Spine2 int Eth2 ###
       no switchport
       ip address 10.2.2.0/31
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
    end

- ##### DC1-Leaf-02:
- ##### DC1-Leaf-03:
- ##### DC1-Spine-01:
- ##### DC1-Spine-01:

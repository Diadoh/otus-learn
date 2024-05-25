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
| DC1-Leaf1   | Lo1   |10.1.1.0/32    |Loopback0|
| DC1-Leaf1   | Eth1   |10.2.1.0/31  |Link to DC1-Spine1|
| DC1-Leaf1   | Eth2   |10.2.2.0/31  |Link to DC1-Spine2|
| DC1-Leaf2   | Lo1   |10.1.2.0/32    |Loopback0|
| DC1-Leaf2   | Eth1   |10.2.1.2/31  |Link to DC1-Spine1|
| DC1-Leaf2   | Eth2   |10.2.2.2/31  |Link to DC1-Spine2|
| DC1-Leaf3   | Lo1   |10.1.3.0/32    |Loopback0|
| DC1-Leaf3   | Eth1   |10.2.1.4/31  |Link to DC1-Spine1|
| DC1-Leaf3   | Eth2   |10.2.2.4/31  |Link to DC1-Spine2|
| DC1-Spine1   | Lo1   |10.0.1.0/32    |Loopback0|
| DC1-Spine1   | Eth1   |10.2.1.1/31  |Link to DC1-Leaf1|
| DC1-Spine1   | Eth2   |10.2.1.3/31  |Link to DC1-Leaf2|
| DC1-Spine1   | Eth3   |10.2.1.5/31  |Link to DC1-Leaf3|
| DC1-Spine2   | Lo1   |10.0.2.0/32    |Loopback0|
| DC1-Spine2   | Eth1   |10.2.2.1/31  |Link to DC1-Leaf1|
| DC1-Spine2   | Eth2   |10.2.2.3/31  |Link to DC1-Leaf2|
| DC1-Spine2   | Eth3   |10.2.2.5/31  |Link to DC1-Leaf3|


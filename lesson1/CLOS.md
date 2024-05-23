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
| DC1-Leaf1   | Lo1   |10.0.1.0/32    |Loopback0|
| Ячейка 3    | Ячейка 4   |Ячейка 4   |Ячейка 4|

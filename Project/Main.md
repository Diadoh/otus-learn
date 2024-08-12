
|DC   | NETWORK  |  Description |
| ------------ | ------------ | ------------ |
| DC1  | 172.16.0.0/16  |  Loopback1-Spine |
|  DC1 |  172.17.0.0/16 | Loopback1-Leaf  |
| DC1  |  172.18.0.0/16 | P2P links  |
| DC1  |  172.19.0.0/16 | Reserve  |
| DC1  |  172.20.0.0/16 |Services   |
| DC2  |  172.21.0.0/16 | Loopback1-Spine |
| DC2  | 172.22.0.0/16  | Loopback1-Leaf  |
| DC2  |  172.23.0.0/16 | P2P links  |
| DC2  | 172.24.0.0/16  | Reserve  |
| DC2  |  172.25.0.0/16 | Services  |
| DC1-DC2  |172.26.0.0/16   | P2P links DCs   |
| Site A  |  10.0.0.0/12 | Production  |
| Site A  | 10.16.0.0/16  |VMWare Hosts   |
| Site B | 10.0.0.0/12  | Production  |
|Site B   | 10.16.0.0/16  | VMWare Hosts  |
|Site B   | 10.16.0.0/16  |VMWare Hosts   |

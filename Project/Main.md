


| DC      | NETWORK       | Description     |
|---------|---------------|-----------------|
| DC1     | 172.16.0.0/16 | Loopback1-Spine |
| DC1     | 172.17.0.0/16 | Loopback1-Leaf  |
| DC1     | 172.18.0.0/16 | P2P links       |
| DC1     | 172.19.0.0/16 | Reserve         |
| DC1     | 172.20.0.0/16 | Services        |
| DC2     | 172.21.0.0/16 | Loopback1-Spine |
| DC2     | 172.22.0.0/16 | Loopback1-Leaf  |
| DC2     | 172.23.0.0/16 | P2P links       |
| DC2     | 172.24.0.0/16 | Reserve         |
| DC2     | 172.25.0.0/16 | Services        |
| DC1-DC2 | 172.26.0.0/16 | P2P links DCs   |
| Site A  | 10.0.0.0/12   | Production      |
| Site A  | 10.16.0.0/16  | VMWare Hosts    |
| Site B  | 10.0.0.0/12   | Production      |
| Site B  | 10.16.0.0/16  | VMWare Hosts    |
| Site C  | 10.16.0.0/16  | VMWare Hosts    |

| Hostname     | Interface         | IP/Mask        | Description                                   |
|--------------|-------------------|----------------|-----------------------------------------------|
| DC1-TORSW-01 | Lo1               | 172.16.0.1/32  | For BGP/EVPN                                  |
| DC1-TORSW-01 | Eth1              | 172.18.0.0/31  | Link to DC1-CSW-01 int Eth1                   |
| DC1-TORSW-01 | Eth2              | 172.18.0.2/31  | Link to DC1-CSW-02 int Eth1                   |
| DC1-TORSW-01 | Eth4              | -              | Link to DC1-FW-01 int Port 2                  |
| DC1-TORSW-01 | Eth4 (VLAN 1001)  | 10.0.0.3/29    | Link to DC1-FW-01 int VLAN 1001 vrf PROD      |
| DC1-TORSW-01 | Eth4 (VLAN 1002)  | 10.16.0.3/29   | Link to DC1-FW-01 int VLAN 1002 vrf VMWARE    |
| DC1-TORSW-01 | Eth5              | 172.18.0.16/31 | Link to CH1-CSW-01 int Eth1                   |
| DC1-TORSW-02 | Lo1               | 172.16.0.2/32  | For BGP/EVPN                                  |
| DC1-TORSW-02 | Eth1              | 172.18.0.6/31  | Link to DC1-CSW-01 int Eth2                   |
| DC1-TORSW-02 | Eth2              | 172.18.0.8/31  | Link to DC1-CSW-02 int Eth2                   |
| DC1-TORSW-03 | Lo1               | 172.16.0.3/32  | For BGP/EVPN                                  |
| DC1-TORSW-03 | Eth1              | 172.18.0.10/31 | Link to DC1-CSW-01 int Eth3                   |
| DC1-TORSW-03 | Eth2              | 172.18.0.12/31 | Link to DC1-CSW-02 int Eth3                   |
| DC1-TORSW-04 | Lo1               | 172.16.0.4/132 | For BGP/EVPN                                  |
| DC1-TORSW-04 | Eth1              | 172.18.0.14/31 | Link to DC1-CSW-01 int Eth4                   |
| DC1-TORSW-04 | Eth2              | 172.18.0.16/31 | Link to DC1-CSW-02 int Eth4                   |
| DC1-CSW-01   | Lo1               | 172.17.0.1/32  | For BGP/EVPN                                  |
| DC1-CSW-01   | Eth1              | 172.18.0.1/31  | Link to DC1-TORSW-01 int Eth1                 |
| DC1-CSW-01   | Eth2              | 172.18.0.7/31  | Link to DC1-TORSW-02int Eth1                  |
| DC1-CSW-01   | Eth3              | 172.18.0.11/31 | Link to DC1-TORSW-03 int Eth1                 |
| DC1-CSW-01   | Eth4              | 172.18.0.15/31 | Link to DC1-TORSW-04 int Eth1                 |
| DC1-CSW-01   | Eth5              | 172.26.0.0/31  | Link to DC2-CSW-01 int Eth5                   |
| DC1-CSW-01   | Eth6              | 172.26.0.2/31  | Link to DC2-CSW-01 int Eth6                   |
| DC1-CSW-01   | Eth7              | 172.26.0.4/31  | Link to DC2-CSW-02 int Eth7                   |
| DC1-CSW-01   | Eth8              | 172.26.0.6/31  | Link to DC2-CSW-02 int Eth8                   |
| DC1-CSW-02   | Lo1               | 172.17.0.2/32  | For BGP/EVPN                                  |
| DC1-CSW-02   | Eth1              | 172.18.0.3/31  | Link to DC1-TORSW-01 int Eth2                 |
| DC1-CSW-02   | Eth2              | 172.18.0.9/31  | Link to DC1-TORSW-02int Eth2                  |
| DC1-CSW-02   | Eth3              | 172.18.0.13/31 | Link to DC1-TORSW-03 int Eth2                 |
| DC1-CSW-02   | Eth4              | 172.18.0.17/31 | Link to DC1-TORSW-04 int Eth2                 |
| DC1-CSW-02   | Eth5              | 172.26.0.8/31  | Link to DC2-CSW-02 int Eth5                   |
| DC1-CSW-02   | Eth6              | 172.26.0.10/31 | Link to DC2-CSW-02int Eth6                    |
| DC1-CSW-02   | Eth7              | 172.26.0.12/31 | Link to DC2-CSW-01 int Eth7                   |
| DC1-CSW-02   | Eth8              | 172.26.0.14/31 | Link to DC2-CSW-01 int Eth8                   |
| DC1-FW-01    | Port2             | -              | Link to DC1-TORSW-01 int Eth4                 |
| DC1-FW-01    | Port2 (VLAN 1001) | 10.0.0.1/29    | Link to DC1-TORSW-01 int VLAN 1001 vrf PROF   |
| DC1-FW-01    | Port2 (VLAN 1002) | 10.16.0.1/29   | Link to DC1-TORSW-01 int VLAN 1002 vrf VMWARE |
| DC2-TORSW-01 | Lo1               | 172.22.0.1/32  | For BGP/EVPN                                  |
| DC2-TORSW-01 | Eth1              | 172.23.0.0/31  | Link to DC2-CSW-01 int Eth1                   |
| DC2-TORSW-01 | Eth2              | 172.23.0.2/31  | Link to DC2-CSW-02 int Eth1                   |
| DC2-TORSW-01 | Eth4              | -              | Link to DC2-FW-01 int Port 2                  |
| DC2-TORSW-01 | Eth4 (VLAN 1001)  | 10.0.0.11/29   | Link to DC2-FW-01 int VLAN 1001 vrf PROD      |
| DC2-TORSW-01 | Eth4 (VLAN 1002)  | 10.16.0.11/29  | Link to DC2-FW-01 int VLAN 1002 vrf VMWARE    |
| DC2-TORSW-01 | Eth5              | 172.23.0.16/31 | Link to CH1-CSW-01 int Eth2                   |
| DC2-TORSW-02 | Lo1               | 172.22.0.2/32  | For BGP/EVPN                                  |
| DC2-TORSW-02 | Eth1              | 172.23.0.4/31  | Link to DC2-CSW-01 int Eth2                   |
| DC2-TORSW-02 | Eth2              | 172.23.0.6/31  | Link to DC2-CSW-02 int Eth2                   |
| DC2-TORSW-03 | Lo1               | 172.22.0.3/32  | For BGP/EVPN                                  |
| DC2-TORSW-03 | Eth1              | 172.23.0.8/31  | Link to DC2-CSW-01 int Eth3                   |
| DC2-TORSW-03 | Eth2              | 172.23.0.10/31 | Link to DC2-CSW-02 int Eth3                   |
| DC2-TORSW-04 | Lo1               | 172.22.0.4/32  | For BGP/EVPN                                  |
| DC2-TORSW-04 | Eth1              | 172.23.0.12/31 | Link to DC2-CSW-01 int Eth4                   |
| DC2-TORSW-04 | Eth2              | 172.23.0.14/31 | Link to DC2-CSW-02 int Eth4                   |
| DC2-CSW-01   | Lo1               | 172.21.0.1/32  | For BGP/EVPN                                  |
| DC2-CSW-01   | Eth1              | 172.23.0.1/31  | Link to DC2-TORSW-01 int Eth1                 |
| DC2-CSW-01   | Eth2              | 172.23.0.5/31  | Link to DC2-TORSW-02int Eth1                  |
| DC2-CSW-01   | Eth3              | 172.23.0.9/31  | Link to DC2-TORSW-03 int Eth1                 |
| DC2-CSW-01   | Eth4              | 172.23.0.13/31 | Link to DC2-TORSW-04 int Eth1                 |
| DC2-CSW-01   | Eth5              | 172.26.0.1/31  | Link to DC1-CSW-01 int Eth5                   |
| DC2-CSW-01   | Eth6              | 172.26.0.3/31  | Link to DC1-CSW-01 int Eth6                   |
| DC2-CSW-01   | Eth7              | 172.26.0.13/31 | Link to DC1-CSW-02 int Eth7                   |
| DC2-CSW-01   | Eth8              | 172.26.0.15/31 | Link to DC1-CSW-02 int Eth8                   |
| DC2-CSW-02   | Lo1               | 172.21.0.2/32  | For BGP/EVPN                                  |
| DC2-CSW-02   | Eth1              | 172.23.0.3/31  | Link to DC2-TORSW-01 int Eth2                 |
| DC2-CSW-02   | Eth2              | 172.23.0.7/31  | Link to DC2-TORSW-02int Eth2                  |
| DC2-CSW-02   | Eth3              | 172.23.0.11/31 | Link to DC2-TORSW-03 int Eth2                 |
| DC2-CSW-02   | Eth4              | 172.23.0.15/31 | Link to DC2-TORSW-04 int Eth2                 |
| DC2-CSW-02   | Eth5              | 172.26.0.9/31  | Link to DC1-CSW-02 int Eth5                   |
| DC2-CSW-02   | Eth6              | 172.26.0.11/31 | Link to DC1-CSW-02int Eth6                    |
| DC2-CSW-02   | Eth7              | 172.26.0.5/31  | Link to DC1-CSW-01 int Eth7                   |
| DC2-CSW-02   | Eth8              | 172.26.0.7/31  | Link to DC1-CSW-01 int Eth8                   |
| DC2-FW-01    | Port2             | -              | Link to DC2-TORSW-01 int Eth4                 |
| DC2-FW-01    | Port2 (VLAN 1001) | 10.0.0.9/29    | Link to DC2-TORSW-01 int VLAN 1001 vrf PROF   |
| DC2-FW-01    | Port2 (VLAN 1002) | 10.16.0.9/29   | Link to DC2-TORSW-01 int VLAN 1002 vrf VMWARE |
| CH1-CSW-01   | Lo1               | 172.19.0.1/32  | For BGP/EVPN                                  |
| CH1-CSW-01   | Eth1              | 172.18.0.17/31 | Link to DC1-TORSW-01 int Eth5                 |
| CH1-CSW-01   | Eth2              | 172.23.0.17/31 | Link to DC2-TORSW-01 int Eth5                 |


| Hostname     | Interface | IP/Mask      | Description |
|--------------|-----------|--------------|-------------|
| DC1-ESXi-01  | VLAN 10   | 10.0.10.11   | vrf PROD    |
| DC1-ESXi-01  | VLAN 11   | 10.0.11.11   | vrf PROD    |
| DC1-ESXi-01  | VLAN 20   | 10.0.20.11   | vrf PROD    |
| DC1-ESXi-01  | VLAN 101  | 10.16.101.11 | vrf VMWARE  |
| DC1-ESXi-02  | VLAN 10   | 10.0.10.12   | vrf PROD    |
| DC1-ESXi-02  | VLAN 11   | 10.0.11.12   | vrf PROD    |
| DC1-ESXi-02  | VLAN 20   | 10.0.20.12   | vrf PROD    |
| DC1-ESXi-02  | VLAN 101  | 10.16.101.12 | vrf VMWARE  |
| DC1-ESXi-03  | VLAN 10   | 10.0.10.13   | vrf PROD    |
| DC1-ESXi-03  | VLAN 11   | 10.0.11.13   | vrf PROD    |
| DC1-ESXi-03  | VLAN 20   | 10.0.20.13   | vrf PROD    |
| DC1-ESXi-03  | VLAN 101  | 10.16.101.13 | vrf VMWARE  |
| DC1-ESXi-04  | VLAN 10   | 10.0.10.14   | vrf PROD    |
| DC1-ESXi-04  | VLAN 11   | 10.0.11.14   | vrf PROD    |
| DC1-ESXi-04  | VLAN 20   | 10.0.20.14   | vrf PROD    |
| DC1-ESXi-04  | VLAN 101  | 10.16.101.14 | vrf VMWARE  |
| DC2-ESXi-01  | VLAN 10   | 10.0.10.21   | vrf PROD    |
| DC2-ESXi-01  | VLAN 11   | 10.0.11.21   | vrf PROD    |
| DC2-ESXi-01  | VLAN 20   | 10.0.20.21   | vrf PROD    |
| DC2-ESXi-01  | VLAN 102  | 10.16.102.2  | vrf VMWARE  |
| DC2-ESXi-02  | VLAN 10   | 10.0.10.22   | vrf PROD    |
| DC2-ESXi-02  | VLAN 11   | 10.0.11.22   | vrf PROD    |
| DC2-ESXi-02  | VLAN 20   | 10.0.20.22   | vrf PROD    |
| DC2-ESXi-02  | VLAN 102  | 10.16.102.22 | vrf VMWARE  |
| DC2-ESXi-03  | VLAN 10   | 10.0.10.23   | vrf PROD    |
| DC2-ESXi-03  | VLAN 11   | 10.0.11.23   | vrf PROD    |
| DC2-ESXi-03  | VLAN 20   | 10.0.20.23   | vrf PROD    |
| DC2-ESXi-03  | VLAN 102  | 10.16.102.23 | vrf VMWARE  |
| DC2-ESXi-04  | VLAN 10   | 10.0.10.24   | vrf PROD    |
| DC2-ESXi-04  | VLAN 11   | 10.0.11.24   | vrf PROD    |
| DC2-ESXi-04  | VLAN 20   | 10.0.20.24   | vrf PROD    |
| DC2-ESXi-04  | VLAN 102  | 10.16.102.24 | vrf VMWARE  |
| Witness Host | VLAN 103  | 10.16.103.10 | vrf VMWARE  |
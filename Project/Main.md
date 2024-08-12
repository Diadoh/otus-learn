# Проектная пабота
## Проектирование распределенной сети ЦОД с использованием VxLAN EVPN для перехода на использование VMWare Stretched cluster.
### Цели:
- Спроектировать распределенную сеть ЦОД для внедрения VMWare Stretched Cluster;
- Реализовать защиту продуктивного сегмента и сегмента управления средствами МСЭ;
- Получить практический опыт конфигурации и использования технологий VXLAN/EVPN;
- Реализовать свой взгляд на реализацию обеспечения работы VMWare Stretched Cluster и отказоустойчивость распредленных ЦОД.

### Задачи:
- Выработать решение для обеспчения работы VMWare Stretched Cluster;
- Обеспечить защиту продуктивного сегмента и сегмента управления средствами МСЭ

### Бизнес-задачи:
- В организации уже используется VMWare vSphere;
- Возможность создать кластер хранения данных и вычислительных мощностей, решая проблему выхода из строя одного из DC (блек-аут);
- Гибкий перенос инстансов виртуальных машин, распределение нагрузки;
- Реализация гиперконвергентной инфраструктуры.

  
### IP-plan segments
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

### IP-plan Leaf-Spine/Firewall

#### DC - Data Center
#### CH - Campus
#### TORSW = Leaf - Коммутатора доступа ЦОД
#### CSW = Spine - Коммутатор ядра ЦОД
#### FW = Firewall - Межсетевой экран

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

### IP-plan ESXi Hosts
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

### Общая схема организации VMWare Stretched cluster
![VMWARE1.png](VMWARE1.png)
### Ограничения VMWare Stretched cluster
![VMWARE2.png](VMWARE2.png)
#### Для работы VMWare Stretched cluster требуется 2-е площадки где будeт распологаться кластеры ESXi и Witness (Свидетель).

- Обязательное наличие отдельной площадки, на которой будет располагаться Witness Host. Данная площадка должна быть связана хорошими каналами по IP с обоими Дата Центрами, где будут находиться ноды Stretched VSAN кластера. Именно с помощью данного этого дополнительного элемента управления кластер сможет понимать какая его часть находится в работоспособном состоянии, а какая нет. Так же надо понимать, что Witness Host может быть в виде гипервизора ESXi, либо в виде виртуальной машины, которая работает в среде VMware vSphere;
- Наличие хороших каналов связи между двумя площадками, где будут располагаться ноды растянутого VSAN кластера. Так жесткими требованиям при этом является наличие 10 гигабитной полосы пропускания между обоими основными площадками, а также величина задержек менее 5 ms RTT (round trip time) на этом пути. Эти требования ограничивают возможности данной технологии одной метрополией;
- Пропускная способность не менее 10 Гбит/c между основным площадками, не менее 5 ms RTT. Для каждых 1000 элементов файловой системы VSAN требуется полоса пропускания в 2 Mb/s между нодами кластера и площадкой с Witness Host. RTT должны быть менее 200 ms при количестве нод менее 10 на каждой площадке, или менее 100 ms, при большем количестве нод кластера;
- Специальные VLAN выделенные для VSAN, vMotion и Management трафика должны быть связаны между обоими основными площадками. L2 или L3 связность. Witness трафик между всеми нодами кластера и Witness хостом должен соответствующим образом коммутироваться или маршрутизироваться.

### Топология сети

![Topology.png](Topology.png)


- В текущей топологии используется Leaf-Spine архитектура для построения сетей ЦОД, фактически имеется 2-ва POD разнесенных географически, связанных высокоскростнымм каналами связи c использованием CWDM;
- Выполнен отказ от использования стека коммутаров ядра для того чтобы получить отказоустойчивость на Control Plane;
- Не используется Super Spine посольку подразумевается ограничение кол-ва стоек на площадке 20-25 шт, а VMWARE Streched Cluster не подразумевает более 2-ух основных сайтов;
- Для базовой IP связности и настройки EVPN используется связка OSPF/iBGP для простоты настройки;
- Для L3 связности между продуктивным сегментом (PROD) и сегментов управления (VMWARE) используется eBGP пиринг с межсетевыми экранами FortiGate в соотвествующих VRF;
- Для подключения ESXi хостов агрегированым каналом к Leaf коммутаторам используется EVPN Multi-homing;
- Для того чтобы виртуальные машины могли беспрепятственно мигрировать и в случае отказа одного из ЦОД имели L3 связность реализуется распределнный шлюз с использованием EVPN VXLAN Anycast Gateway;
- Коммутатор CH1-CSW-01 с точки зрения модели является Leaf коммутатором, подарзумевается что площадака с Witness Host может использоваться для рсположения Backup серверов;
- Основным межсетевым экраном является DC1-FW-01, запасным DC2-FW-01. Однако принято решение не собирать их в Active-Passive кластер, а установить отдельно и с использованием атрибутов BGP раелизовать приоритетный выход во внешнюю сеть через DC1-FW-01. Подразумевается использование централизованого управления с ипользованием FortiManager.


### Cпецификация сетевого оборудования

|-|Назначение|Модель|Порты|Port-to-port Latency|Switching Capacity|Кол-во|
|:----|:----|:----|:----|:----|:----|:----|
|1|Core Switch(Spine)|Arista 7170-32C|32 100G QSFP|Under 1usec|6.4 Tbps|4|
|2|Top of Rack Switches (Leaf)|Arista 7135LB-48Y4C|48 SFP28, 4 QSFP100|From 5ns|3.2 Tbps|9|
|-|Назначение|Модель|Порты|Firewall Latency|Firewall Throughput|Кол-во|
|3|Firewall|FortiGate FG-600F|4x 25G SFP28, 4x 10GE SFP+ , 8x GE SFP, 18 x GE RJ45|4.12 μs / 2.5 μs 7|139 Gbps|2|

### Конфигурации:
- #### Data Center #1
<details>
  <summary>DC1-TORSW-01</summary>
  
```

DC1-TORSW-01(config-router-bgp)#sh run
! Command: show running-config
! device: DC1-TORSW-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-TORSW-01
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 101
   name net-101-vmware
!
vlan 1000
   name Transit-VMWARE
!
vlan 1001
   name Transit-PROD
!
vlan 1002
   name VMWARE
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0012:aaaa:aaaa:0010
      route-target import 01:12:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0012:aaaa:aaaa:0020
      route-target import 00:01:00:12:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.0/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.2/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet4
   mtu 9214
   switchport trunk allowed vlan 1001-1002
   switchport mode trunk
   no ip ospf neighbor bfd
!
interface Ethernet5
   description ### Link to CH1-CSW-01 int Eth1 ###
   mtu 9214
   no switchport
   ip address 172.18.0.16/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 0qDaaPrQfPk=
!
interface Ethernet6
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.16.0.1/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan101
   description ### GW net-101 ###
   vrf VMWARE
   ip address virtual 10.16.101.1/24
   ip virtual-router address 10.16.101.1/24
!
interface Vlan1001
   vrf PROD
   ip address 10.0.0.3/29
!
interface Vlan1002
   vrf VMWARE
   ip address 10.16.0.3/29
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 101 vni 10101
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
ip route vrf VMWARE 10.16.103.0/24 vtep 172.19.0.1 vni 20001 router-mac-address 50:00:00:88:2f:f3 210
!
router bgp 65001
   router-id 172.16.0.1
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.17.0.1 peer group SPINES
   neighbor 172.17.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 101
      rd 65001:101
      route-target both 65001:10101
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      neighbor 10.0.0.1 remote-as 65011
      neighbor 10.0.0.1 update-source Vlan1001
      aggregate-address 10.0.0.0/12 summary-only
      redistribute connected
      !
      address-family ipv4
         neighbor 10.0.0.1 activate
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      neighbor 10.16.0.1 remote-as 65011
      neighbor 10.16.0.1 update-source Vlan1002
      aggregate-address 10.16.0.0/12 summary-only
      redistribute connected
      !
      address-family ipv4
         neighbor 10.16.0.1 activate
!
router ospf 1
   router-id 172.16.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```

</details>

<details>
  <summary>DC1-TORSW-02</summary>

```

DC1-TORSW-02#sh run
! Command: show running-config
! device: DC1-TORSW-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-TORSW-02
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 101
   name net-101-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0012:aaaa:aaaa:0010
      route-target import 01:12:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0012:aaaa:aaaa:0020
      route-target import 00:01:00:12:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.6/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.8/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   no ip ospf neighbor bfd
!
interface Ethernet5
   no ip ospf neighbor bfd
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.16.0.2/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan101
   description ### GW net-101 ###
   vrf VMWARE
   ip address virtual 10.16.101.1/24
   ip virtual-router address 10.16.101.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 101 vni 10101
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
route-map OSPF_VXLAN permit 10
   set local-preference 25
!
route-map OSPF_VXLAN permit 15
   match route-type external
   set local-preference 25
!
router bgp 65001
   router-id 172.16.0.2
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.17.0.1 peer group SPINES
   neighbor 172.17.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 101
      rd 65001:101
      route-target both 65001:10101
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      redistribute connected
!
router ospf 1
   router-id 172.16.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```
  
</details>

<details>
  <summary>DC1-TORSW-03</summary>

```

DC1-TORSW-03(config)#sh run
! Command: show running-config
! device: DC1-TORSW-03 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-TORSW-03
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 101
   name net-101-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0034:aaaa:aaaa:0010
      route-target import 01:34:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0034:aaaa:aaaa:0020
      route-target import 01:34:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.10/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.12/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet4
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet5
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet6
   mtu 9214
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.16.0.3/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan101
   description ### GW net-101 ###
   vrf VMWARE
   ip address virtual 10.16.101.1/24
   ip virtual-router address 10.16.101.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 101 vni 10101
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.16.0.3
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.17.0.1 peer group SPINES
   neighbor 172.17.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 101
      rd 65001:101
      route-target both 65001:10101
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      redistribute connected
!
router ospf 1
   router-id 172.16.0.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```

</details>

<details>
  <summary>DC1-TORSW-04</summary>

```

! Command: show running-config
! device: DC1-TORSW-04 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-TORSW-04
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 101
   name net-101-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0034:aaaa:aaaa:0010
      route-target import 01:34:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0001:0034:aaaa:aaaa:0020
      route-target import 01:34:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.14/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.16/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   no ip ospf neighbor bfd
!
interface Ethernet5
   no ip ospf neighbor bfd
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.16.0.4/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan101
   description ### GW net-101 ###
   vrf VMWARE
   ip address 10.0.101.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 101 vni 10101
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.16.0.4
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.17.0.1 peer group SPINES
   neighbor 172.17.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 101
      rd 65001:101
      route-target both 65001:10101
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      redistribute connected
!
router ospf 1
   router-id 172.16.0.4
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```

</details>

<details>
  <summary>DC1-CSW-01</summary>

```

DC1-CSW-01#sh run
! Command: show running-config
! device: DC1-CSW-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-CSW-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC1-TORSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.1/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-TORSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.7/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   description ### Link to DC1-TORSW-03 ###
   mtu 9214
   no switchport
   ip address 172.18.0.11/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet4
   description ### Link to DC1-TORSW-04 ###
   mtu 9214
   no switchport
   ip address 172.18.0.15/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet5
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.0/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet6
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.2/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet7
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.4/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet8
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.6/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 EebseYaSiEVWe+SfUhSAzQ==
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.17.0.1/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 172.17.0.1
   timers bgp 10 30
   bgp cluster-id 0.0.0.1
   bgp listen range 172.16.0.0/16 peer-group LEAFS remote-as 65001
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback1
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community extended
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback1
   neighbor RR send-community
   neighbor WITNESS peer group
   neighbor WITNESS remote-as 65001
   neighbor WITNESS update-source Loopback1
   neighbor WITNESS send-community extended
   neighbor 172.17.0.2 peer group RR
   neighbor 172.19.0.1 peer group WITNESS
   neighbor 172.21.0.1 peer group RR
   neighbor 172.21.0.2 peer group RR
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
      neighbor WITNESS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor RR activate
      no neighbor WITNESS activate
!
router ospf 1
   router-id 172.17.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC1-CSW-02</summary>

```

DC1-CSW-02#sh run
! Command: show running-config
! device: DC1-CSW-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC1-CSW-02
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC1-TORSW-01 ###
   mtu 9214
   no switchport
   ip address 172.18.0.3/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC1-TORSW-02 ###
   mtu 9214
   no switchport
   ip address 172.18.0.9/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   description ### Link to DC1-TORSW-03 ###
   mtu 9214
   no switchport
   ip address 172.18.0.13/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet4
   description ### Link to DC1-TORSW-04 ###
   mtu 9214
   no switchport
   ip address 172.18.0.17/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet5
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.8/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet6
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.10/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet7
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.12/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet8
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.14/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 EebseYaSiEVWe+SfUhSAzQ==
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.17.0.2/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 172.17.0.2
   timers bgp 10 30
   bgp cluster-id 0.0.0.1
   bgp listen range 172.16.0.0/16 peer-group LEAFS remote-as 65001
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback1
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback1
   neighbor RR send-community
   neighbor WITNESS peer group
   neighbor WITNESS remote-as 65001
   neighbor WITNESS update-source Loopback1
   neighbor WITNESS send-community extended
   neighbor 172.17.0.1 peer group RR
   neighbor 172.19.0.1 peer group WITNESS
   neighbor 172.21.0.1 peer group RR
   neighbor 172.21.0.2 peer group RR
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
      neighbor WITNESS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor RR activate
      no neighbor WITNESS activate
!
router ospf 1
   router-id 172.17.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```

</details>

<details>
  <summary>DC1-FW-01</summary>

```

DC1-iFW-01 # config system interface

DC1-iFW-01 (interface) # edit "VLAN1002"

DC1-iFW-01 (VLAN1002) # show
config system interface
    edit "VLAN1002"
        set vdom "root"
        set vrf 10
        set ip 10.16.0.1 255.255.255.248
        set allowaccess ping
        set alias "VLAN1002"
        set role dmz
        set snmp-index 10
        set interface "port2"
        set vlanid 1002
    next
end

DC1-iFW-01 (VLAN1002) # next

DC1-iFW-01 (interface) # show
config system interface
    edit "port1"
        set vdom "root"
        set vrf 1
        set allowaccess ping
        set type physical
        set alias "1"
        set role lan
        set snmp-index 1
    next
    edit "port2"
        set vdom "root"
        set vrf 10
        set allowaccess ping
        set type physical
        set alias "TRUNK"
        set role dmz
        set snmp-index 2
    next
    edit "port3"
        set vdom "root"
        set type physical
        set snmp-index 3
    next
    edit "port4"
        set vdom "root"
        set ip 10.250.93.110 255.255.255.0
        set allowaccess ping https ssh http
        set type physical
        set snmp-index 4
    next
    edit "naf.root"
        set vdom "root"
        set type tunnel
        set src-check disable
        set snmp-index 5
    next
    edit "l2t.root"
        set vdom "root"
        set type tunnel
        set snmp-index 6
    next
    edit "ssl.root"
        set vdom "root"
        set type tunnel
        set alias "SSL VPN interface"
        set snmp-index 7
    next
    edit "fortilink"
        set vdom "root"
        set fortilink enable
        set ip 10.255.1.1 255.255.255.0
        set allowaccess ping fabric
        set type aggregate
        set lldp-reception enable
        set lldp-transmission enable
        set snmp-index 8
    next
    edit "VLAN1001"
        set vdom "root"
        set vrf 10
        set ip 10.0.0.1 255.255.255.248
        set allowaccess ping
        set alias "VLAN1001"
        set device-identification enable
        set role lan
        set snmp-index 9
        set interface "port2"
        set vlanid 1001
    next
    edit "VLAN1002"
        set vdom "root"
        set vrf 10
        set ip 10.16.0.1 255.255.255.248
        set allowaccess ping
        set alias "VLAN1002"
        set role dmz
        set snmp-index 10
        set interface "port2"
        set vlanid 1002
    next
end

DC1-iFW-01 (interface) #

C1-iFW-01 (policy) # show
config firewall policy
    edit 1
        set name "PROD-to-VMWARE"
        set uuid 4d891d0c-492f-51ef-a03d-3a5ee1cd9fe5
        set srcintf "VLAN1001"
        set dstintf "VLAN1002"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
    next
    edit 2
        set uuid 52459456-492f-51ef-2018-581b86a94adb
        set srcintf "VLAN1002"
        set dstintf "VLAN1001"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set comments " (Copy of PROD-to-VMWARE) (Reverse of PROD-to-VMWARE)"
    next
end

DC1-iFW-01 # config router bgp

DC1-iFW-01 (bgp) # show
config router bgp
    set as 65011
    set router-id 10.0.0.1
    config neighbor
        edit "10.0.0.3"
            set soft-reconfiguration enable
            set as-override enable
            set distribute-list-in "PROD_IN"
            set interface "VLAN1001"
            set remote-as 65001
        next
        edit "10.16.0.3"
            set soft-reconfiguration enable
            set as-override enable
            set distribute-list-in "VMWARE_IN"
            set interface "VLAN1002"
            set remote-as 65001
            set update-source "VLAN1002"
        next
    end
    config network6
        edit 1
            set prefix6 ::/128
        next
    end
    config redistribute "connected"
    end
    config redistribute "rip"
    end
    config redistribute "ospf"
    end
    config redistribute "static"
    end
    config redistribute "isis"
    end
    config redistribute6 "connected"
    end
    config redistribute6 "rip"
    end
    config redistribute6 "ospf"
    end
    config redistribute6 "static"
    end
    config redistribute6 "isis"
    end
end

DC1-iFW-01 (access-list) # show
config router access-list
    edit "PROD_IN"
        config rule
            edit 1
                set prefix 10.0.0.0 255.240.0.0
            next
            edit 2
                set action deny
                set prefix 10.16.0.0 255.240.0.0
            next
        end
    next
    edit "VMWARE_IN"
        config rule
            edit 1
                set prefix 10.16.0.0 255.240.0.0
            next
            edit 2
                set action deny
                set prefix 10.0.0.0 255.240.0.0
            next
        end
    next
end

```

</details>

- #### Data Center #2

<details>
  <summary>DC2-TORSW-01</summary>

```
DC2-TORSW-01#sh run
! Command: show running-config
! device: DC2-TORSW-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-TORSW-01
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 102
   name net-102-vmware
!
vlan 1001
   name OSPF-Prod
!
vlan 1002
   name OSPF-VMWARE
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0012:aaaa:aaaa:0010
      route-target import 02:12:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0012:aaaa:aaaa:0020
      route-target import 02:12:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.0/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.2/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   switchport trunk allowed vlan 1001-1002
   switchport mode trunk
   no ip ospf neighbor bfd
!
interface Ethernet5
   description ### Link to CH1-CSW-01 int Eth2 ###
   mtu 9164
   no switchport
   ip address 172.23.0.16/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 0qDaaPrQfPk=
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.22.0.1/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan102
   description ### VMWARE ###
   vrf VMWARE
   ip address virtual 10.16.102.1/24
!
interface Vlan1001
   vrf PROD
   ip address 10.0.0.11/29
!
interface Vlan1002
   vrf VMWARE
   ip address 10.16.0.11/29
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 102 vni 10102
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
ip route vrf VMWARE 10.16.103.0/24 vtep 172.19.0.1 vni 20001 router-mac-address 50:00:00:88:2f:f3 210
!
route-map AS-PATH permit 10
   set as-path prepend 65001
!
route-map LOC-PREF permit 10
   set local-preference 50
!
router bgp 65001
   router-id 172.22.0.1
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.21.0.1 peer group SPINES
   neighbor 172.21.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 102
      rd 65001:102
      route-target both 65001:10102
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      neighbor 10.0.0.9 remote-as 65012
      neighbor 10.0.0.9 update-source Vlan1001
      neighbor 10.0.0.9 route-map LOC-PREF in
      neighbor 10.0.0.9 route-map AS-PATH out
      aggregate-address 10.0.0.0/12 summary-only
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      neighbor 10.16.0.9 remote-as 65012
      neighbor 10.16.0.9 update-source Vlan1002
      neighbor 10.16.0.9 route-map LOC-PREF in
      neighbor 10.16.0.9 route-map AS-PATH out
      aggregate-address 10.16.0.0/12 summary-only
      redistribute connected
!
router ospf 1
   router-id 172.22.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC2-TORSW-02</summary>

```

DC2-TORSW-02#sh run
! Command: show running-config
! device: DC2-TORSW-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-TORSW-02
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 102
   name net-102-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0012:aaaa:aaaa:0010
      route-target import 02:12:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0012:aaaa:aaaa:0020
      route-target import 02:12:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.4/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.6/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   no ip ospf neighbor bfd
!
interface Ethernet5
   no ip ospf neighbor bfd
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.22.0.2/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan102
   description ### VMWARE ###
   vrf VMWARE
   ip address virtual 10.16.102.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 102 vni 10102
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.22.0.2
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.21.0.1 peer group SPINES
   neighbor 172.21.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 102
      rd 65001:102
      route-target both 65001:10102
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
!
router ospf 1
   router-id 172.22.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC2-TORSW-03</summary>

```

DC2-TORSW-03#sh run
! Command: show running-config
 ! device: DC2-TORSW-03 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-TORSW-03
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 102
   name net-102-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0034:aaaa:aaaa:0010
      route-target import 02:34:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0034:aaaa:aaaa:0020
      route-target import 02:34:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.8/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.10/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   no ip ospf neighbor bfd
!
interface Ethernet5
   no ip ospf neighbor bfd
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.22.0.3/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan102
   description ### VMWARE ###
   vrf VMWARE
   ip address virtual 10.16.102.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 102 vni 10102
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.22.0.3
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.21.0.1 peer group SPINES
   neighbor 172.21.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 102
      rd 65001:102
      route-target both 65001:10102
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
!
router ospf 1
   router-id 172.22.0.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC2-TORSW-04</summary>

```
DC2-TORSW-04#sh run
! Command: show running-config
! device: DC2-TORSW-04 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-TORSW-04
!
spanning-tree mode mstp
!
vlan 10
   name net-10-prod
!
vlan 11
   name net-11-prod
!
vlan 20
   name net-20-prod
!
vlan 102
   name net-102-vmware
!
vrf instance PROD
   description ### PROD SERVERS ###
!
vrf instance VMWARE
   description ### VMWARE SERVICES ###
!
interface Port-Channel10
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0034:aaaa:aaaa:0010
      route-target import 02:34:aa:aa:00:10
   lacp system-id 0010.aaaa.aaaa
!
interface Port-Channel20
   switchport trunk allowed vlan 10-11,20,102
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0002:0034:aaaa:aaaa:0020
      route-target import 02:34:aa:aa:00:20
   lacp system-id 0020.aaaa.aaaa
!
interface Ethernet1
   description ### Link to DC2-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.12/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.14/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   no ip ospf neighbor bfd
!
interface Ethernet4
   no ip ospf neighbor bfd
!
interface Ethernet5
   no ip ospf neighbor bfd
!
interface Ethernet6
   no ip ospf neighbor bfd
!
interface Ethernet7
   channel-group 10 mode active
   no ip ospf neighbor bfd
!
interface Ethernet8
   channel-group 20 mode active
   no ip ospf neighbor bfd
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.22.0.4/32
!
interface Management1
!
interface Vlan10
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan11
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.11.1/24
!
interface Vlan20
   description ### GW net-10 ###
   vrf PROD
   ip address virtual 10.0.20.1/24
!
interface Vlan102
   description ### VMWARE ###
   vrf VMWARE
   ip address virtual 10.16.102.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 11 vni 10011
   vxlan vlan 20 vni 10020
   vxlan vlan 102 vni 10102
   vxlan vrf PROD vni 20000
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf PROD
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.22.0.4
   timers bgp 10 30
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES update-source Loopback1
   neighbor SPINES send-community extended
   neighbor 172.21.0.1 peer group SPINES
   neighbor 172.21.0.2 peer group SPINES
   !
   vlan 10
      rd 65001:10
      route-target both 65001:10010
      redistribute learned
   !
   vlan 102
      rd 65001:102
      route-target both 65001:10102
      redistribute learned
   !
   vlan 11
      rd 65001:11
      route-target both 65001:10011
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
   vrf PROD
      rd 10.1.1.0:20000
      route-target import 65001:20000
      route-target export 65001:20000
      redistribute connected
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      redistribute connected
!
router ospf 1
   router-id 172.22.0.4
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC2-CSW-01</summary>

```
DC2-CSW-01#sh run
! Command: show running-config
! device: DC2-CSW-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-CSW-01
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC2-TORSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.1/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-TORSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.5/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   description ### Link to DC2-TORSW-03 ###
   mtu 9214
   no switchport
   ip address 172.23.0.9/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet4
   description ### Link to DC2-TORSW-04 ###
   mtu 9214
   no switchport
   ip address 172.23.0.13/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet5
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.1/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet6
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.3/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet7
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.13/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet8
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.15/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 EebseYaSiEVWe+SfUhSAzQ==
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.21.0.1/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 172.21.0.1
   timers bgp 10 30
   bgp cluster-id 0.0.0.2
   bgp listen range 172.22.0.0/16 peer-group LEAFS remote-as 65001
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback1
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback1
   neighbor RR send-community
   neighbor WITNESS peer group
   neighbor WITNESS remote-as 65001
   neighbor WITNESS update-source Loopback1
   neighbor WITNESS send-community extended
   neighbor 172.17.0.1 peer group RR
   neighbor 172.17.0.2 peer group RR
   neighbor 172.19.0.1 peer group WITNESS
   neighbor 172.21.0.2 peer group RR
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
      neighbor WITNESS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor WITNESS activate
!
router ospf 1
   router-id 172.21.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end

```

</details>

<details>
  <summary>DC2-CSW-02</summary>

```
DC2-CSW-02#sh run
! Command: show running-config
 ! device: DC2-CSW-02 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname DC2-CSW-02
!
spanning-tree mode mstp
!
interface Ethernet1
   description ### Link to DC2-TORSW-01 ###
   mtu 9214
   no switchport
   ip address 172.23.0.3/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 sWOHWEVSxhBbE4FH0cwrRg==
!
interface Ethernet2
   description ### Link to DC2-TORSW-02 ###
   mtu 9214
   no switchport
   ip address 172.23.0.7/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet3
   description ### Link to DC2-TORSW-03 ###
   mtu 9214
   no switchport
   ip address 172.23.0.11/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 cTjzXaMM0v0jtIw/jkVf4g==
!
interface Ethernet4
   description ### Link to DC2-TORSW-04 ###
   mtu 9214
   no switchport
   ip address 172.23.0.15/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet5
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.9/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet6
   description ### Link to DC1-CSW-02 ###
   mtu 9214
   no switchport
   ip address 172.26.0.11/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet7
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.5/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 H3pqkq44OhZvfiz0RSOn7g==
!
interface Ethernet8
   description ### Link to DC1-CSW-01 ###
   mtu 9214
   no switchport
   ip address 172.26.0.7/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 EebseYaSiEVWe+SfUhSAzQ==
!
interface Ethernet9
   no ip ospf neighbor bfd
!
interface Loopback1
   description ### For BGP/EVPN ###
   ip address 172.21.0.2/32
!
interface Management1
!
ip routing
!
router bgp 65001
   router-id 172.21.0.2
   timers bgp 10 30
   bgp cluster-id 0.0.0.2
   bgp listen range 172.22.0.0/16 peer-group LEAFS remote-as 65001
   neighbor LEAFS peer group
   neighbor LEAFS remote-as 65001
   neighbor LEAFS update-source Loopback1
   neighbor LEAFS route-reflector-client
   neighbor LEAFS send-community
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback1
   neighbor RR send-community
   neighbor WITNESS peer group
   neighbor WITNESS remote-as 65001
   neighbor WITNESS update-source Loopback1
   neighbor WITNESS send-community extended
   neighbor 172.17.0.1 peer group RR
   neighbor 172.17.0.2 peer group RR
   neighbor 172.19.0.1 peer group WITNESS
   neighbor 172.21.0.1 peer group RR
   !
   address-family evpn
      neighbor LEAFS activate
      neighbor RR activate
      neighbor WITNESS activate
   !
   address-family ipv4
      no neighbor LEAFS activate
      no neighbor WITNESS activate
!
router ospf 1
   router-id 172.21.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   no passive-interface Ethernet6
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
   maximum-paths 16
!
end
```

</details>

<details>
  <summary>DC2-FW-01</summary>

```
DC2-iFW-01 (VLAN1001) # show
config system interface
    edit "VLAN1001"
        set vdom "root"
        set vrf 10
        set ip 10.0.0.9 255.255.255.248
        set allowaccess ping
        set alias "VLAN1001"
        set device-identification enable
        set role lan
        set snmp-index 9
        set interface "port2"
        set vlanid 1001
    next
end

DC2-iFW-01 (VLAN1001) # next

DC2-iFW-01 (interface) # 
DC2-iFW-01 (interface) # 
DC2-iFW-01 (interface) # 
DC2-iFW-01 (interface) # 
DC2-iFW-01 (interface) # show
config system interface
    edit "port1"
        set vdom "root"
        set mode dhcp
        set allowaccess ping https ssh fgfm
        set type physical
        set snmp-index 1
    next
    edit "port2"
        set vdom "root"
        set vrf 10
        set type physical
        set alias "TRUNK"
        set snmp-index 2
    next
    edit "port3"
        set vdom "root"
        set type physical
        set snmp-index 3
    next
    edit "port4"
        set vdom "root"
        set ip 10.250.93.111 255.255.255.0
        set allowaccess ping https ssh http
        set type physical
        set snmp-index 4
    next
    edit "naf.root"
        set vdom "root"
        set type tunnel
        set src-check disable
        set snmp-index 5
    next
    edit "l2t.root"
        set vdom "root"
        set type tunnel
        set snmp-index 6
    next
    edit "ssl.root"
        set vdom "root"
        set type tunnel
        set alias "SSL VPN interface"
        set snmp-index 7
    next
    edit "fortilink"
        set vdom "root"
        set fortilink enable
        set ip 10.255.1.1 255.255.255.0
        set allowaccess ping fabric
        set type aggregate
        set lldp-reception enable
        set lldp-transmission enable
        set snmp-index 8
    next
    edit "VLAN1001"
        set vdom "root"
        set vrf 10
        set ip 10.0.0.9 255.255.255.248
        set allowaccess ping
        set alias "VLAN1001"
        set device-identification enable
        set role lan
        set snmp-index 9
        set interface "port2"
        set vlanid 1001
    next
    edit "VLAN1002"
        set vdom "root"
        set vrf 10
        set ip 10.16.0.9 255.255.255.248
        set allowaccess ping
        set alias "VLAN1002"
        set device-identification enable
        set role lan
        set snmp-index 10
        set interface "port2"
        set vlanid 1002
    next
end

DC2-iFW-01 (policy) # show
config firewall policy
    edit 1
        set name "PROD-to-VMWARE"
        set uuid 3fb019e6-53df-51ef-b992-2f05a68faa1a
        set srcintf "VLAN1001"
        set dstintf "VLAN1002"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
    next
    edit 2
        set uuid 43c57562-53df-51ef-ebfc-0173cb5a1e61
        set srcintf "VLAN1002"
        set dstintf "VLAN1001"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set comments " (Copy of PROD-to-VMWARE) (Reverse of PROD-to-VMWARE)"
    next
end

DC2-iFW-01 # config router bgp

DC2-iFW-01 (bgp) # show
config router bgp
    set as 65012
    set router-id 10.0.0.9
    config neighbor
        edit "10.0.0.11"
            set soft-reconfiguration enable
            set as-override enable
            set distribute-list-in "PROD_IN"
            set interface "VLAN1001"
            set remote-as 65001
            set update-source "VLAN1001"
        next
        edit "10.16.0.11"
            set soft-reconfiguration enable
            set as-override enable
            set distribute-list-in "VMWARE_IN"
            set interface "VLAN1002"
            set remote-as 65001
            set update-source "VLAN1002"
        next
    end
    config network6
        edit 1
            set prefix6 ::/128
        next
    end
    config redistribute "connected"
    end
    config redistribute "rip"
    end
    config redistribute "ospf"
    end
    config redistribute "static"
    end
    config redistribute "isis"
    end
    config redistribute6 "connected"
    end
    config redistribute6 "rip"
    end
    config redistribute6 "ospf"
    end  
    config redistribute6 "static"
    end
    config redistribute6 "isis"
    end
end

DC2-iFW-01 (access-list) # show
config router access-list
    edit "PROD_IN"
        config rule
            edit 1
                set prefix 10.0.0.0 255.240.0.0
            next
            edit 2
                set action deny
                set prefix 10.16.0.0 255.240.0.0
            next
        end
    next
    edit "VMWARE_IN"
        config rule
            edit 1
                set prefix 10.16.0.0 255.240.0.0
            next
            edit 2
                set action deny
                set prefix 10.0.0.0 255.240.0.0
            next
        end
    next
end

```

</details>

- #### Campus

<details>
  <summary>CH1-CSW-01</summary>
  
```
CH1-CSW-01(config)#sh run
! Command: show running-config
! device: CH1-CSW-01 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname CH1-CSW-01
!
spanning-tree mode mstp
!
vlan 103
   name VMWARE
!
vrf instance VMWARE
!
interface Ethernet1
   description ### Link to DC1-TORSW-01 int Eth5 ###
   mtu 9214
   no switchport
   ip address 172.18.0.17/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 VoOL9kzywbE=
!
interface Ethernet2
   description ### Link to DC2-TORSW-01 int Eth5 ###
   mtu 9214
   no switchport
   ip address 172.23.0.17/31
   bfd interval 1000 min-rx 1000 multiplier 25
   no ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 sha512 7 Vvfzqj00sns=
!
interface Ethernet3
   description ### Link to Witness ###
   mtu 9164
   switchport trunk allowed vlan 103
   switchport mode trunk
   no ip ospf neighbor bfd
!
interface Ethernet4
   mtu 9164
   no ip ospf neighbor bfd
!
interface Ethernet5
   mtu 9164
   no ip ospf neighbor bfd
   ip ospf message-digest-key 1 sha512 7 VvYj7xR9hxZ85W1CvCZOUw==
!
interface Ethernet6
   mtu 9164
   no ip ospf neighbor bfd
!
interface Ethernet7
   mtu 9164
   no ip ospf neighbor bfd
!
interface Ethernet8
   mtu 9164
   no ip ospf neighbor bfd
!
interface Ethernet9
   mtu 9164
   no ip ospf neighbor bfd
!
interface Loopback1
   ip address 172.19.0.1/32
!
interface Management1
!
interface Vlan103
   description ### VMWARE ###
   mtu 9164
   vrf VMWARE
   ip address 10.16.103.1/24
!
interface Vlan193
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vrf VMWARE vni 20001
!
ip virtual-router mac-address 00:00:00:65:00:10
!
ip routing
ip routing vrf VMWARE
!
router bgp 65001
   router-id 172.19.0.1
   timers bgp 10 30
   neighbor RR peer group
   neighbor RR remote-as 65001
   neighbor RR update-source Loopback1
   neighbor RR send-community
   neighbor 172.17.0.1 peer group RR
   neighbor 172.17.0.2 peer group RR
   neighbor 172.21.0.1 peer group RR
   neighbor 172.21.0.2 peer group RR
   !
   address-family evpn
      neighbor RR activate
   !
   address-family ipv4
      no neighbor RR activate
   !
   vrf VMWARE
      rd 10.1.1.0:20001
      route-target import 65001:20001
      route-target export 65001:20001
      redistribute connected
!
router ospf 1
   router-id 172.19.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 172.16.0.0/12 area 0.0.0.0
   max-lsa 12000
!
end

```
  
</details>



### Проверки:


<details>
  <summary>DC1-TORSW-01</summary>
  
  ```
DC1-TORSW-01(config)#show ip route vrf PROD

VRF: PROD
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

 C        10.0.0.0/29 is directly connected, Vlan1001
 B I      10.0.10.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.10.14/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.10.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.10.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.10.0/24 is directly connected, Vlan10
 B I      10.0.11.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.11.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.11.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.11.0/24 is directly connected, Vlan11
 B I      10.0.20.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.20.14/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.20.21/32 [200/0] via VTEP 172.22.0.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      10.0.20.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.20.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.20.0/24 is directly connected, Vlan20
 A B      10.0.0.0/12 is directly connected, Null0
 B E      10.16.0.0/12 [200/0] via 10.0.0.1, Vlan1001

```

```
DC1-TORSW-01(config)#show ip route vrf VMWARE

VRF: VMWARE
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

 B I      10.0.101.0/24 [200/0] via VTEP 172.16.0.4 VNI 20001 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      10.0.0.0/12 [200/0] via 10.16.0.1, Vlan1002
 C        10.16.0.0/29 is directly connected, Vlan1002
 C        10.16.101.0/24 is directly connected, Vlan101
 B I      10.16.102.22/32 [200/0] via VTEP 172.22.0.1 VNI 20001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      10.16.102.23/32 [200/0] via VTEP 172.22.0.3 VNI 20001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.16.102.24/32 [200/0] via VTEP 172.22.0.3 VNI 20001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.16.102.0/24 [200/0] via VTEP 172.22.0.4 VNI 20001 router-mac 50:00:00:1b:5e:8d local-interface Vxlan1
 B I      10.16.103.0/24 [200/0] via VTEP 172.19.0.1 VNI 20001 router-mac 50:00:00:88:2f:f3 local-interface Vxlan1
 A B      10.16.0.0/12 is directly connected, Null0

```

```
DC1-TORSW-01(config)#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 172.16.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >      RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >Ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 * >Ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 *  ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 * >Ec    RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.1
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.1
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.1
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.1
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2

```

```
DC1-TORSW-01(config)#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 172.16.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.0.0/12
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:20001 ip-prefix 10.0.0.0/12
                                 -                     -       100     0       65011 65011 i
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.10.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.10.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.11.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.11.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.20.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.20.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.0.101.0/24
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.0.101.0/24
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.16.0.0/12
                                 -                     -       100     0       65011 65011 i
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.0.0/12
                                 -                     -       -       0       i
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.101.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.101.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.102.0/24
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.102.0/24
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.1 0.0.0.2
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.103.0/24
                                 172.19.0.1            -       100     0       i Or-ID: 172.19.0.1 C-LST: 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.103.0/24
                                 172.19.0.1            -       100     0       i Or-ID: 172.19.0.1 C-LST: 0.0.0.1
```

```
DC1-TORSW-01(config)#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10 mac-ip aabb.cc80.d000
                                 -                     -       -       0       i
 * >      RD: 65001:11 mac-ip aabb.cc80.d000
                                 -                     -       -       0       i
 * >      RD: 65001:20 mac-ip aabb.cc80.d000
                                 -                     -       -       0       i
 * >      RD: 65001:101 mac-ip aabb.cc80.d000
                                 -                     -       -       0       i
 * >      RD: 65001:10 mac-ip aabb.cc80.d000 10.0.10.11
                                 -                     -       -       0       i
 * >      RD: 65001:20 mac-ip aabb.cc80.d000 10.0.20.11
                                 -                     -       -       0       i
 * >      RD: 65001:101 mac-ip aabb.cc80.d000 10.16.101.11
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.e000 10.0.10.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.e000 10.0.10.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc80.e000 10.0.11.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc80.e000 10.0.11.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.e000 10.0.20.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.e000 10.0.20.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.f000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc80.f000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc80.f000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc80.f000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.f000 10.0.20.21
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc80.f000 10.0.20.21
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.0000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.0000
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.0000 10.16.102.22
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.0000 10.16.102.22
                                 172.22.0.1            -       100     0       i Or-ID: 172.22.0.1 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.1000 10.0.10.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.1000 10.0.10.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.1000 10.0.11.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.1000 10.0.11.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.1000 10.0.20.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.1000 10.0.20.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.1000 10.16.102.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.1000 10.16.102.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >      RD: 65001:10 mac-ip aabb.cc81.2000
                                 -                     -       -       0       i
 * >      RD: 65001:11 mac-ip aabb.cc81.2000
                                 -                     -       -       0       i
 * >      RD: 65001:101 mac-ip aabb.cc81.2000
                                 -                     -       -       0       i
 * >      RD: 65001:10 mac-ip aabb.cc81.2000 10.0.10.12
                                 -                     -       -       0       i
 * >      RD: 65001:101 mac-ip aabb.cc81.2000 10.16.101.12
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.3000 10.0.10.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.3000 10.0.10.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.3000 10.0.20.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc81.3000 10.0.20.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.4000 10.0.10.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.4000 10.0.10.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.4000 10.0.11.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.4000 10.0.11.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.4000 10.0.20.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.4000 10.0.20.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.4000 10.16.102.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.4000 10.16.102.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.1 0.0.0.2
```

</details>

<details>
  <summary>DC2-TORSW-01</summary>
  
```
DC2-TORSW-01#show ip route vrf PROD

VRF: PROD
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

 C        10.0.0.8/29 is directly connected, Vlan1001
 B I      10.0.10.11/32 [200/0] via VTEP 172.16.0.1 VNI 20000 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      10.0.10.12/32 [200/0] via VTEP 172.16.0.1 VNI 20000 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      10.0.10.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.10.14/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.10.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.10.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.10.0/24 is directly connected, Vlan10
 B I      10.0.11.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.11.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.11.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.11.0/24 is directly connected, Vlan11
 B I      10.0.20.11/32 [200/0] via VTEP 172.16.0.1 VNI 20000 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      10.0.20.13/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.20.14/32 [200/0] via VTEP 172.16.0.3 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B I      10.0.20.23/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.0.20.24/32 [200/0] via VTEP 172.22.0.3 VNI 20000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.0.20.0/24 is directly connected, Vlan20
 A B      10.0.0.0/12 is directly connected, Null0
 B I      10.16.0.0/12 [200/0] via VTEP 172.16.0.1 VNI 20000 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

```

```
DC2-TORSW-01#show ip route vrf VMWARE

VRF: VMWARE
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

 B I      10.0.101.0/24 [200/0] via VTEP 172.16.0.4 VNI 20001 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B I      10.0.0.0/12 [200/0] via VTEP 172.16.0.1 VNI 20001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        10.16.0.8/29 is directly connected, Vlan1002
 B I      10.16.101.11/32 [200/0] via VTEP 172.16.0.1 VNI 20001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      10.16.101.12/32 [200/0] via VTEP 172.16.0.1 VNI 20001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      10.16.101.0/24 [200/0] via VTEP 172.16.0.2 VNI 20001 router-mac 50:00:00:cb:38:c2 local-interface Vxlan1
 B I      10.16.102.23/32 [200/0] via VTEP 172.22.0.3 VNI 20001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.16.102.24/32 [200/0] via VTEP 172.22.0.3 VNI 20001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        10.16.102.0/24 is directly connected, Vlan102
 B I      10.16.103.0/24 [200/0] via VTEP 172.19.0.1 VNI 20001 router-mac 50:00:00:88:2f:f3 local-interface Vxlan1
 A B      10.16.0.0/12 is directly connected, Null0

```

```
DC2-TORSW-01#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 172.22.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.1
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.1
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0010 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.1
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.1:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.1
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.2:1 ethernet-segment 0001:0012:aaaa:aaaa:0020 172.16.0.2
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0010 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.3:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.3
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 172.16.0.4:1 ethernet-segment 0001:0034:aaaa:aaaa:0020 172.16.0.4
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0010 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.2
 * >      RD: 172.22.0.1:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.2:1 ethernet-segment 0002:0012:aaaa:aaaa:0020 172.22.0.2
                                 172.22.0.2            -       100     0       i Or-ID: 172.22.0.2 C-LST: 0.0.0.2
 * >Ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0010 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
 * >Ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.3:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.3
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
 *  ec    RD: 172.22.0.4:1 ethernet-segment 0002:0034:aaaa:aaaa:0020 172.22.0.4
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
```

```
DC2-TORSW-01#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 172.22.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.0.0/12
                                 -                     -       -       0       i
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.0.0/12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.0.0/12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.0.0.0/12
                                 172.16.0.1            -       100     0       65011 65011 i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.0.0.0/12
                                 172.16.0.1            -       100     0       65011 65011 i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.0.0.0/12
                                 -                     -       50      0       65012 65012 65012 i
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.10.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.10.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.11.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.11.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.0.20.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.0.20.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.0.101.0/24
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.0.101.0/24
                                 172.16.0.4            -       100     0       i Or-ID: 172.16.0.4 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20000 ip-prefix 10.16.0.0/12
                                 172.16.0.1            -       100     0       65011 65011 i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.16.0.0/12
                                 172.16.0.1            -       100     0       65011 65011 i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20000 ip-prefix 10.16.0.0/12
                                 -                     -       50      0       65012 65012 65012 i
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.0.0/12
                                 -                     -       -       0       i
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.0.0/12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.0.0/12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.101.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.101.0/24
                                 172.16.0.2            -       100     0       i Or-ID: 172.16.0.2 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.102.0/24
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.102.0/24
                                 172.22.0.4            -       100     0       i Or-ID: 172.22.0.4 C-LST: 0.0.0.2
 * >      RD: 10.1.1.0:20001 ip-prefix 10.16.103.0/24
                                 172.19.0.1            -       100     0       i Or-ID: 172.19.0.1 C-LST: 0.0.0.2
 *        RD: 10.1.1.0:20001 ip-prefix 10.16.103.0/24
                                 172.19.0.1            -       100     0       i Or-ID: 172.19.0.1 C-LST: 0.0.0.2
```

```
DC2-TORSW-01#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.22.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:101 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:101 mac-ip aabb.cc80.d000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.d000 10.0.10.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.d000 10.0.10.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.d000 10.0.20.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.d000 10.0.20.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:101 mac-ip aabb.cc80.d000 10.16.101.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:101 mac-ip aabb.cc80.d000 10.16.101.11
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.e000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc80.e000 10.0.10.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc80.e000 10.0.10.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc80.e000 10.0.11.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc80.e000 10.0.11.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc80.e000 10.0.20.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc80.e000 10.0.20.13
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >      RD: 65001:20 mac-ip aabb.cc80.f000
                                 -                     -       -       0       i
 * >      RD: 65001:102 mac-ip aabb.cc80.f000
                                 -                     -       -       0       i
 * >      RD: 65001:20 mac-ip aabb.cc80.f000 10.0.20.21
                                 -                     -       -       0       i
 * >      RD: 65001:102 mac-ip aabb.cc81.0000
                                 -                     -       -       0       i
 * >      RD: 65001:102 mac-ip aabb.cc81.0000 10.16.102.22
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.1000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.1000 10.0.10.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.1000 10.0.10.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.1000 10.0.11.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.1000 10.0.11.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.1000 10.0.20.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.1000 10.0.20.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.1000 10.16.102.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.1000 10.16.102.23
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:101 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:101 mac-ip aabb.cc81.2000
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.2000 10.0.10.12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.2000 10.0.10.12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:101 mac-ip aabb.cc81.2000 10.16.101.12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:101 mac-ip aabb.cc81.2000 10.16.101.12
                                 172.16.0.1            -       100     0       i Or-ID: 172.16.0.1 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:11 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc81.3000
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.3000 10.0.10.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:10 mac-ip aabb.cc81.3000 10.0.10.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.3000 10.0.20.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 *  ec    RD: 65001:20 mac-ip aabb.cc81.3000 10.0.20.14
                                 172.16.0.3            -       100     0       i Or-ID: 172.16.0.3 C-LST: 0.0.0.2 0.0.0.1
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.4000
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:10 mac-ip aabb.cc81.4000 10.0.10.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:10 mac-ip aabb.cc81.4000 10.0.10.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:11 mac-ip aabb.cc81.4000 10.0.11.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:11 mac-ip aabb.cc81.4000 10.0.11.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:20 mac-ip aabb.cc81.4000 10.0.20.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:20 mac-ip aabb.cc81.4000 10.0.20.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 * >Ec    RD: 65001:102 mac-ip aabb.cc81.4000 10.16.102.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
 *  ec    RD: 65001:102 mac-ip aabb.cc81.4000 10.16.102.24
                                 172.22.0.3            -       100     0       i Or-ID: 172.22.0.3 C-LST: 0.0.0.2
```

</details>

<details>
  <summary>ESXi Hosts</summary>
  

ICMP к хосту DC1-ESXi-04 vrf PROD

```
DC2-ESXi-04#ping vrf VMWARE-102 10.0.10.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.10.14, timeout is 2 seconds:
!!!!!
```

ICMP к хосту DC1-ESXi-04 vrf PROD

```
DC1-ESXi-04#ping vrf PROD-10 10.0.20.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/24/58 ms
```

ICMP к хосту DC1-ESXi-01 vrf VMWARE

```
DC1-ESXi-04#ping vrf PROD-10 10.16.101.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.16.101.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 55/358/1399 ms
```

ICMP к хосту DC2-ESXi-04 vrf PROD

```
DC1-ESXi-04#ping vrf PROD-10 10.0.20.24
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.24, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 45/80/156 ms
```

ICMP к хосту DC1-ESXi-01 vrf VMWARE

```
DC1-ESXi-01#ping vrf PROD-10 10.16.101.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.16.101.11, timeout is 2 seconds:
!!!!!
```

ICMP к хосту DC2-ESXi-01 vrf VMWARE

```
DC1-ESXi-01#ping vrf PROD-10 10.16.102.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.16.102.21, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 63/142/268 ms

```
  
</details>

<details>
  <summary>Witness Host</summary>
     
  ```
Witness_Host#ping 10.0.10.13
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.10.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 108/209/351 ms

Witness_Host#ping 10.0.10.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.10.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 66/221/615 ms

Witness_Host#ping 10.16.101.12
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.16.101.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 80/555/1171 ms

Witness_Host#ping 10.16.102.23
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.16.102.23, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 78/129/184 ms

Witness_Host#ping 10.0.20.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 62/274/875 ms

Witness_Host#ping 10.0.20.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.20.21, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 73/128/200 ms

  ```
  
</details>

<details>
  <summary>Трассировка при отказе DC1-FW-01 </summary>


Трассировка в сторону vrf PROD - DC1-iFW-01 включен

```
DC2-ESXi-04#traceroute vrf VMWARE-102 10.0.10.14
Type escape sequence to abort.
Tracing the route to 10.0.10.14
VRF info: (vrf in name/id, vrf out name/id)
  1 10.16.102.1 58 msec 14 msec 15 msec
  2 10.16.0.3 1444 msec 577 msec 48 msec
  3 10.16.0.1 679 msec 1353 msec 67 msec
  4 10.0.0.3 315 msec 411 msec 60 msec
  5 10.0.20.1 227 msec 157 msec 500 msec
  6 10.0.10.14 118 msec 1045 msec 127 msec

```
Трассировка в сторону vrf PROD - DC1-iFW-01 выключен

```
DC2-ESXi-04#traceroute vrf VMWARE-102 10.0.10.14
Type escape sequence to abort.
Tracing the route to 10.0.10.14
VRF info: (vrf in name/id, vrf out name/id)
  1 10.16.102.1 20 msec 938 msec 159 msec
  2 10.16.102.1 2376 msec 33 msec 27 msec
  3 10.16.0.9 45 msec 41 msec 36 msec
  4 10.0.0.11 39 msec 93 msec 46 msec
  5 10.0.20.1 130 msec 114 msec 207 msec
  6 10.0.10.14 275 msec 329 msec *

```
</details>

<details>
  <summary>Spoiler warning</summary>

DC1-iFW-01  
  ```
  DC1-iFW-01 # get router info bgp network 




VRF 10 BGP table version is 136, local router ID is 10.0.0.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 10.0.0.0/12      10.0.0.3                 0             0        0 65001 i <-/1>
*> 10.16.0.0/12     10.16.0.3                0             0        0 65001 i <-/1>

Total number of prefixes 2


DC1-iFW-01 # get router info routing-table all 
Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

Routing table for VRF=0
S*      0.0.0.0/0 [10/0] via 10.250.93.1, port4
C       10.250.93.0/24 is directly connected, port4

Routing table for VRF=10
B       10.0.0.0/12 [20/0] via 10.0.0.3 (recursive is directly connected, VLAN1001), 07:19:43
C       10.0.0.0/29 is directly connected, VLAN1001
B       10.16.0.0/12 [20/0] via 10.16.0.3 (recursive is directly connected, VLAN1002), 07:29:55
C       10.16.0.0/29 is directly connected, VLAN1002
  ```

DC1-iFW-01 

```
DC2-iFW-01 # get router info bgp network


VRF 10 BGP table version is 254, local router ID is 10.0.0.9
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 10.0.0.0/12      10.0.0.11                0             0        0 65001 65001 i <-/1>
*> 10.16.0.0/12     10.16.0.11               0             0        0 65001 65001 i <-/1>

Total number of prefixes 2


DC2-iFW-01 # get router info routing-table all
Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

Routing table for VRF=0
S*      0.0.0.0/0 [10/0] via 10.250.93.1, port4
C       10.250.93.0/24 is directly connected, port4

Routing table for VRF=10
B       10.0.0.0/12 [20/0] via 10.0.0.11 (recursive is directly connected, VLAN1001), 07:33:41
C       10.0.0.8/29 is directly connected, VLAN1001
B       10.16.0.0/12 [20/0] via 10.16.0.11 (recursive is directly connected, VLAN1002), 07:33:35
C       10.16.0.8/29 is directly connected, VLAN1002
```
  
</details>

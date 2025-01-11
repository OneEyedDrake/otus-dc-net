### **VxLAN. L3VNI**



Настроить маршрутизацию в рамках Overlay между клиентами

Для настройки VxLAN. L3 VNI будет использован стенд из 4-ой лабораторной работы, с добавлением дополнительной подсети и клиента рамещенного в нем.

### **План работы:**
1. Настройки будут произвоиться только на leaf, т.к. настройка идёт в Overlay сети;
2. На leaf3 создадим 70-й vlan для дальнешего подключения к vxlan (VLAN-based VXLAN) и подключения в него нового клиента;

   2.1 Добавим порт в который подключены хосты в *vlan 70*, **switchport access vlan 70**

   2.2 Мапинг VLAN к VXLAN **vxlan vlan 70 vni 70**

   2.3 Внесем дополнительные настройки в BGP для передачи информации о vxlan:
    - Route Distinquishers **rd auto** будет сформирован автоматически;
    - Route-target **route-target both 65000:70** должен совпадать на всех leaf для этого **vxlan**
    - Включить изучение mac в overlay, для данного *vxlan* **mac redistribute learned**
4. На всех leaf:
   
    3.1 создадим VRF инстанс **vrf instance *tenant1***в котором будут расположены клиенты;
   
    3.2 включим маршрутизацию в **vrf *tenant1***, командой **ip routing vrf tenant1**
   
    3.3 создадим SVI для VLAN 100 и 70(только на leaf3 т.к. данный клиент есть только на данном коммутаторе) соотвественно
   
        3.3.1 добавим их в **vrf *tenant1***
   
        3.3.2 настроим им вирутальные адреса (anycast gateway)  командой **ip address virtual** 10.4.70.1/24 и 10.4.100.1/24 (на каждом leaf адрес будет одинаковый)

    3.4 настроим глобально макадрес машрутизатора командой **ip virtual-router mac-address 00:00:00:00:00:01** (на всех leaf будет одинаковый)
   
    3.5 сделаем vxlan vni необходимый для работы симметирчной маршрутизации в vxlan **vxlan vrf tenant1 vni 10000**
       
    3.6 Настроим BGP для передачи машрутной информации в **vrf *tenant1***:
   
        - Route Distinquishers **rd 10.0.0.3:10000** будет уникальный для каждого leaf;
   
        - Route-target на импорт и экспорт маршрутов для L3 VNI **route-target import evpn 65000:10000** **route-target export evpn 65000:10000** должен совпадать на всех leaf для этого **vxlan**
   
        - Включить **redistribute connected** для передачи маршрутной информации о подключенных подсетях;
   
**Готово!** можно проверять доступность хостов host4 и host1

**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/ptp%20network.png)

**Таблица 3 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/host%20add.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/as%2065000.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/eve-ng-scheme.png)

## Конфигурация dc1-leaf1

```
vlan 100
   name vxlan100-cli
!
vrf instance tenant1
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.1/31
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.1/31
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
   switchport access vlan 100
!
interface Loopback1
   ip address 10.0.0.1/32
!
interface Loopback2
   ip address 10.1.0.1/32
!
interface Management1
!
interface Vlan1
!
interface Vlan100
   vrf tenant1
   ip address virtual 10.4.100.1/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 100
   vxlan vrf tenant1 vni 10000
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf tenant1
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.0.1
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay timers 3 9
   neighbor overlay password 7 wnmhtPzjs2lTHT7Eds39N3BzyPA2LfZA
   neighbor overlay send-community extended
   neighbor 10.1.1.0 peer group overlay
   neighbor 10.1.2.0 peer group overlay
   neighbor 10.2.1.0 peer group dc1
   neighbor 10.2.2.0 peer group dc1
   redistribute connected route-map redistrib_connect
   !
   vlan 100
      rd auto
      route-target both 65000:100
      redistribute learned
   !
   address-family evpn
      neighbor overlay activate
   !
   vrf tenant1
      rd 10.0.0.1:10000
      route-target import evpn 65000:10000
      route-target export evpn 65000:10000
!
```
## Конфигурация dc1-leaf2
```
vlan 100
   name vxlan100-cli
!
vrf instance tenant1
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.3/31
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.3/31
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
   switchport access vlan 100
!
interface Loopback1
   ip address 10.0.0.2/32
!
interface Loopback2
   ip address 10.1.0.2/32
!
interface Management1
!
interface Vlan100
   vrf tenant1
   ip address virtual 10.4.100.1/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 100
   vxlan vrf tenant1 vni 10000
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf tenant1
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.0.2
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay timers 3 9
   neighbor overlay password 7 wnmhtPzjs2lTHT7Eds39N3BzyPA2LfZA
   neighbor overlay send-community extended
   neighbor 10.1.1.0 peer group overlay
   neighbor 10.1.2.0 peer group overlay
   neighbor 10.2.1.2 peer group dc1
   neighbor 10.2.2.2 peer group dc1
   redistribute connected route-map redistrib_connect
   !
   vlan 100
      rd auto
      route-target both 65000:100
      redistribute learned
   !
   address-family evpn
      neighbor overlay activate
   !
   vrf tenant1
      rd 10.0.0.2:10000
      route-target import evpn 65000:10000
      route-target export evpn 65000:10000
!
```

## Конфигурация dc1-leaf3
```
hostname dc1-leaf3
!
spanning-tree mode mstp
!
vlan 70
   name tenant1-test2
!
vlan 100
   name vxlan-client
!
vrf instance tenant1
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.5/31
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.5/31
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
   switchport access vlan 100
!
interface Ethernet8
   switchport access vlan 70
!
interface Loopback1
   ip address 10.0.0.3/32
!
interface Loopback2
   ip address 10.1.0.3/32
!
interface Management1
!
interface Vlan70
   vrf tenant1
   ip address virtual 10.4.70.1/24
!
interface Vlan100
   vrf tenant1
   ip address virtual 10.4.100.1/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 70 vni 70
   vxlan vlan 100 vni 100
   vxlan vrf tenant1 vni 10000
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf tenant1
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.0.3
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay timers 3 9
   neighbor overlay password 7 wnmhtPzjs2lTHT7Eds39N3BzyPA2LfZA
   neighbor overlay send-community extended
   neighbor 10.1.1.0 peer group overlay
   neighbor 10.1.2.0 peer group overlay
   neighbor 10.2.1.4 peer group dc1
   neighbor 10.2.2.4 peer group dc1
   redistribute connected route-map redistrib_connect
   !
   vlan 100
      rd auto
      route-target both 65000:100
      redistribute learned
   !
   vlan 70
      rd auto
      route-target both 65000:70
      redistribute learned
   !
   address-family evpn
      neighbor overlay activate
   !
   vrf tenant1
      rd 10.0.0.3:10000
      route-target import evpn 65000:10000
      route-target export evpn 65000:10000
!
end
```
## Вывод команд show bgp evpn route-type mac-ip, show ip route vrf tenant1 dc1-leaf1
```
dc1-leaf1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:100 mac-ip 0050.7966.6802
                                 -                     -       -       0       i
 * >      RD: 10.0.0.1:100 mac-ip 0050.7966.6802 10.4.100.11
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808 10.4.100.12
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808 10.4.100.12
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 * >Ec    RD: 10.0.0.3:70 mac-ip 0050.7966.680a
                                 10.1.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.3:70 mac-ip 0050.7966.680a
                                 10.1.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.0
 * >Ec    RD: 10.0.0.3:70 mac-ip 0050.7966.680a 10.4.70.13
                                 10.1.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.3:70 mac-ip 0050.7966.680a 10.4.70.13
                                 10.1.0.3              -       100     0       i Or-ID: 10.0.0.3 C-LST: 10.0.1.0
dc1-leaf1#
dc1-leaf1#show ip route vrf tenant1

VRF: tenant1
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

 B I      10.4.70.13/32 [200/0] via VTEP 10.1.0.3 VNI 10000 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      10.4.100.12/32 [200/0] via VTEP 10.1.0.2 VNI 10000 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 C        10.4.100.0/24 is directly connected, Vlan100

```
## Вывод команд show bgp evpn route-type mac-ip, show ip route vrf tenant1 dc1-leaf3
```
dc1-leaf3#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.0.1:100 mac-ip 0050.7966.6802
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.1:100 mac-ip 0050.7966.6802
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >Ec    RD: 10.0.0.1:100 mac-ip 0050.7966.6802 10.4.100.11
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.1:100 mac-ip 0050.7966.6802 10.4.100.11
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808 10.4.100.12
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808 10.4.100.12
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 * >      RD: 10.0.0.3:70 mac-ip 0050.7966.680a
                                 -                     -       -       0       i
 * >      RD: 10.0.0.3:70 mac-ip 0050.7966.680a 10.4.70.13
                                 -                     -       -       0       i
dc1-leaf3#show ip route vrf tenant1

VRF: tenant1
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

 C        10.4.70.0/24 is directly connected, Vlan70
 B I      10.4.100.11/32 [200/0] via VTEP 10.1.0.1 VNI 10000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      10.4.100.12/32 [200/0] via VTEP 10.1.0.2 VNI 10000 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 C        10.4.100.0/24 is directly connected, Vlan100

```
### **Проверка доcтупности узлов**
---
## host1 to host4
```
VPCS> ping 10.4.70.13

84 bytes from 10.4.70.13 icmp_seq=1 ttl=63 time=661.463 ms
84 bytes from 10.4.70.13 icmp_seq=2 ttl=62 time=65.768 ms
84 bytes from 10.4.70.13 icmp_seq=3 ttl=62 time=38.719 ms
84 bytes from 10.4.70.13 icmp_seq=4 ttl=62 time=38.263 ms
84 bytes from 10.4.70.13 icmp_seq=5 ttl=62 time=42.072 ms

```
## host2 to host4
```
VPCS> ping 10.4.70.13

84 bytes from 10.4.70.13 icmp_seq=1 ttl=62 time=38.103 ms
84 bytes from 10.4.70.13 icmp_seq=2 ttl=62 time=37.017 ms
84 bytes from 10.4.70.13 icmp_seq=3 ttl=62 time=43.536 ms
84 bytes from 10.4.70.13 icmp_seq=4 ttl=62 time=48.725 ms

```
## host4 to host1

```
VPCS> ping 10.4.100.11

84 bytes from 10.4.100.11 icmp_seq=1 ttl=62 time=46.161 ms
84 bytes from 10.4.100.11 icmp_seq=2 ttl=62 time=57.466 ms
84 bytes from 10.4.100.11 icmp_seq=3 ttl=62 time=39.475 ms
84 bytes from 10.4.100.11 icmp_seq=4 ttl=62 time=63.441 ms

```

      

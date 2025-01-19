### **VxLAN. Routing**

Цель:
Реализовать маршрутизацию между "клиентами" через EVPN route-type 5

Для настройки маршрутизацию между "клиентами" через EVPN route-type 5 будет использован стенд из 6-ой лабораторной работы, добавлением новых клиентов в новом VRF и подключением пограничего маршрутизатора через который будет осуществляться маршрутизация между VRF подключенного в leaf 2. 
В схеме с подключением к leaf1 и leaf2 с использованием **lacp** и ESI в виртуализации были проблемы с высокими задержками с клиентов.

### **План работы:**
Настройка на leaf1-2:
*Настройка L2 VNI:*
1. Создадим *vlan 10,20*, для дальнешего подключения к vxlan (VLAN-based VXLAN);
2. Добавим порты в который подключены хосты в *vlan 10,20*, **switchport access vlan 10** **switchport access vlan 20**
11. Сделаем мапинг VLAN к VXLAN **vxlan vlan 10 vni 10** **vxlan vlan 20 vni 20** в настрйоках **interface Vxlan1**
12. Внесем дополнительные настройки в BGP для передачи информации о vxlan:
    - Route Distinquishers **rd auto** будет сформирован автоматически;
    - Route-target **route-target both 65000:10**(для vlan10) **route-target both 65000:20**(для vlan 20) должен совпадать на всех leaf для этого **vxlan**
    - Включить изучение mac в overlay, для данного *vxlan* **redistribute learned**
*Настройка L3 VNI:*




**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab07/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab07/ptp%20network.png)

**Таблица 3 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab07/host%20add.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab07/as%2065000-65535.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab07/eve-ng-scheme.png)

## Конфигурация dc1-leaf1

```
hostname dc1-leaf1
!
spanning-tree mode mstp
!
vlan 100
   name vxlan100-cli
!
vrf instance tenant1
!
interface Port-Channel1
   switchport trunk allowed vlan 100
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:2222:3333:4444:0000
      route-target import 55:55:55:55:55:55
   lacp system-id 7777.7777.7777
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
   channel-group 1 mode active
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
end
```
## Конфигурация dc1-leaf2
```
hostname dc1-leaf2
!
spanning-tree mode mstp
!
vlan 100
   name vxlan100-cli
!
vrf instance tenant1
!
interface Port-Channel1
   switchport trunk allowed vlan 100
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:2222:3333:4444:0000
      route-target import 55:55:55:55:55:55
   lacp system-id 7777.7777.7777
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.3/31
!
interface Ethernet1/6
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
   channel-group 1 mode active
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
end
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
## Вывод команд show bgp evpn route-type auto-discovery esi, show bgp evpn route-type ethernet-segment esi, show bgp evpn instance vlan, dc1-leaf1
```
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.2:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.2:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 * >      RD: 10.1.0.1:1 auto-discovery 0000:2222:3333:4444:0000
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.2:1 auto-discovery 0000:2222:3333:4444:0000
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec    RD: 10.1.0.2:1 auto-discovery 0000:2222:3333:4444:0000
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0

dc1-leaf1#show bgp evpn route-type ethernet-segment esi 0000:2222:3333:4444:0000
BGP routing table information for VRF default
Router identifier 10.0.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.2:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.2
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec    RD: 10.1.0.2:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.2
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
dc1-leaf1#show bgp evpn instance vlan 100
EVPN instance: VLAN 100
  Route distinguisher: 0:0
  Route target import: Route-Target-AS:65000:100
  Route target export: Route-Target-AS:65000:100
  Service interface: VLAN-based
  Local VXLAN IP address: 10.1.0.1
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 0000:2222:3333:4444:0000
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: 55:55:55:55:55:55
      DF election algorithm: modulus
      Designated forwarder: 10.1.0.1
      Non-Designated forwarder: 10.1.0.2

```

## Вывод команд show bgp evpn route-type auto-discovery esi, show bgp evpn route-type ethernet-segment esi, show bgp evpn instance vlan, dc1-leaf2
```
BGP routing table information for VRF default
Router identifier 10.0.0.2, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.0.1:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.1:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >      RD: 10.0.0.2:100 auto-discovery 0 0000:2222:3333:4444:0000
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.1:1 auto-discovery 0000:2222:3333:4444:0000
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.1.0.1:1 auto-discovery 0000:2222:3333:4444:0000
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >      RD: 10.1.0.2:1 auto-discovery 0000:2222:3333:4444:0000
                                 -                     -       -       0       i


BGP routing table information for VRF default
Router identifier 10.0.0.2, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >      RD: 10.1.0.2:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.2
                                 -                     -       -       0       i

dc1-leaf2#show bgp evpn instance vlan 100
EVPN instance: VLAN 100
  Route distinguisher: 0:0
  Route target import: Route-Target-AS:65000:100
  Route target export: Route-Target-AS:65000:100
  Service interface: VLAN-based
  Local VXLAN IP address: 10.1.0.2
  VXLAN: enabled
  MPLS: disabled
  Local ethernet segment:
    ESI: 0000:2222:3333:4444:0000
      Interface: Port-Channel1
      Mode: all-active
      State: up
      ES-Import RT: 55:55:55:55:55:55
      DF election algorithm: modulus
      Designated forwarder: 10.1.0.1
      Non-Designated forwarder: 10.1.0.2
```

## Вывод команды show ip route vrf tenant1, dc1-leaf3
```
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
 B I      10.4.100.21/32 [200/0] via VTEP 10.1.0.1 VNI 10000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                 via VTEP 10.1.0.2 VNI 10000 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 C        10.4.100.0/24 is directly connected, Vlan100
```
### **Проверка доcтупности узлов**
---
## node1 to host4
```
node1#ping 10.4.70.13
PING 10.4.70.13 (10.4.70.13) 72(100) bytes of data.
80 bytes from 10.4.70.13: icmp_seq=1 ttl=62 time=120 ms
80 bytes from 10.4.70.13: icmp_seq=2 ttl=62 time=192 ms
80 bytes from 10.4.70.13: icmp_seq=3 ttl=62 time=198 ms
80 bytes from 10.4.70.13: icmp_seq=4 ttl=62 time=193 ms
80 bytes from 10.4.70.13: icmp_seq=5 ttl=62 time=188 ms


```
## node1 to host3
```
node1#ping 10.4.100.13
PING 10.4.100.13 (10.4.100.13) 72(100) bytes of data.
80 bytes from 10.4.100.13: icmp_seq=1 ttl=64 time=259 ms
80 bytes from 10.4.100.13: icmp_seq=2 ttl=64 time=244 ms
80 bytes from 10.4.100.13: icmp_seq=3 ttl=64 time=243 ms
80 bytes from 10.4.100.13: icmp_seq=4 ttl=64 time=275 ms
80 bytes from 10.4.100.13: icmp_seq=5 ttl=64 time=294 ms


```
## host4 to node1

```
VPCS> ping 10.4.100.21

84 bytes from 10.4.100.21 icmp_seq=1 ttl=62 time=102.622 ms
84 bytes from 10.4.100.21 icmp_seq=2 ttl=62 time=64.213 ms
84 bytes from 10.4.100.21 icmp_seq=3 ttl=62 time=82.767 ms
84 bytes from 10.4.100.21 icmp_seq=4 ttl=62 time=75.940 ms
84 bytes from 10.4.100.21 icmp_seq=5 ttl=62 time=62.414 ms

```

## Вывод команды show ip route vrf tenant1, dc1-leaf3 в случае отключения интерфейса eth1 на устройсве node1
*Из таблицы маршрутизации видно, что пропал маршрут до node1 через leaf1*

```
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
 B I      10.4.100.21/32 [200/0] via VTEP 10.1.0.2 VNI 10000 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 C        10.4.100.0/24 is directly connected, Vlan100
```


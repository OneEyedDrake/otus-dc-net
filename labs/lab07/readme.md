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
3. Сделаем мапинг VLAN к VXLAN **vxlan vlan 10 vni 10** **vxlan vlan 20 vni 20** в настрйоках **interface Vxlan1**
4. Внесем дополнительные настройки в BGP для передачи информации о vxlan:
    - Route Distinquishers **rd auto** будет сформирован автоматически;
    - Route-target **route-target both 65000:10**(для vlan10) **route-target both 65000:20**(для vlan 20) должен совпадать на всех leaf для этого **vxlan**
    - Включить изучение mac в overlay, для данного *vxlan* **redistribute learned**
*Настройка L3 VNI и настройка соседства с AS65535:*
 1. создадим VRF инстанс **vrf instance *tenant2***в котором будут расположены клиенты;
   
 2. включим маршрутизацию в **vrf *tenant2***, командой **ip routing vrf tenant2**
   
 3. создадим SVI для VLAN 10 и 20 соотвественно
   
 4. добавим их в **vrf *tenant2***
   
 5. настроим им вирутальные адреса (anycast gateway)  командой **ip address virtual** 10.4.10.1/24 и 10.4.20.1/24 (на каждом leaf адрес будет одинаковый)

 6.сделаем vxlan vni необходимый для работы симметирчной маршрутизации в vxlan **vxlan vrf tenant2 vni 10001**
       
 6.1 Настроим BGP для передачи машрутной информации в **vrf *tenant2***:
   
        - Route Distinquishers **rd 10.0.0.1:10001** будет уникальный для каждого leaf;
   
        - Route-target на импорт и экспорт маршрутов для L3 VNI **route-target import evpn 65000:10001** **route-target export evpn 65000:10001** должен совпадать на всех leaf для этого **vxlan**
   
        - Включить **redistribute connected** для передачи маршрутной информации о подключенных подсетях;
        
        - на **leaf2** добавим neighbor 172.16.1.3 peer group border(для vrf tenant1) neighbor 172.16.2.3 peer group border(для vrf tenant2) для построения соседства с маршрутизатором **border**

        - настройки для bgp(ликинг между vrf): 
            -   neighbor border peer group
            -   neighbor border remote-as 65535
            -    neighbor border timers 3 9
            
6.2 Только на **leaf 2** cоздадим два vlan 1100 и 1200 и SVI с аналогчичными номерами для построения соседства через *border*
6.3. На SVI добавим адреса и поместим каждый в свой VRF. *SVI 1100 172.16.1.2/29 vrf tenant1*  и *SVI 1200 172.16.2.2/29 vrf tenant2* 
6.3 Интерфейс eth5 переведем в режим trunk и разрешим только хождение vlan с тегами 1100 и 1200;

Настройка border:
1. Cоздадим два vlan 1100 и 1200 и SVI с аналогчичными номерами для построения соседства через *border*
2. На SVI добавим адреса *SVI 1100 172.16.1.3/29* и *SVI 1200 172.16.2.3/29*
3. Интерфейс eth2 переведем в режим trunk и разрешим только хождение vlan с тегами 1100 и 1200;
4. Произведем настройку BGP:
1. Заранее создадим route-map для перезаписи AS, т.к. в BGP маршруты переданыне не могут быть получены в одной и той же AS. А VRF tenant1 и tenant2 находся в AS65000. 
2. Включить процесс **BGP**
3. Выполнить команду router-id *адрес loopback1*;
4. Cделаем настройки peer group аналогичные **leaf2**
```
neighbor tenant1,tenant2 peer group
   neighbor tenant1 peer group
   neighbor tenant1 remote-as 65000
   neighbor tenant1 timers 3 9
   neighbor tenant2 peer group
   neighbor tenant2 remote-as 65000
   neighbor tenant2 timers 3 9
```
5. В явном виде укажем в конфигурации соседей **leaf2**
```
   neighbor 172.16.1.2 peer group tenant1
   neighbor 172.16.2.2 peer group tenant2
```
6. Перезапишим номер AS на локальный:
```
    neighbor tenant1 route-map as-override out
    neighbor tenant2 route-map as-override out
```
Все готово,можно проверять доступноть между хостами из разных **VRF**. 





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
vlan 10
   name tenant2-prod
!
vlan 20
   name tenant2-test
!
vlan 100
   name vxlan100-cli
!
vrf instance tenant1
!
vrf instance tenant2
!
interface Port-Channel1
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
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 20
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
interface Vlan10
   vrf tenant2
   ip address virtual 10.4.10.1/24
!
interface Vlan20
   vrf tenant2
   ip address virtual 10.4.20.1/24
!
interface Vlan100
   vrf tenant1
   ip address virtual 10.4.100.1/24
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 10 vni 10
   vxlan vlan 20 vni 20
   vxlan vlan 100 vni 100
   vxlan vrf tenant1 vni 10000
   vxlan vrf tenant2 vni 10001
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf tenant1
ip routing vrf tenant2
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.0.1
   maximum-paths 2
   neighbor border peer group
   neighbor border remote-as 65535
   neighbor border timers 3 9
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
   vlan 10
      rd auto
      route-target both 65000:10
      redistribute learned
   !
   vlan 100
      rd auto
      route-target both 65000:100
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:20
      redistribute learned
   !
   address-family evpn
      neighbor overlay activate
   !
   vrf tenant1
      rd 10.0.0.1:10000
      route-target import evpn 65000:10000
      route-target export evpn 65000:10000
      neighbor 172.16.1.3 peer group border
      redistribute connected
   !
   vrf tenant2
      rd 10.0.0.1:10001
      route-target import evpn 65000:10001
      route-target export evpn 65000:10001
      neighbor 172.16.2.3 peer group border
      redistribute connected
!

```
## Конфигурация dc1-leaf2
```
hostname dc1-leaf2
!
spanning-tree mode mstp
!
vlan 10
   name tenant2-prod
!
vlan 20
   name tenant2-test
!
vlan 100
   name vxlan100-cli
!
vlan 1100
   name tenant1_to_tenant2
!
vlan 1200
   name tenant2_to_tenant1
!
vrf instance tenant1
!
vrf instance tenant2
!
interface Port-Channel1
   lacp system-id 7777.7777.7777
!
interface Port-Channel2
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
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 20
!
interface Ethernet5
   switchport trunk allowed vlan 1100,1200
   switchport mode trunk
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
interface Vlan10
   vrf tenant2
   ip address virtual 10.4.10.1/24
!
interface Vlan20
   vrf tenant2
   ip address virtual 10.4.20.1/24
!
interface Vlan100
   vrf tenant1
   ip address virtual 10.4.100.1/24
!
interface Vlan1100
   vrf tenant1
   ip address 172.16.1.2/29
!
interface Vlan1200
   vrf tenant2
   ip address 172.16.2.2/29
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 10 vni 10
   vxlan vlan 20 vni 20
   vxlan vlan 100 vni 100
   vxlan vrf tenant1 vni 10000
   vxlan vrf tenant2 vni 10001
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf tenant1
ip routing vrf tenant2
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.0.2
   maximum-paths 2
   neighbor border peer group
   neighbor border remote-as 65535
   neighbor border timers 3 9
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
   vlan 10
      rd auto
      route-target both 65000:10
      redistribute learned
   !
   vlan 100
      rd auto
      route-target both 65000:100
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:20
      redistribute learned
   !
   address-family evpn
      neighbor overlay activate
   !
   vrf tenant1
      rd 10.0.0.2:10000
      route-target import evpn 65000:10000
      route-target export evpn 65000:10000
      neighbor 172.16.1.3 peer group border
      redistribute connected
   !
   vrf tenant2
      rd 10.0.0.2:10001
      route-target import evpn 65000:10001
      route-target export evpn 65000:10001
      neighbor 172.16.2.3 peer group border
      redistribute connected
!
end
```

## Конфигурация Border
```
hostname border
!
spanning-tree mode mstp
!
vlan 1100
   name tenant1-to-tenant2
!
vlan 1200
   name tenant2-to-tenant1
!
interface Ethernet1
   shutdown
!
interface Ethernet2
   switchport trunk allowed vlan 1100,1200
   switchport mode trunk
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
   ip address 172.16.255.1/32
!
interface Management1
!
interface Vlan1
!
!
interface Vlan1100
   ip address 172.16.1.3/29
!
interface Vlan1200
   ip address 172.16.2.3/29
!
ip routing
!
route-map as-override permit 10
   set as-path match all replacement auto
!
router bgp 65535
   router-id 172.16.255.1
   neighbor tenant1 peer group
   neighbor tenant1 remote-as 65000
   neighbor tenant1 timers 3 9
   neighbor tenant1 route-map as-override out
   neighbor tenant2 peer group
   neighbor tenant2 remote-as 65000
   neighbor tenant2 timers 3 9
   neighbor tenant2 route-map as-override out
   neighbor 172.16.1.1 peer group tenant1
   neighbor 172.16.1.2 peer group tenant1
   neighbor 172.16.2.1 peer group tenant2
   neighbor 172.16.2.2 peer group tenant2
!
end

```
## Вывод команд show bgp evpn route-type ip-prefix, show ip route vrf tenant1,2, dc1-leaf2
```
          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.0.1:10001 ip-prefix 10.4.10.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.1:10001 ip-prefix 10.4.10.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 * >      RD: 10.0.0.2:10000 ip-prefix 10.4.10.0/24
                                 -                     -       100     0       65535 65535 i
 * >      RD: 10.0.0.2:10001 ip-prefix 10.4.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.0.0.2:10000 ip-prefix 10.4.10.11/32
                                 -                     -       100     0       65535 65535 i
 * >Ec    RD: 10.0.0.1:10001 ip-prefix 10.4.20.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.1:10001 ip-prefix 10.4.20.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 * >      RD: 10.0.0.2:10000 ip-prefix 10.4.20.0/24
                                 -                     -       100     0       65535 65535 i
 * >      RD: 10.0.0.2:10001 ip-prefix 10.4.20.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.1:10000 ip-prefix 10.4.100.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.1:10000 ip-prefix 10.4.100.0/24
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 * >      RD: 10.0.0.2:10000 ip-prefix 10.4.100.0/24
                                 -                     -       -       0       i
 * >      RD: 10.0.0.2:10001 ip-prefix 10.4.100.0/24
                                 -                     -       100     0       65535 65535 i
 * >      RD: 10.0.0.2:10001 ip-prefix 10.4.100.11/32
                                 -                     -       100     0       65535 65535 i
 * >      RD: 10.0.0.2:10000 ip-prefix 172.16.1.0/29
                                 -                     -       -       0       i
 * >      RD: 10.0.0.2:10001 ip-prefix 172.16.2.0/29



dc1-leaf2#show ip route vr tenant1

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

 B E      10.4.10.0/24 [200/0] via 172.16.1.3, Vlan1100
 B E      10.4.20.0/24 [200/0] via 172.16.1.3, Vlan1100
 B I      10.4.100.11/32 [200/0] via VTEP 10.1.0.1 VNI 10000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        10.4.100.0/24 is directly connected, Vlan100
 C        172.16.1.0/29 is directly connected, Vlan1100

dc1-leaf2#
dc1-leaf2#show ip route vr tenant2

VRF: tenant2
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

 C        10.4.10.0/24 is directly connected, Vlan10
 C        10.4.20.0/24 is directly connected, Vlan20
 B E      10.4.100.11/32 [200/0] via 172.16.2.3, Vlan1200
 B E      10.4.100.0/24 [200/0] via 172.16.2.3, Vlan1200
 C        172.16.2.0/29 is directly connected, Vlan1200
```

## Вывод команд show bgp evpn route-type ip-prefix, show ip route bgp, border
```
border#show ip route bgp

VRF: default
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

 B E      10.4.10.0/24 [200/0] via 172.16.2.2, Vlan1200
 B E      10.4.20.0/24 [200/0] via 172.16.2.2, Vlan1200
 B E      10.4.100.0/24 [200/0] via 172.16.1.2, Vlan1100

```

### **Проверка доcтупности узлов**
---
## tenant2-cli-1 to host1
```
VPCS> ping 10.4.100.11

84 bytes from 10.4.100.11 icmp_seq=1 ttl=59 time=99.488 ms
84 bytes from 10.4.100.11 icmp_seq=2 ttl=59 time=106.362 ms
84 bytes from 10.4.100.11 icmp_seq=3 ttl=59 time=122.457 ms
84 bytes from 10.4.100.11 icmp_seq=4 ttl=59 time=104.171 ms
84 bytes from 10.4.100.11 icmp_seq=5 ttl=59 time=98.795 ms
^C



```
## tenant2-cli-1 to host2
```
VPCS> ping 10.4.100.12

84 bytes from 10.4.100.12 icmp_seq=1 ttl=60 time=83.958 ms
84 bytes from 10.4.100.12 icmp_seq=2 ttl=60 time=78.673 ms
84 bytes from 10.4.100.12 icmp_seq=3 ttl=60 time=99.022 ms
84 bytes from 10.4.100.12 icmp_seq=4 ttl=60 time=71.539 ms
84 bytes from 10.4.100.12 icmp_seq=5 ttl=60 time=77.386 ms



```
## host2 to tenant2-cli-2

```
VPCS> ping 10.4.20.11

84 bytes from 10.4.20.11 icmp_seq=1 ttl=60 time=613.464 ms
84 bytes from 10.4.20.11 icmp_seq=2 ttl=60 time=77.043 ms
84 bytes from 10.4.20.11 icmp_seq=3 ttl=60 time=82.806 ms
84 bytes from 10.4.20.11 icmp_seq=4 ttl=60 time=83.158 ms
84 bytes from 10.4.20.11 icmp_seq=5 ttl=60 time=84.347 ms

```


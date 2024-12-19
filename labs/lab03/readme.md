### **Underlay. BGP**

-Настроить iBGP для Underlay сети

Для настройки iBGP будет использован стенд из 1-ой лабораторной работы.

### **План работы:**
1. Настроить **ptp** между spine и leaf в соответствии со адресацией и схемой;
2. Настроить **Loopback** интерфейсы на spine и leaf (для использования BGP в качестве RID);
3. Включить функционал маршрутизации на spine и leaf командой **ip routing**
4. Создадим route-map и приматчим **interface Loopback1**(в дальнейшем будет использован для редистрибьюции loopback1;
5. Включить процесс **BGP**
6. Выполнить команду router-id *адрес loopback1*;
7. На Spine, воспользуемся **peer group**, для упрощения конфигруации (все настройки принадлежащие одной **peer group** будут применяться для установления соседсва; **neighbor *dc1* peer group**
8. Зададим номер AS из приватного дипазона  neighbor dc1 **remote-as 65000**
9. Зададим таймеры *Keepalive Timer* и *Hold* neighbor dc1 **timers 3 9**
10. Зададим пароль который будет использован для поднятия сессии с соседом neighbor dc1 **password 7 arista**
11.Для избежания петель в **IBGP** используется связанность *Full Mesh*, т.к. при передаче маршрута внутри автономной системы AS-path не меняется. Но так как у нас не *Full Mesh*, для решения данный проблемы поможет включение функционала Route Reflector **neighbor dc1 route-reflector-client**, все полученные маршруты будут переданы на все остальные маршрутизаторы в сехеме.
12. В IBGP чтоб соседи поместили маршрут в таблицу маршрутизации необходимо чтобы у получателя такого анонса был маршрут до Next-Hop, сделать это можно командой **neighbor dc1 next-hop-self**
13. Настроим автоматическое обнаружение соседсва в соотвествии с префиксом подсети и настроек *peer group dc1* **bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000**
14. Анонсируем адрес Loopback1 в в таблицу маршрутизаци **redistribute connected route-map redistrib_connect**;
15. **Настрока spine завершена!**

На Leaf:
1. Включить процесс **BGP**
2. Выполнить команду router-id *адрес loopback1*;
3. Cделаем настройки peer group аналогичные **Spine**
```
neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7
```
4. В явном виде укажем в конфигурации соседей **Spine**
```
    neighbor 10.2.1.0 peer group dc1
    neighbor 10.2.2.0 peer group dc1
```
5. Анонсируем connected сети командой network
6. Для настройки ECMP используем команду **maximum-paths 2**. Т.к. по умолчанию BGP заносит в таблицу маршрутизации только лучший маршрут. Значение 2, т.к. на схеме 2 Spine.
7. Анонсируем адрес Loopback1 в в таблицу маршрутизаци **redistribute connected route-map redistrib_connect**
Проверим таблицы маршрутизации (они должны содержать все анонсированные connected сети и адрес loopback1, каждого устройства.  


**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/ptp%20network.png)

**Таблица 3 подсетей хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/host-network.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03/as%2065000.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/eve-ng-scheme.png)



## Конфигурация BGP dc1-spine1

```
router bgp 65000
   router-id 10.0.1.0
   bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 next-hop-self
   neighbor dc1 route-reflector-client
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   redistribute connected route-map redistrib_connect


interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.1.0/31
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.1.2/31
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.1.4/31
!
interface Loopback1
   ip address 10.0.1.0/32
!
interface Management1
!
ip routing
!
route-map redistrib_connect permit 10
   match interface Loopback1
```

## Конфигурация BGP dc1-spine2

```
router bgp 65000
   router-id 10.0.2.0
   bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 next-hop-self
   neighbor dc1 route-reflector-client
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   redistribute connected route-map redistrib_connect

interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.2.0/31
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.2.2/31
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.2.4/31
!

interface Loopback1
   ip address 10.0.2.0/32
!
interface Management1
!
ip routing
!
route-map redistrib_connect permit 10
   match interface Loopback1

```
## Конфигурация BGP dc1-leaf1
```
router bgp 65000
   router-id 10.0.0.1
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor 10.2.1.0 peer group dc1
   neighbor 10.2.2.0 peer group dc1
   network 10.4.1.0/24
   redistribute connected route-map redistrib_connect
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

interface Ethernet8
   switchport access vlan 10
!
interface Loopback1
   ip address 10.0.0.1/32
!
interface Management1
!
interface Vlan10
   ip address 10.4.1.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
!
ip routing
!
route-map redistrib_connect permit 10
   match interface Loopback1

```
## Конфигурация BGP dc1-leaf2
```
router bgp 65000
   router-id 10.0.0.2
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor 10.2.1.2 peer group dc1
   neighbor 10.2.2.2 peer group dc1
   network 10.4.2.0/24
   redistribute connected route-map redistrib_connect

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
interface Ethernet8
   switchport access vlan 20
!
interface Loopback1
   ip address 10.0.0.2/32
!
interface Management1
!
interface Vlan20
   description esxi-host
   ip address 10.4.2.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
!
ip routing
!
route-map redistrib_connect permit 10
   match interface Loopback1

```
## Конфигурация BGP dc1-leaf3
```
router bgp 65000
   router-id 10.0.0.3
   maximum-paths 2
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor 10.2.1.4 peer group dc1
   neighbor 10.2.2.4 peer group dc1
   network 10.4.3.0/24
   redistribute connected route-map redistrib_connect
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
interface Ethernet7
   switchport access vlan 30
!
interface Ethernet8
   switchport access vlan 30
!
interface Loopback1
   ip address 10.0.0.3/32
!
interface Management1
!
interface Vlan30
   description esxi-host
   ip address 10.4.3.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
!
ip routing
!
route-map redistrib_connect permit 10
   match interface Loopback1
```
## Вывод команд show bgp evpn summary, show bgp evpn и show bgp evpn route-type mac-ip dc1-spine1

```
dc1-spine1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1 4 65000           1616      1628    0    0 01:08:42 Estab   2      2
  10.1.0.2 4 65000           1601      1606    0    0 01:07:57 Estab   2      2
  10.1.0.3 4 65000           1597      1603    0    0 01:07:56 Estab   2      2

dc1-spine1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:100 mac-ip 0050.7966.6802
                                 10.1.0.1              -       100     0       i
 * >      RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i
 * >      RD: 10.0.0.3:100 mac-ip 0050.7966.680a
                                 10.1.0.3              -       100     0       i
 * >      RD: 10.0.0.1:100 imet 10.1.0.1
                                 10.1.0.1              -       100     0       i
 * >      RD: 10.0.0.2:100 imet 10.1.0.2
                                 10.1.0.2              -       100     0       i
 * >      RD: 10.0.0.3:100 imet 10.1.0.3
                                 10.1.0.3              -       100     0       i

dc1-spine1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.0.1:100 mac-ip 0050.7966.6802
                                 10.1.0.1              -       100     0       i
 * >      RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i
 * >      RD: 10.0.0.3:100 mac-ip 0050.7966.680a
                                 10.1.0.3              -       100     0       i


```
## Вывод команд show bgp evpn summary, show bgp evpn и show bgp evpn route-type mac-ip dc1-leaf3
```
dc1-leaf3#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.0.3, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           1718      1700    0    0 01:09:55 Estab   4      4
  10.1.2.0 4 65000           1717      1700    0    0 01:09:42 Estab   4      4

dc1-leaf3#show bgp evpn
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
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 * >      RD: 10.0.0.3:100 mac-ip 0050.7966.680a
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.0.1:100 imet 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.1:100 imet 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 * >Ec    RD: 10.0.0.2:100 imet 10.1.0.2
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec    RD: 10.0.0.2:100 imet 10.1.0.2
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 * >      RD: 10.0.0.3:100 imet 10.1.0.3

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
 * >Ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 *  ec    RD: 10.0.0.2:100 mac-ip 0050.7966.6808
                                 10.1.0.2              -       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 * >      RD: 10.0.0.3:100 mac-ip 0050.7966.680a
                                 -                     -       -       0       i

```

### **Проверка доcтупности узлов**
---
## host3 to loopback1 dc1-spine1, spine2, leaf1, leaf2, host 1,2
```
VPCS> ping 10.0.1.0

84 bytes from 10.0.1.0 icmp_seq=1 ttl=63 time=21.370 ms
84 bytes from 10.0.1.0 icmp_seq=2 ttl=63 time=16.367 ms
84 bytes from 10.0.1.0 icmp_seq=3 ttl=63 time=18.376 ms
84 bytes from 10.0.1.0 icmp_seq=4 ttl=63 time=16.744 ms
^C
VPCS> ping 10.0.2.0

84 bytes from 10.0.2.0 icmp_seq=1 ttl=63 time=17.696 ms
84 bytes from 10.0.2.0 icmp_seq=2 ttl=63 time=17.297 ms
84 bytes from 10.0.2.0 icmp_seq=3 ttl=63 time=16.024 ms
84 bytes from 10.0.2.0 icmp_seq=4 ttl=63 time=16.977 ms
^C
VPCS> ping 10.0.0.1

84 bytes from 10.0.0.1 icmp_seq=1 ttl=62 time=27.207 ms
84 bytes from 10.0.0.1 icmp_seq=2 ttl=62 time=44.234 ms
84 bytes from 10.0.0.1 icmp_seq=3 ttl=62 time=25.547 ms
^C
VPCS> ping 10.0.0.2

84 bytes from 10.0.0.2 icmp_seq=1 ttl=62 time=27.058 ms
84 bytes from 10.0.0.2 icmp_seq=2 ttl=62 time=28.238 ms
84 bytes from 10.0.0.2 icmp_seq=3 ttl=62 time=39.399 ms
84 bytes from 10.0.0.2 icmp_seq=4 ttl=62 time=28.071 ms

VPCS> ping 10.4.1.2

84 bytes from 10.4.1.2 icmp_seq=1 ttl=61 time=58.593 ms
84 bytes from 10.4.1.2 icmp_seq=2 ttl=61 time=32.760 ms
84 bytes from 10.4.1.2 icmp_seq=3 ttl=61 time=33.329 ms
84 bytes from 10.4.1.2 icmp_seq=4 ttl=61 time=36.893 ms

VPCS> ping 10.4.2.2

84 bytes from 10.4.2.2 icmp_seq=1 ttl=61 time=72.740 ms
84 bytes from 10.4.2.2 icmp_seq=2 ttl=61 time=32.113 ms
84 bytes from 10.4.2.2 icmp_seq=3 ttl=61 time=30.890 ms
84 bytes from 10.4.2.2 icmp_seq=4 ttl=61 time=32.290 ms
84 bytes from 10.4.2.2 icmp_seq=5 ttl=61 time=30.585 ms

```
## dc1-spine1 to host1,2,3 и loopback 1 (leaf1,2,3)
```
dc1-spine1#ping 10.4.1.2 source loopback 1
PING 10.4.1.2 (10.4.1.2) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.4.1.2: icmp_seq=1 ttl=63 time=24.3 ms
80 bytes from 10.4.1.2: icmp_seq=2 ttl=63 time=19.0 ms
80 bytes from 10.4.1.2: icmp_seq=3 ttl=63 time=15.1 ms
80 bytes from 10.4.1.2: icmp_seq=4 ttl=63 time=12.4 ms
80 bytes from 10.4.1.2: icmp_seq=5 ttl=63 time=17.0 ms

dc1-spine1#ping 10.4.2.2 source loopback 1
PING 10.4.2.2 (10.4.2.2) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.4.2.2: icmp_seq=1 ttl=63 time=46.3 ms
80 bytes from 10.4.2.2: icmp_seq=2 ttl=63 time=40.7 ms
80 bytes from 10.4.2.2: icmp_seq=3 ttl=63 time=36.3 ms
80 bytes from 10.4.2.2: icmp_seq=4 ttl=63 time=30.8 ms
80 bytes from 10.4.2.2: icmp_seq=5 ttl=63 time=13.5 ms

dc1-spine1#ping 10.4.3.2 source loopback 1
PING 10.4.3.2 (10.4.3.2) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.4.3.2: icmp_seq=1 ttl=63 time=35.0 ms
80 bytes from 10.4.3.2: icmp_seq=2 ttl=63 time=30.0 ms
80 bytes from 10.4.3.2: icmp_seq=3 ttl=63 time=26.2 ms
80 bytes from 10.4.3.2: icmp_seq=4 ttl=63 time=27.2 ms
80 bytes from 10.4.3.2: icmp_seq=5 ttl=63 time=20.4 ms

dc1-spine1#ping 10.0.0.1 source loopback 1
PING 10.0.0.1 (10.0.0.1) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=24.2 ms
80 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=16.2 ms
80 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=13.5 ms
80 bytes from 10.0.0.1: icmp_seq=4 ttl=64 time=9.98 ms
80 bytes from 10.0.0.1: icmp_seq=5 ttl=64 time=7.06 ms

dc1-spine1#ping 10.0.0.2 source loopback 1
PING 10.0.0.2 (10.0.0.2) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=11.3 ms
80 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=12.1 ms
80 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=10.4 ms
80 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=13.1 ms
80 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=9.32 ms

dc1-spine1#ping 10.0.0.3 source loopback 1
PING 10.0.0.3 (10.0.0.3) from 10.0.1.0 : 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=11.8 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=10.7 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=64 time=9.50 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=64 time=10.1 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=64 time=12.5 ms

```

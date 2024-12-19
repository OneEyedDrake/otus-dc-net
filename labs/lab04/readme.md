### **VxLAN. L2 VNI**

-Настроить VxLAN. L2 VNI

Для настройки VxLAN. L2 VNI будет использован стенд из 3-ой лабораторной работы, с iBGP в качестве Underlay сети.

### **План работы:**
1. Настроить **ptp** между spine и leaf в соответствии со адресацией и схемой;
2. Настроить **Loopback** интерфейсы **lo1** и **lo2** на spine и leaf (для использования в Underlay и Overlay сетях соответсвенно)
4. Создадим ip-префикc для анонса адресов **lo1-2** интефейсов *ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32* и route-map и приматчим **prefix-list loopback**(в дальнейшем будет использован для редистрибьюции loopback1-2);
5. Включим поддержку EVPN на Arista командой **service routing protocols model multi-agent**
6. Используем ранее созданный **BGP процесс 65000**, добавим настройки для Overlay сети
7. На Spine, воспользуемся **peer group**, для упрощения конфигруации (все настройки принадлежащие одной **peer group** будут применяться для установления соседсва; **neighbor *overlay* peer group**
9. Зададим номер AS из приватного дипазона  neighbor overlay **remote-as 65000**
10. Зададим таймеры *Keepalive Timer* и *Hold* neighbor overlay **timers 3 9**
11. Зададим пароль который будет использован для поднятия сессии с соседом neighbor overlay **password 7 arista-overlay**
12. Для передачи EVPN маршрутов между leaf включим функционал Route Reflector **neighbor overlay route-reflector-client**, все полученные маршруты будут переданы на все остальные маршрутизаторы в сехеме.
13. Настроим автоматическое обнаружение соседсва в соотвествии с префиксом подсети и настроек *peer group overlay* **bgp listen range 10.1.0.0/16 peer-group overlay remote-as 65000**
14. Включим поддрежку расширенного комьюнити для передачи EVPN информации **neighbor overlay send-community extended**
15. В качестве интерфейса для BGP сессии в overlay используем lo2 **neighbor overlay update-source Loopback2**
16. Активируем возможноть передачи EVPN информации:
```
  address-family evpn
      neighbor overlay activate
```
17. Анонсируем адреса Loopback1-2 в в таблицу маршрутизаци **redistribute connected route-map redistrib_connect**;
18. **Настрока spine завершена!**

На Leaf:
1. Используем ранее созданный **BGP процесс 65000**, добавим настройки для Overlay сети
3. Cделаем настройки peer group *overlay* аналогичные **Spine**
```
neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay timers 3 9
   neighbor overlay password 7 
   neighbor overlay send-community extended

```
4. В явном виде укажем в конфигурации соседей **Spine**
```
    neighbor 10.2.1.0 peer group overlay
    neighbor 10.2.2.0 peer group overlay
```
5. Анонсируем адреса Loopback1-2 в в таблицу маршрутизаци **redistribute connected route-map redistrib_connect**;
6. Создадим *vlan 100*, для дальнешего подключения к vxlan (VLAN-based VXLAN);
7. Добавим порт в который подключены хосты в *vlan 100*, **switchport access vlan 100**
8. Создадим VTI интерфейс **interface Vxlan1**
9. Настроим интерфейс котоырый будет использован для организации VXLAN туннеля **vxlan source-interface Loopback2**
10. Выбор порта VXLAN **vxlan udp-port 4789**
11. Мапинг VLAN к VXLAN **vxlan vlan 100 vni 100**
12. Разрешить для обнаружения VTEP использовать BGP и static режимы  **vxlan learn-restrict any**
13. Внесем дополнительные настройки в BGP для передачи информации о vxlan:
    - Route Distinquishers **rd auto** будет сформирован автоматически;
    - Route-target **route-target both 65000:100** должен совпадать на всех leaf для этого **vxlan**
    - Включить изучение mac в overlay, для данного *vxlan* **mac redistribute learned**
14.  **Настрока leaf завершена!**, можно проверять дотупность host1 до host2.




**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/ptp%20network.png)

**Таблица 3 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/add%20hosts.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/as%2065000.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/eve-ng-scheme.png)

## Конфигурация BGP dc1-spine1

```
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
interface Loopback2
   ip address 10.1.1.0/32
!
interface Management1
!
ip routing
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.1.0
   bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000
   bgp listen range 10.1.0.0/16 peer-group overlay remote-as 65000
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 next-hop-self
   neighbor dc1 route-reflector-client
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay route-reflector-client
   neighbor overlay timers 3 9
   neighbor overlay password 7 wnmhtPzjs2lTHT7Eds39N3BzyPA2LfZA
   neighbor overlay send-community extended
   redistribute connected route-map redistrib_connect
   !
   address-family evpn
      neighbor overlay activate
!

```

## Конфигурация BGP dc1-spine2

```
spanning-tree mode mstp
!
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
interface Loopback2
   ip address 10.1.2.0/32
!
interface Management1
!
ip routing
!
ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32
!
route-map redistrib_connect permit 10
   match ip address prefix-list loopback
!
router bgp 65000
   router-id 10.0.2.0
   bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000
   bgp listen range 10.1.0.0/16 peer-group overlay remote-as 65000
   neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 next-hop-self
   neighbor dc1 route-reflector-client
   neighbor dc1 timers 3 9
   neighbor dc1 password 7 vYrGBbkC/fzfCBVXbtDK4w==
   neighbor overlay peer group
   neighbor overlay remote-as 65000
   neighbor overlay update-source Loopback2
   neighbor overlay route-reflector-client
   neighbor overlay timers 3 9
   neighbor overlay password 7 wnmhtPzjs2lTHT7Eds39N3BzyPA2LfZA
   neighbor overlay send-community extended
   redistribute connected route-map redistrib_connect
   !
   address-family evpn
      neighbor overlay activate
!

```
## Конфигурация BGP dc1-leaf1
```
vlan 100
   name vxlan100-cli
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
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 100
   vxlan learn-restrict any
!
ip routing
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

```
## Конфигурация BGP dc1-leaf2
```
vlan 100
   name vxlan100-cli
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
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 100
   vxlan learn-restrict any
!
ip routing
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

```
## Конфигурация BGP dc1-leaf3
```
vlan 100
   name vxlan-client
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
   switchport access vlan 100
!
interface Ethernet8
   switchport access vlan 100
!
interface Loopback1
   ip address 10.0.0.3/32
!
interface Loopback2
   ip address 10.1.0.3/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan 100 vni 100
   vxlan learn-restrict any
!
ip routing
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
   address-family evpn
      neighbor overlay activate
!
end
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
## host3 to host1 и host2
```
host3> ping 10.4.100.11

84 bytes from 10.4.100.11 icmp_seq=1 ttl=64 time=125.889 ms
84 bytes from 10.4.100.11 icmp_seq=2 ttl=64 time=47.588 ms
84 bytes from 10.4.100.11 icmp_seq=3 ttl=64 time=34.994 ms
84 bytes from 10.4.100.11 icmp_seq=4 ttl=64 time=35.895 ms
84 bytes from 10.4.100.11 icmp_seq=5 ttl=64 time=33.612 ms

host3> ping 10.4.100.12

84 bytes from 10.4.100.12 icmp_seq=1 ttl=64 time=54.599 ms
84 bytes from 10.4.100.12 icmp_seq=2 ttl=64 time=38.559 ms
84 bytes from 10.4.100.12 icmp_seq=3 ttl=64 time=35.291 ms
84 bytes from 10.4.100.12 icmp_seq=4 ttl=64 time=32.934 ms
84 bytes from 10.4.100.12 icmp_seq=5 ttl=64 time=36.670 ms
```
## host2 to host1 и host3

```
host2> ping 10.4.100.11

84 bytes from 10.4.100.11 icmp_seq=1 ttl=64 time=37.645 ms
84 bytes from 10.4.100.11 icmp_seq=2 ttl=64 time=42.218 ms
84 bytes from 10.4.100.11 icmp_seq=3 ttl=64 time=34.512 ms
84 bytes from 10.4.100.11 icmp_seq=4 ttl=64 time=40.211 ms
84 bytes from 10.4.100.11 icmp_seq=5 ttl=64 time=40.984 ms

host2> ping 10.4.100.13

84 bytes from 10.4.100.13 icmp_seq=1 ttl=64 time=34.271 ms
84 bytes from 10.4.100.13 icmp_seq=2 ttl=64 time=84.336 ms
84 bytes from 10.4.100.13 icmp_seq=3 ttl=64 time=35.141 ms
84 bytes from 10.4.100.13 icmp_seq=4 ttl=64 time=42.346 ms
84 bytes from 10.4.100.13 icmp_seq=5 ttl=64 time=30.644 ms
```
## host1 to host2 и host3

```
host1> ping 10.4.100.12

84 bytes from 10.4.100.12 icmp_seq=1 ttl=64 time=35.724 ms
84 bytes from 10.4.100.12 icmp_seq=2 ttl=64 time=65.572 ms
84 bytes from 10.4.100.12 icmp_seq=3 ttl=64 time=41.538 ms
84 bytes from 10.4.100.12 icmp_seq=4 ttl=64 time=29.584 ms
84 bytes from 10.4.100.12 icmp_seq=5 ttl=64 time=35.489 ms

host1> ping 10.4.100.13

84 bytes from 10.4.100.13 icmp_seq=1 ttl=64 time=30.009 ms
84 bytes from 10.4.100.13 icmp_seq=2 ttl=64 time=30.638 ms
84 bytes from 10.4.100.13 icmp_seq=3 ttl=64 time=33.904 ms
84 bytes from 10.4.100.13 icmp_seq=4 ttl=64 time=30.549 ms
84 bytes from 10.4.100.13 icmp_seq=5 ttl=64 time=38.930 ms
```

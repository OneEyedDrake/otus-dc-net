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

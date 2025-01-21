### **Underlay. IS-IS**

-Настроить IS-IS для Underlay сети

Для настройки IS-IS будет использован стенд из 1-ой лабораторной работы.

### **План работы:**
1. Настроить **ptp** между spine и leaf в соответствии со адресацией и схемой;
2. Настроить **Loopback** интерфейсы на spine и leaf (для использования ISIS в качестве RID и переобразования в SYSID);
3. Включить функционал маршрутизации на spine и leaf командой **ip routing**
4. Включить процесс **isis**
5. Выполнить команду router-id ipv4 *адрес loopback1* ;
6. Настроить Network Entity Title уникальный адрес назначается маршрутизатору при запуске процесса ISIS(необходим для опеределения AREA ID и SYSID) net 49.0001.0100.0000.2000.00; 
7. Настроить аутентификацию для построения соседства(глобально для процеса ISIS: **authentication mode md5 authentication key 7 password**;
8. Включить передачу маршрутной инфофрмации IPv4 **address-family ipv4 unicast**
9. Включить is-type level-2 (соседство уровня 2 (L2) может быть сформировано как между маршрутизаторами одной area, так и между маршрутизаторами разных area) В нашем случае не пренципиально, т.к. зона будет только одна AREA 49.0001.
10. Включить ISIS на необходимых интерфейсах командой  isis enable *имя процесса isis*, на ptp линках дополнительно указать тип подключения **isis network point-to-point**;
11. Проверить командой show isis neighbor, состояние соседства;
12. Проверить командой show ip isis database базу данных состояни каналов, соседство только 2го уровня(l2), факстически все маршрутизаторы находятся в зоне backbone и все ptp линки в режиме network point-to-point.
13. Проверить доступность командой ping с разных устройств.

**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03.00(is-is)/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03.00(is-is)/ptp%20network.png)

**Таблица 3 подсетей хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03.00(is-is)/host-network.png)

### **Cхема с адресацией и указанием зоны IS-IS**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03.00(is-is)/draw.io.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab03.00(is-is)/eve-ng-scheme.png)

## Конфигурация OSPF dc1-spine1

```
hostname dc1-spine1
!
spanning-tree mode mstp
!
interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.1.0/31
   isis enable spine1
   isis network point-to-point
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.1.2/31
   isis enable spine1
   isis network point-to-point
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.1.4/31
   isis enable spine1
   isis network point-to-point
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
   ip address 10.0.1.0/32
   isis enable spine1
!
interface Loopback2
   ip address 10.1.1.0/32
   isis enable spine1
!
interface Management1
!
ip routing
!
router isis spine1
   net 49.0001.0100.0000.1000.00
   router-id ipv4 10.0.1.0
   is-type level-2
   authentication mode md5
   authentication key 7 /Beytnuj8KSjk59dyO0v4w==
   !
   address-family ipv4 unicast
!
end
```

## Конфигурация OSPF dc1-spine2

```
hostname dc1-spine2
!
spanning-tree mode mstp
!
interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.2.0/31
   isis enable spine2
   isis network point-to-point
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.2.2/31
   isis enable spine2
   isis network point-to-point
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.2.4/31
   isis enable spine2
   isis network point-to-point
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
   ip address 10.0.2.0/32
   isis enable spine2
!
interface Loopback2
   ip address 10.1.2.0/32
   isis enable spine2
!
interface Management1
!
ip routing
!
router isis spine2
   net 49.0001.0100.0000.2000.00
   router-id ipv4 10.0.2.0
   authentication mode md5
   authentication key 7 RyBQ5L1LcNK2FQY7BaOXIA==
   !
   address-family ipv4 unicast
!
end
```
## Конфигурация OSPF dc1-leaf1
```
hostname dc1-leaf1
!
spanning-tree mode mstp
!
vlan 10
   name cli
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.1/31
   isis enable leaf1
   isis network point-to-point
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.1/31
   isis enable leaf1
   isis network point-to-point
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
   switchport access vlan 10
!
interface Loopback1
   ip address 10.0.0.1/32
   isis enable leaf1
!
interface Loopback2
   ip address 10.1.0.1/32
   isis enable leaf1
!
interface Management1
!
interface Vlan10
   ip address 10.4.1.1/24
   isis enable leaf1
!
ip routing
!
router isis leaf1
   net 49.0001.0100.0200.1001.00
   router-id ipv4 10.0.0.1
   is-type level-2
   authentication mode md5
   authentication key 7 GNzsn4SnrcllcDnRQiQRQw==
   !
   address-family ipv4 unicast
!
end

```
## Конфигурация OSPF dc1-leaf2
```
hostname dc1-leaf2
!
spanning-tree mode mstp
!
vlan 20
   name cli
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.3/31
   isis enable leaf2
   isis network point-to-point
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.3/31
   isis enable leaf2
   isis network point-to-point
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
   switchport access vlan 20
!
interface Loopback1
   ip address 10.0.0.2/32
   isis enable leaf2
!
interface Loopback2
   ip address 10.1.0.2/32
   isis enable leaf2
!
interface Management1
!
interface Vlan20
   ip address 10.4.2.1/24
   isis enable leaf2
!
ip routing
!
router isis leaf2
   net 49.0001.0100.0200.1002.00
   router-id ipv4 10.0.0.2
   is-type level-2
   authentication mode md5
   authentication key 7 lc/p9Jp732vdBZ9MX/NaRQ==
   !
   address-family ipv4 unicast
!

```
## Конфигурация OSPF dc1-leaf3
```
hostname dc1-leaf3
!
spanning-tree mode mstp
!
vlan 30
   name cli
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.5/31
   isis enable leaf3
   isis network point-to-point
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.5/31
   isis enable leaf3
   isis network point-to-point
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
   switchport access vlan 30
!
interface Ethernet8
   switchport access vlan 30
!
interface Loopback1
   ip address 10.0.0.3/32
   isis enable leaf3
!
interface Loopback2
   ip address 10.1.0.3/32
   isis enable leaf3
!
interface Management1
!
interface Vlan30
   ip address 10.4.3.1/24
   isis enable leaf3
!
ip routing
!
router isis leaf3
   net 49.0001.0100.0200.1003.00
   router-id ipv4 10.0.0.3
   is-type level-2
   authentication mode md5
   authentication key 7 lc/p9Jp732vdBZ9MX/NaRQ==
   !
   address-family ipv4 unicast
!
end

```
## Вывод команд show isis neighbor и show isis database, show isis hostname, show ip route isis на dc1-spine1
```
dc1-spine1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
spine1    default  dc1-leaf1        L2   Ethernet1          P2P               UP    25          0F
spine1    default  dc1-leaf2        L2   Ethernet2          P2P               UP    24          0F
spine1    default  dc1-leaf3        L2   Ethernet3          P2P               UP    30          0F


dc1-spine1#show isis database

IS-IS Instance: spine1 VRF: default
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    dc1-spine1.00-00              8  46719   589    182 L2 <>
    dc1-spine2.00-00              7  14807   880    182 L2 <>
    dc1-leaf1.00-00               8  20973   968    169 L2 <>
    dc1-leaf2.00-00               8  42155   773    169 L2 <>
    dc1-leaf3.00-00               7  22135   528    169 L2 <>

dc1-spine1#show isis hostname

IS-IS Instance: spine1 VRF: default
Level  System ID           Hostname
L2     0100.0000.1000      dc1-spine1
L2     0100.0000.2000      dc1-spine2
L2     0100.0200.1001      dc1-leaf1
L2     0100.0200.1002      dc1-leaf2
L2     0100.0200.1003      dc1-leaf3

dc1-spine1#sh ip route isis

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

 I L2     10.0.0.1/32 [115/20] via 10.2.1.1, Ethernet1
 I L2     10.0.0.2/32 [115/20] via 10.2.1.3, Ethernet2
 I L2     10.0.0.3/32 [115/20] via 10.2.1.5, Ethernet3
 I L2     10.0.2.0/32 [115/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 I L2     10.1.0.1/32 [115/20] via 10.2.1.1, Ethernet1
 I L2     10.1.0.2/32 [115/20] via 10.2.1.3, Ethernet2
 I L2     10.1.0.3/32 [115/20] via 10.2.1.5, Ethernet3
 I L2     10.1.2.0/32 [115/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 I L2     10.2.2.0/31 [115/20] via 10.2.1.1, Ethernet1
 I L2     10.2.2.2/31 [115/20] via 10.2.1.3, Ethernet2
 I L2     10.2.2.4/31 [115/20] via 10.2.1.5, Ethernet3
 I L2     10.4.1.0/24 [115/20] via 10.2.1.1, Ethernet1
 I L2     10.4.2.0/24 [115/20] via 10.2.1.3, Ethernet2
 I L2     10.4.3.0/24 [115/20] via 10.2.1.5, Ethernet3


```
## Вывод команд show isis neighbor и show isis database, show isis hostname, show ip route isis на dc1-leaf3
```
dc1-leaf3#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
leaf3     default  dc1-spine1       L2   Ethernet1          P2P               UP    24          11
leaf3     default  dc1-spine2       L2   Ethernet2          P2P               UP    25          11

dc1-leaf3#show isis database

IS-IS Instance: leaf3 VRF: default
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    dc1-spine1.00-00              9  65184  1149    182 L2 <>
    dc1-spine2.00-00              7  14807   662    182 L2 <>
    dc1-leaf1.00-00               8  20973   751    169 L2 <>
    dc1-leaf2.00-00               8  42155   556    169 L2 <>
    dc1-leaf3.00-00               8  49496  1169    169 L2 <>

dc1-leaf3#show isis hostname

IS-IS Instance: leaf3 VRF: default
Level  System ID           Hostname
L2     0100.0000.1000      dc1-spine1
L2     0100.0000.2000      dc1-spine2
L2     0100.0200.1001      dc1-leaf1
L2     0100.0200.1002      dc1-leaf2
L2     0100.0200.1003      dc1-leaf3

dc1-leaf3#show ip route isis

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

 I L2     10.0.0.1/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L2     10.0.0.2/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L2     10.0.1.0/32 [115/20] via 10.2.1.4, Ethernet1
 I L2     10.0.2.0/32 [115/20] via 10.2.2.4, Ethernet2
 I L2     10.1.0.1/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L2     10.1.0.2/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L2     10.1.1.0/32 [115/20] via 10.2.1.4, Ethernet1
 I L2     10.1.2.0/32 [115/20] via 10.2.2.4, Ethernet2
 I L2     10.2.1.0/31 [115/20] via 10.2.1.4, Ethernet1
 I L2     10.2.1.2/31 [115/20] via 10.2.1.4, Ethernet1
 I L2     10.2.2.0/31 [115/20] via 10.2.2.4, Ethernet2
 I L2     10.2.2.2/31 [115/20] via 10.2.2.4, Ethernet2
 I L2     10.4.1.0/24 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L2     10.4.2.0/24 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2


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
## dc1-spine1 to host1,2,3 и loopback 1 (leaf1)
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

```


### **Underlay. OSPF**

-Настроить OSPF для Underlay сети

Для настройки OSPF будет использован стенд из 1-ой лабораторной работы.

### **План работы:**
1. Настроить **ptp** между spine и leaf в соответствии со адресацией и схемой;
2. Настроить **Loopback** интерфейсы на spine и leaf (для использования OSPF в качестве RID);
3. Включить функционал маршрутизации на spine и leaf командой **ip routing**
4. Включить процесс **ospf**
5. Выполнить команду router-id *адрес loopback1* ;
6. Выполнить команду passive-interface default, чтоб по умолчанию ни один интерфейс не отправлял служебную инфомацию протокола OSPF для построения соседства;
7. Выполнить команду no passive-interface *"наименование интерфейса"*, для интерфейсов которые будут отправлять и получать служебную инфомацию протокола OSPF для построения соседства, будет включено на всех **ptp** линках;
8. Выполнить команду на интерфейсах из пункта 7 ip ospf network point-to-point для переключения интерфейса, в режим работы point-to-point, т.к. в схеме не требуется LSA 2 типа, для Brodcast сетей. Все подключения выполнены точка-точка;
9. Добавить интерфесы в процесс ospf **ip ospf area 0.0.0.0**, для анонса и получения марштурной информации о подсетях находящися на данных интерфесах:
- ptp интерфейсы;
- loopback1 интерфесы;
- interface vlan **X** для анонса подсетей хостов;
10. Проверить командой show ip ospf neighbor, состояние соседства;
11. Проверить командой show ip ospf database базу данных состояни каналов, должны быть LSA только 1го типа, т.к. все маршрутизаторы находятся в зоне 0.0.0.0 и все линки в режиме network point-to-point.
12. Проверить доступность командой ping с разных устройств.

**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/ptp%20network.png)

**Таблица 3 подсетей хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/host-network.png)

### **Cхема с адресацией и указанием зоны OSPF**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/draw.io.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab02/eve-ng-scheme.png)

## Конфигурация OSPF dc1-spine1

```
router ospf 1
   router-id 10.0.1.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000

interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.1.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.1.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.1.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.1.0/32
   ip ospf area 0.0.0.0
```

## Конфигурация OSPF dc1-spine2

```
router ospf 1
   router-id 10.0.2.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
interface Ethernet1
   description to_dc1-leaf1
   no switchport
   ip address 10.2.2.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_dc1-leaf2
   no switchport
   ip address 10.2.2.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description to_dc1-leaf3
   no switchport
   ip address 10.2.2.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.2.0/32
   ip ospf area 0.0.0.0
```
## Конфигурация OSPF dc1-leaf1
```
router ospf 1
   router-id 10.0.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
!
interface Loopback1
   ip address 10.0.0.1/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   ip address 10.4.1.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
```
## Конфигурация OSPF dc1-leaf2
```
router ospf 1
   router-id 10.0.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.0.2/32
   ip ospf area 0.0.0.0
!
interface Vlan20
   description esxi-host
   ip address 10.4.2.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
```
## Конфигурация OSPF dc1-leaf3
```
router ospf 1
   router-id 10.0.0.3
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
interface Ethernet1
   description to_dc1-spine1
   no switchport
   ip address 10.2.1.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description to_dc1-spine2
   no switchport
   ip address 10.2.2.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.0.0.3/32
   ip ospf area 0.0.0.0
!
interface Vlan30
   description esxi-host
   ip address 10.4.3.1/24
   dhcp server ipv4
   ip ospf area 0.0.0.0
```
## Вывод команд show ip ospf neighbor и show ip ospf database dc1-spine1
```
dc1-spine1#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.0.1        1        default  0   FULL                   00:00:33    10.2.1.1        Ethernet1
10.0.0.2        1        default  0   FULL                   00:00:34    10.2.1.3        Ethernet2
10.0.0.3        1        default  0   FULL                   00:00:36    10.2.1.5        Ethernet3
dc1-spine1#
dc1-spine1#show ip ospf database

            OSPF Router with ID(10.0.1.0) (Instance ID 1) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.0.0.1        10.0.0.1        820         0x80000011   0x3fdd   6
10.0.2.0        10.0.2.0        412         0x8000000b   0xf5ec   7
10.0.0.2        10.0.0.2        485         0x8000000b   0xab6b   6
10.0.0.3        10.0.0.3        96          0x8000000a   0xefd    6
10.0.1.0        10.0.1.0        436         0x80000012   0x875d   7
```
## Вывод команды show ip ospf neighbor и show ip ospf database dc1-spine1
```
dc1-spine1#show ip route ospf

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

 O        10.0.0.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.0.0.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.0.0.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.0.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 O        10.2.2.0/31 [110/20] via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.1.5, Ethernet3
 O        10.4.1.0/24 [110/20] via 10.2.1.1, Ethernet1
 O        10.4.2.0/24 [110/20] via 10.2.1.3, Ethernet2
 O        10.4.3.0/24 [110/20] via 10.2.1.5, Ethernet3
```

## Вывод команд show ip ospf neighbor и show ip ospf database dc1-leaf3
```
dc1-leaf3#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.0        1        default  0   FULL                   00:00:33    10.2.1.4        Ethernet1
10.0.2.0        1        default  0   FULL                   00:00:34    10.2.2.4        Ethernet2
dc1-leaf3#show ip ospf database

            OSPF Router with ID(10.0.0.3) (Instance ID 1) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.0.1.0        10.0.1.0        491         0x80000012   0x875d   7
10.0.2.0        10.0.2.0        466         0x8000000b   0xf5ec   7
10.0.0.2        10.0.0.2        540         0x8000000b   0xab6b   6
10.0.0.1        10.0.0.1        876         0x80000011   0x3fdd   6
10.0.0.3        10.0.0.3        150         0x8000000a   0xefd    6
```

## Вывод команды show ip ospf neighbor и show ip ospf database dc1-leaf3
```
dc1-leaf3#show ip route ospf

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

 O        10.0.0.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.0.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.4.1.0/24 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.4.2.0/24 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
```




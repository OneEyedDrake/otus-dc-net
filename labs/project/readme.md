# **Проектная работа**
## **Тема: "Построение сетевой фабрики на основе VxLAN EVPN с фильтрацией трафика на внешнем firewall"**

## **Цель:**
Реализовать разграничение на обмен информацией между сегментами сети, с возможнотью масштабирования и учетом миграции имеющегося оборудования и сервисов.  

## **Задачи:**
1) Повысить надежность сетевой инфраструктуры ЦОД;
2) Выполнение требований регуляторов по сегментации сети ЦОД, между сегментами сети содержащими общедоступную и конфеденциальную информацию;
3) Переключение имеющегося оборудования в новую сетевую фабрику;
4) Миграция сервисов, на VxLAN EVPN фабрику с фильтрацией трафика на Firewall;
## **Шаги:**
1) Построить параллельно имеющейся сетевой инфраструктуре, сетевую фабрику на базе VxLAN EVPN;
2) Подключить VPC пару, для "плавной" миграции сервисов и сетей на новую фабрику;
3) Произвести физическое переключение хостов виртуализации, напрямую в коммутаторы leaf;
4) Произвести переключение шасси на пару leaf;
5) Отключить старые ptp линки.

# **Используемые технологии:**
**VxLAN, EVPN, BGP, MLAG, LACP, OSPF, ESI LAG**
- Стек VxLAN, EVPN выбран т.к. он прекрасно ложится на реализацию сети по архитектуре Leaf-Spine(Clos), позволяет использовать все имеющиеся uplink-и, повышает надежноть за счет использования уровня spine(коммутаторы spine не соеденены между собой напрямую, и фактически не могут оказывать негативного влияния друг на друга в случае проблем с одним из устройств);
- OSPF в качестве протокола маршрутизации на уровне Underlay;
- IBGP для распространения маршрутной инфомрации внутри VxLAN/EVPN фабрики;
- ESI LAG для подключения хостов виртуализации;
- MLAG пара(border leaf) для подключения к паре Firewall Active-Passive и bypass линков к кампусной сети.



### Cхема до перехода на VxLAN EVPN с Firewall 

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/Scheme%20do1.png)

### Cхема "промежуточная" до полного перехода на VxLAN EVPN с Firewall

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/Scheme%20after1.png)

### Cхема итоговая все сервисы ходят через фабрику VxLAN EVPN с фильтрацией трафика на Firewall

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/scheme%20final1.png)

# **Описание практической части:**
- Практическая часть работы выполнялась в виртуальном стенде на базе EVE NG;
- Схема собрана для отражения технически навыков полученных на этапе обучения, но не в полном объеме показывает план миграции;
- В качестве пары Firewall Active-Passive, была использована Mlag пара с VIR(для маршрутизации исключетельно на VIP адресс и корректной работы Firewall, был использован roadmap с указанием ip next-hope);
- В качестве хоста виртуализации был использован обычный маршрутизатор Arista, с настроеным LACP, на стороне leaf настроен ESI LAG.
  
**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/eve-ng/address%20loop.png)

**Таблица 2 *IP адресов* ptp интерейсов spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/eve-ng/address%20ptp-spine.png)

**Таблица 3 *IP адресов* ptp интерейсов leaf**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/eve-ng/address%20ptp-leaf.png)

**Таблица 4 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/eve-ng/address%20hosts.png)

### **Cхема EVE-NG**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/eve-ng/eve-ng-main.png)

## Проверка информации о маршрутах на spine1
```
dc1-spine1#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/20] via 10.2.1.253, Ethernet7
 O        10.0.0.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.0.0.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.0.0.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.0.0.4/32 [110/20] via 10.2.1.7, Ethernet4
 O        10.0.0.254/32 [110/20] via 10.2.1.253, Ethernet7
 C        10.0.1.0/32 is directly connected, Loopback1
 O        10.0.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
                               via 10.2.1.7, Ethernet4
                               via 10.2.1.253, Ethernet7
 O        10.1.0.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.1.0.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.1.0.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.1.0.4/32 [110/20] via 10.2.1.7, Ethernet4
 O        10.1.0.254/32 [110/20] via 10.2.1.253, Ethernet7
 C        10.1.1.0/32 is directly connected, Loopback2
 O        10.1.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
                               via 10.2.1.7, Ethernet4
                               via 10.2.1.253, Ethernet7
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3
 C        10.2.1.6/31 is directly connected, Ethernet4
 C        10.2.1.252/31 is directly connected, Ethernet7
 C        10.2.1.254/31 is directly connected, Ethernet8
 O        10.2.2.0/31 [110/20] via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.1.5, Ethernet3
 O        10.2.2.6/31 [110/20] via 10.2.1.7, Ethernet4
 O        10.2.2.252/31 [110/20] via 10.2.1.253, Ethernet7
 O        10.2.2.254/31 [110/30] via 10.2.1.1, Ethernet1
                                 via 10.2.1.3, Ethernet2
                                 via 10.2.1.5, Ethernet3
                                 via 10.2.1.7, Ethernet4
                                 via 10.2.1.253, Ethernet7

dc1-spine1#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.1.0, local AS number 65000
Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
---------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.0.1         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.1         65000 Established   L2VPN EVPN              Negotiated              8          8
10.1.0.2         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.2         65000 Established   L2VPN EVPN              Negotiated              5          5
10.1.0.3         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.3         65000 Established   L2VPN EVPN              Negotiated              9          9
10.1.0.4         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.4         65000 Established   L2VPN EVPN              Negotiated              7          7
10.1.0.254       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.254       65000 Established   L2VPN EVPN              Negotiated              8          8


dc1-spine1#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1   4 65000           1486      1518    0    0 01:03:03 Estab   8      8
  10.1.0.2   4 65000           3119      3181    0    0 02:13:01 Estab   5      5
  10.1.0.3   4 65000           1469      1506    0    0 01:02:28 Estab   9      9
  10.1.0.4   4 65000           3121      3180    0    0 02:12:59 Estab   7      7
  10.1.0.254 4 65000           2816      2856    0    0 01:59:50 Estab   8      8
```
## Проверка информации о маршрутах на spine2
```
dc1-spine2#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/20] via 10.2.2.253, Ethernet7
 O        10.0.0.1/32 [110/20] via 10.2.2.1, Ethernet1
 O        10.0.0.2/32 [110/20] via 10.2.2.3, Ethernet2
 O        10.0.0.3/32 [110/20] via 10.2.2.5, Ethernet3
 O        10.0.0.4/32 [110/20] via 10.2.2.7, Ethernet4
 O        10.0.0.254/32 [110/20] via 10.2.2.253, Ethernet7
 O        10.0.1.0/32 [110/30] via 10.2.2.1, Ethernet1
                               via 10.2.2.3, Ethernet2
                               via 10.2.2.5, Ethernet3
                               via 10.2.2.7, Ethernet4
                               via 10.2.2.253, Ethernet7
 C        10.0.2.0/32 is directly connected, Loopback1
 O        10.1.0.1/32 [110/20] via 10.2.2.1, Ethernet1
 O        10.1.0.2/32 [110/20] via 10.2.2.3, Ethernet2
 O        10.1.0.3/32 [110/20] via 10.2.2.5, Ethernet3
 O        10.1.0.4/32 [110/20] via 10.2.2.7, Ethernet4
 O        10.1.0.254/32 [110/20] via 10.2.2.253, Ethernet7
 O        10.1.1.0/32 [110/30] via 10.2.2.1, Ethernet1
                               via 10.2.2.3, Ethernet2
                               via 10.2.2.5, Ethernet3
                               via 10.2.2.7, Ethernet4
                               via 10.2.2.253, Ethernet7
 C        10.1.2.0/32 is directly connected, Loopback2
 O        10.2.1.0/31 [110/20] via 10.2.2.1, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.2.3, Ethernet2
 O        10.2.1.4/31 [110/20] via 10.2.2.5, Ethernet3
 O        10.2.1.6/31 [110/20] via 10.2.2.7, Ethernet4
 O        10.2.1.252/31 [110/20] via 10.2.2.253, Ethernet7
 O        10.2.1.254/31 [110/30] via 10.2.2.1, Ethernet1
                                 via 10.2.2.3, Ethernet2
                                 via 10.2.2.5, Ethernet3
                                 via 10.2.2.7, Ethernet4
                                 via 10.2.2.253, Ethernet7
 C        10.2.2.0/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet3
 C        10.2.2.6/31 is directly connected, Ethernet4
 C        10.2.2.252/31 is directly connected, Ethernet7
 C        10.2.2.254/31 is directly connected, Ethernet8

dc1-spine2#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.2.0, local AS number 65000
Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
---------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.0.1         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.1         65000 Established   L2VPN EVPN              Negotiated              8          8
10.1.0.2         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.2         65000 Established   L2VPN EVPN              Negotiated              5          5
10.1.0.3         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.3         65000 Established   L2VPN EVPN              Negotiated              9          9
10.1.0.4         65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.4         65000 Established   L2VPN EVPN              Negotiated              7          7
10.1.0.254       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.0.254       65000 Established   L2VPN EVPN              Negotiated              8          8

dc1-spine2#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.2.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.0.1   4 65000           1562      1606    0    0 01:06:22 Estab   8      8
  10.1.0.2   4 65000           1573      1632    0    0 01:07:07 Estab   5      5
  10.1.0.3   4 65000           1572      1586    0    0 01:06:04 Estab   9      9
  10.1.0.4   4 65000           1578      1632    0    0 01:07:10 Estab   5      5
  10.1.0.254 4 65000           1578      1628    0    0 01:07:05 Estab   7      7
```


## Проверка информации о маршрутах на leaf1
```
dc1-leaf1#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/30] via 10.2.1.0, Ethernet1
                              via 10.2.2.0, Ethernet2
 C        10.0.0.1/32 is directly connected, Loopback1
 O        10.0.0.2/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.0.0.3/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.0.0.4/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.0.0.254/32 [110/30] via 10.2.1.0, Ethernet1
                                 via 10.2.2.0, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.0, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.0, Ethernet2
 C        10.1.0.1/32 is directly connected, Loopback2
 O        10.1.0.2/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.1.0.3/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.1.0.4/32 [110/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 O        10.1.0.254/32 [110/30] via 10.2.1.0, Ethernet1
                                 via 10.2.2.0, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.0, Ethernet1
 O        10.1.2.0/32 [110/20] via 10.2.2.0, Ethernet2
 C        10.2.1.0/31 is directly connected, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.1.6/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.1.252/31 [110/20] via 10.2.1.0, Ethernet1
 O        10.2.1.254/31 [110/20] via 10.2.1.0, Ethernet1
 C        10.2.2.0/31 is directly connected, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.2.6/31 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.2.252/31 [110/20] via 10.2.2.0, Ethernet2
 O        10.2.2.254/31 [110/20] via 10.2.2.0, Ethernet2

dc1-leaf1#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated             26         26
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated             26         26

dc1-leaf1#sh bg evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           1642      1601    0    0 01:08:03 Estab   26     26
  10.1.2.0 4 65000           1657      1606    0    0 01:07:45 Estab   26     26

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

 B I      172.18.0.0/29 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.11.0/24 [200/0] via VTEP 10.1.0.4 VNI 10001 router-mac 50:00:00:1b:5e:8d local-interface Vxlan1
                                  via VTEP 10.1.0.2 VNI 10001 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 B I      192.168.110.4/32 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.111.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

dc1-leaf1#show ip route vrf tenant2

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

 B I      172.18.0.8/29 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.11.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        192.168.110.0/24 is directly connected, Vlan110
 B I      192.168.111.0/24 [200/0] via VTEP 10.1.0.4 VNI 10002 router-mac 50:00:00:1b:5e:8d local-interface Vxlan1
                                   via VTEP 10.1.0.2 VNI 10002 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
dc1-leaf1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.2:11 mac-ip 0050.7966.680d
                                 10.1.0.2              -       100     0       i Or-ID: 10.1.0.2 C-LST: 10.1.1.0
 *  ec    RD: 10.1.0.2:11 mac-ip 0050.7966.680d
                                 10.1.0.2              -       100     0       i Or-ID: 10.1.0.2 C-LST: 10.1.2.0
 * >Ec    RD: 10.1.0.4:11 mac-ip 0050.7966.680f
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.1.0
 *  ec    RD: 10.1.0.4:11 mac-ip 0050.7966.680f
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.2.0
 * >Ec    RD: 10.1.0.3:110 mac-ip 5000.0068.a17f
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.2.0
 *  ec    RD: 10.1.0.3:110 mac-ip 5000.0068.a17f
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.1.0
 * >      RD: 10.1.0.1:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.3:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.2.0
 *  ec    RD: 10.1.0.3:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.1.0
dc1-leaf1#sh bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.3:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.2.0
 *  ec    RD: 10.1.0.3:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.3
                                 10.1.0.3              -       100     0       i Or-ID: 10.1.0.3 C-LST: 10.1.1.0


```

## Проверка информации о маршрутах на leaf2
```
dc1-leaf2#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/30] via 10.2.1.2, Ethernet1
                              via 10.2.2.2, Ethernet2
 O        10.0.0.1/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 C        10.0.0.2/32 is directly connected, Loopback1
 O        10.0.0.3/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.0.0.4/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.0.0.254/32 [110/30] via 10.2.1.2, Ethernet1
                                 via 10.2.2.2, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.2, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.2, Ethernet2
 O        10.1.0.1/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 C        10.1.0.2/32 is directly connected, Loopback2
 O        10.1.0.3/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.1.0.4/32 [110/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 O        10.1.0.254/32 [110/30] via 10.2.1.2, Ethernet1
                                 via 10.2.2.2, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.2, Ethernet1
 O        10.1.2.0/32 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.2, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.1.6/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.1.252/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.1.254/31 [110/20] via 10.2.1.2, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.2, Ethernet2
 C        10.2.2.2/31 is directly connected, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.2.6/31 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.2.252/31 [110/20] via 10.2.2.2, Ethernet2
 O        10.2.2.254/31 [110/20] via 10.2.2.2, Ethernet2

dc1-leaf2#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI                                                                                                                                                                                                           Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ------                                                                                                                                                                                                          ---- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated                                                                                                                                                                                                                        0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated                                                                                                                                                                                                                       29         29
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated                                                                                                                                                                                                                        0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated                                                                                                                                                                                                                       29         29
dc1-leaf2#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated             29         29
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated             29         29
dc1-leaf2#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.2, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           3356      3284    0    0 02:20:11 Estab   29     29
  10.1.2.0 4 65000           1713      1657    0    0 01:10:39 Estab   29     29

dc1-leaf2#sh ip route vrf tenant1

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

 B I      172.18.0.0/29 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 10.1.0.1 VNI 10001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                  via VTEP 10.1.0.3 VNI 10001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        192.168.11.0/24 is directly connected, Vlan11
 B I      192.168.110.4/32 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.111.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

dc1-leaf2#sh ip route vrf tenant2

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

 B I      172.18.0.8/29 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.11.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.4/32 [200/0] via VTEP 10.1.0.1 VNI 10002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                   via VTEP 10.1.0.3 VNI 10002 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 10.1.0.1 VNI 10002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                   via VTEP 10.1.0.3 VNI 10002 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        192.168.111.0/24 is directly connected, Vlan111

```
## Проверка информации о маршрутах на leaf3
```
dc1-leaf3#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/30] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 O        10.0.0.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.0.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 C        10.0.0.3/32 is directly connected, Loopback1
 O        10.0.0.4/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.0.0.254/32 [110/30] via 10.2.1.4, Ethernet1
                                 via 10.2.2.4, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.1.0.1/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.1.0.2/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 C        10.1.0.3/32 is directly connected, Loopback2
 O        10.1.0.4/32 [110/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 O        10.1.0.254/32 [110/30] via 10.2.1.4, Ethernet1
                                 via 10.2.2.4, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.4, Ethernet1
 O        10.1.2.0/32 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.4, Ethernet1
 C        10.2.1.4/31 is directly connected, Ethernet1
 O        10.2.1.6/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.252/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.1.254/31 [110/20] via 10.2.1.4, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.4, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet2
 O        10.2.2.6/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.252/31 [110/20] via 10.2.2.4, Ethernet2
 O        10.2.2.254/31 [110/20] via 10.2.2.4, Ethernet2

dc1-leaf3#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated             25         25
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated             25         25
dc1-leaf3#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           1734      1695    0    0 01:11:55 Estab   25     25
  10.1.2.0 4 65000           1726      1710    0    0 01:11:55 Estab   25     25
dc1-leaf3#sh ip route vrf tenant1

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

 B I      172.18.0.0/29 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.11.0/24 [200/0] via VTEP 10.1.0.4 VNI 10001 router-mac 50:00:00:1b:5e:8d local-interface Vxlan1
                                  via VTEP 10.1.0.2 VNI 10001 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
 B I      192.168.110.4/32 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.111.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

dc1-leaf3#sh ip route vrf tenant2

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

 B I      172.18.0.8/29 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.11.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 C        192.168.110.0/24 is directly connected, Vlan110
 B I      192.168.111.0/24 [200/0] via VTEP 10.1.0.2 VNI 10002 router-mac 50:00:00:72:8b:31 local-interface Vxlan1
                                   via VTEP 10.1.0.4 VNI 10002 router-mac 50:00:00:1b:5e:8d local-interface Vxlan1

dc1-leaf3#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.2:11 mac-ip 0050.7966.680d
                                 10.1.0.2              -       100     0       i Or-ID: 10.1.0.2 C-LST: 10.1.1.0
 *  ec    RD: 10.1.0.2:11 mac-ip 0050.7966.680d
                                 10.1.0.2              -       100     0       i Or-ID: 10.1.0.2 C-LST: 10.1.2.0
 * >Ec    RD: 10.1.0.4:11 mac-ip 0050.7966.680f
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.1.0
 *  ec    RD: 10.1.0.4:11 mac-ip 0050.7966.680f
                                 10.1.0.4              -       100     0       i Or-ID: 10.1.0.4 C-LST: 10.1.2.0
 * >      RD: 10.1.0.3:110 mac-ip 5000.0068.a17f
                                 -                     -       -       0       i
 * >Ec    RD: 10.1.0.1:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 10.1.0.1              -       100     0       i Or-ID: 10.1.0.1 C-LST: 10.1.1.0
 *  ec    RD: 10.1.0.1:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 10.1.0.1              -       100     0       i Or-ID: 10.1.0.1 C-LST: 10.1.2.0
 * >      RD: 10.1.0.3:110 mac-ip 5000.0068.a17f 192.168.110.4
                                 -                     -       -       0       i
dc1-leaf3#sh bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.1.0.3, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.1.0.1 C-LST: 10.1.2.0
 *  ec    RD: 10.1.0.1:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.1
                                 10.1.0.1              -       100     0       i Or-ID: 10.1.0.1 C-LST: 10.1.1.0
 * >      RD: 10.1.0.3:1 ethernet-segment 0000:2222:3333:4444:0000 10.1.0.3
                                 -                     -       -       0       i

```

## Проверка информации о маршрутах на leaf4
```
dc-leaf4#sh ip route

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

Gateway of last resort is not set

 O        1.1.1.1/32 [110/30] via 10.2.1.6, Ethernet1
                              via 10.2.2.6, Ethernet2
 O        10.0.0.1/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 O        10.0.0.2/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 O        10.0.0.3/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 C        10.0.0.4/32 is directly connected, Loopback1
 O        10.0.0.254/32 [110/30] via 10.2.1.6, Ethernet1
                                 via 10.2.2.6, Ethernet2
 O        10.0.1.0/32 [110/20] via 10.2.1.6, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.6, Ethernet2
 O        10.1.0.1/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 O        10.1.0.2/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 O        10.1.0.3/32 [110/30] via 10.2.1.6, Ethernet1
                               via 10.2.2.6, Ethernet2
 C        10.1.0.4/32 is directly connected, Loopback2
 O        10.1.0.254/32 [110/30] via 10.2.1.6, Ethernet1
                                 via 10.2.2.6, Ethernet2
 O        10.1.1.0/32 [110/20] via 10.2.1.6, Ethernet1
 O        10.1.2.0/32 [110/20] via 10.2.2.6, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.6, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.6, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.6, Ethernet1
 C        10.2.1.6/31 is directly connected, Ethernet1
 O        10.2.1.252/31 [110/20] via 10.2.1.6, Ethernet1
 O        10.2.1.254/31 [110/20] via 10.2.1.6, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.6, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.6, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.6, Ethernet2
 C        10.2.2.6/31 is directly connected, Ethernet2
 O        10.2.2.252/31 [110/20] via 10.2.2.6, Ethernet2
 O        10.2.2.254/31 [110/20] via 10.2.2.6, Ethernet2

dc-leaf4#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated             29         29
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated             29         29
dc-leaf4#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.4, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           3438      3371    0    0 02:23:40 Estab   29     29
  10.1.2.0 4 65000           1801      1743    0    0 01:14:15 Estab   29     29
dc-leaf4#sh ip route vrf tenant1

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

 B I      172.18.0.0/29 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 10.1.0.1 VNI 10001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                  via VTEP 10.1.0.3 VNI 10001 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        192.168.11.0/24 is directly connected, Vlan11
 B I      192.168.110.4/32 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.111.0/24 [200/0] via VTEP 1.1.1.1 VNI 10001 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1

dc-leaf4#sh ip route vrf tenant2

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

 B I      172.18.0.8/29 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.10.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.11.0/24 [200/0] via VTEP 1.1.1.1 VNI 10002 router-mac 50:00:00:d7:ee:0b local-interface Vxlan1
 B I      192.168.110.4/32 [200/0] via VTEP 10.1.0.1 VNI 10002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                   via VTEP 10.1.0.3 VNI 10002 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 B I      192.168.110.0/24 [200/0] via VTEP 10.1.0.1 VNI 10002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
                                   via VTEP 10.1.0.3 VNI 10002 router-mac 50:00:00:f6:ad:37 local-interface Vxlan1
 C        192.168.111.0/24 is directly connected, Vlan111
```

## Проверка информации о маршрутах на leaf-border1
```
dc-leaf-border1#sh ip route

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

Gateway of last resort is not set

 C        1.1.1.1/32 is directly connected, Loopback0
 O        10.0.0.1/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.0.0.2/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.0.0.3/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.0.0.4/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 C        10.0.0.254/32 is directly connected, Loopback1
 O        10.0.1.0/32 [110/20] via 10.2.1.252, Ethernet1
 O        10.0.2.0/32 [110/20] via 10.2.2.252, Ethernet2
 O        10.1.0.1/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.1.0.2/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.1.0.3/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 O        10.1.0.4/32 [110/30] via 10.2.1.252, Ethernet1
                               via 10.2.2.252, Ethernet2
 C        10.1.0.254/32 is directly connected, Loopback2
 O        10.1.1.0/32 [110/20] via 10.2.1.252, Ethernet1
 O        10.1.2.0/32 [110/20] via 10.2.2.252, Ethernet2
 O        10.2.1.0/31 [110/20] via 10.2.1.252, Ethernet1
 O        10.2.1.2/31 [110/20] via 10.2.1.252, Ethernet1
 O        10.2.1.4/31 [110/20] via 10.2.1.252, Ethernet1
 O        10.2.1.6/31 [110/20] via 10.2.1.252, Ethernet1
 C        10.2.1.252/31 is directly connected, Ethernet1
 O        10.2.1.254/31 [110/20] via 10.2.1.252, Ethernet1
 O        10.2.2.0/31 [110/20] via 10.2.2.252, Ethernet2
 O        10.2.2.2/31 [110/20] via 10.2.2.252, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.2.252, Ethernet2
 O        10.2.2.6/31 [110/20] via 10.2.2.252, Ethernet2
 C        10.2.2.252/31 is directly connected, Ethernet2
 O        10.2.2.254/31 [110/20] via 10.2.2.252, Ethernet2
 C        172.17.0.0/30 is directly connected, Vlan4094

dc-leaf-border1#sh bgp summary
BGP summary information for VRF default
Router identifier 10.1.0.254, local AS number 65000
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.1.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.1.0       65000 Established   L2VPN EVPN              Negotiated             27         27
10.1.2.0       65000 Established   IPv4 Unicast            Negotiated              0          0
10.1.2.0       65000 Established   L2VPN EVPN              Negotiated             27         27
dc-leaf-border1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.254, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           3158      3110    0    0 02:12:21 Estab   27     27
  10.1.2.0 4 65000           1839      1785    0    0 01:15:59 Estab   27     27

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

 C        172.18.0.0/29 is directly connected, Vlan3900
 B I      192.168.10.0/24 is directly connected, Null0
 B I      192.168.11.0/24 is directly connected, Null0
 B E      192.168.110.4/32 [200/0] via 172.18.0.3, Vlan3900
 B E      192.168.110.0/24 [200/0] via 172.18.0.3, Vlan3900
 B E      192.168.111.0/24 [200/0] via 172.18.0.3, Vlan3900

dc-leaf-border1#show ip route vrf tenant2

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

 C        172.18.0.8/29 is directly connected, Vlan3901
 B E      192.168.10.0/24 [200/0] via 172.18.0.14, Vlan3901
 B E      192.168.11.0/24 [200/0] via 172.18.0.14, Vlan3901
 B I      192.168.110.4/32 is directly connected, Null0
 B I      192.168.110.0/24 is directly connected, Null0
 B I      192.168.111.0/24 is directly connected, Null0
```
## Проверка информации о маршрутах на fw1
```
dc-fw1(config)#sh ip bgp
BGP routing table information for VRF default
Router identifier 172.16.0.253, local AS number 65535
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * #     172.18.0.0/29          172.18.0.4            0       100     0       65000 i
 * #     172.18.0.8/29          172.18.0.9            0       100     0       65000 i
 * >     192.168.10.0/24        172.18.0.4            0       100     0       65000 i
 * >     192.168.11.0/24        172.18.0.4            0       100     0       65000 i
 * >     192.168.110.0/24       172.18.0.9            0       100     0       65000 i
 * >     192.168.110.4/32       172.18.0.9            0       100     0       65000 i
 * >     192.168.111.0/24       172.18.0.9            0       100     0       65000 i

dc-fw1#sh ip route

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

Gateway of last resort is not set

 C        172.17.0.0/30 is directly connected, Vlan4094
 C        172.18.0.0/29 is directly connected, Vlan3900
 C        172.18.0.8/29 is directly connected, Vlan3901
 B E      192.168.10.0/24 [200/0] via 172.18.0.4, Vlan3900
 B E      192.168.11.0/24 [200/0] via 172.18.0.4, Vlan3900
 B E      192.168.110.4/32 [200/0] via 172.18.0.9, Vlan3901
 B E      192.168.110.0/24 [200/0] via 172.18.0.9, Vlan3901
 B E      192.168.111.0/24 [200/0] via 172.18.0.9, Vlan3901
```

## Проверка l2 VNI связанности ten1-cli2 и ten1-cli4
```
ten1-cli2> ping 192.168.11.4

84 bytes from 192.168.11.4 icmp_seq=1 ttl=64 time=390.763 ms
84 bytes from 192.168.11.4 icmp_seq=2 ttl=64 time=485.903 ms
84 bytes from 192.168.11.4 icmp_seq=3 ttl=64 time=232.376 ms
84 bytes from 192.168.11.4 icmp_seq=4 ttl=64 time=434.410 ms
84 bytes from 192.168.11.4 icmp_seq=5 ttl=64 time=309.420 ms
```
## Проверка l3 VNI связанности ten2-host и ten2-cli2
```
ten2-host#ping 192.168.111.2
PING 192.168.111.2 (192.168.111.2) 72(100) bytes of data.
80 bytes from 192.168.111.2: icmp_seq=1 ttl=62 time=1244 ms
80 bytes from 192.168.111.2: icmp_seq=2 ttl=62 time=1254 ms
80 bytes from 192.168.111.2: icmp_seq=3 ttl=62 time=1488 ms
80 bytes from 192.168.111.2: icmp_seq=4 ttl=62 time=1528 ms
80 bytes from 192.168.111.2: icmp_seq=5 ttl=62 time=1621 ms
```
#### Проверку VRF Tenan1 до Tenant2 (leakin) через FW1 проверить не удалось, в такой схеме(вся необходимая маршрутная информация присутсвует, но при этом узел все равно недоступен, предположительно проблемы в производительнсоти стенда, т.к. на маленькой схеме, все работало корректно).


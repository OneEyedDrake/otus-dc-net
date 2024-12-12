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
## Вывод команд show ip bgp, show ip route и show ip bgp neighbor dc1-spine1

```
dc1-spine1#show ip bgp
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.0.0.1/32            10.2.1.1              0       100     0       i
 * >     10.0.0.2/32            10.2.1.3              0       100     0       i
 * >     10.0.0.3/32            10.2.1.5              0       100     0       i
 * >     10.0.1.0/32            -                     0       0       -       i
 * >     10.4.1.0/24            10.2.1.1              0       100     0       i
 * >     10.4.2.0/24            10.2.1.3              0       100     0       i
 * >     10.4.3.0/24            10.2.1.5              0       100     0       i

dc1-spine1#show ip route

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

 B I      10.0.0.1/32 [200/0] via 10.2.1.1, Ethernet1
 B I      10.0.0.2/32 [200/0] via 10.2.1.3, Ethernet2
 B I      10.0.0.3/32 [200/0] via 10.2.1.5, Ethernet3
 C        10.0.1.0/32 is directly connected, Loopback1
 C        10.1.1.0/32 is directly connected, Loopback2
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3
 B I      10.4.1.0/24 [200/0] via 10.2.1.1, Ethernet1
 B I      10.4.2.0/24 [200/0] via 10.2.1.3, Ethernet2
 B I      10.4.3.0/24 [200/0] via 10.2.1.5, Ethernet3


dc1-spine1#show ip bgp neighbors
BGP neighbor is 10.2.1.1, remote AS 65000, internal link
  BGP version 4, remote router ID 10.0.0.1, VRF default
  Inherits configuration from and member of peer-group dc1
  Negotiated BGP version 4
  Member of update group 2
  Last read 00:00:02, last write 00:00:03
  Hold time is 9, keepalive interval is 3 seconds
  Configured hold time is 9, keepalive interval is 3 seconds
  Connect timer is inactive
  Idle-restart timer is inactive
  BGP state is Established, up for 01:32:50
  Number of transitions to established: 1
  Last state was OpenConfirm
  Last event was RecvKeepAlive
  Neighbor is a route reflector client
  Neighbor Capabilities:
    Multiprotocol IPv4 Unicast: advertised and received and negotiated
    Four Octet ASN: advertised and received and negotiated
    Route Refresh: advertised and received and negotiated
    Send End-of-RIB messages: advertised and received and negotiated
    Additional-paths recv capability:
      IPv4 Unicast: advertised
    Additional-paths send capability:
      IPv4 Unicast: received
  Restart timer is inactive
  End of rib timer is inactive
  Message Statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent      Rcvd
    Opens:                  1         1
    Notifications:          0         0
    Updates:                4         2
    Keepalives:          1858      1858
    Route-Refresh:          0         0
    Total messages:      1863      1861
  Prefix Statistics:
                         Sent      Rcvd     Best Paths     Best ECMP Paths
    IPv4 Unicast:           5         2              2                   0
    IPv6 Unicast:           0         0              0                   0
    IPv4 SR-TE:             0         0              0                   0
    IPv6 SR-TE:             0         0              0                   0
  Inbound updates dropped by reason:
    AS path loop detection: 0
    Enforced First AS: 0
    Originator ID matches local router ID: 0
    Nexthop matches local IP address: 0
    Unexpected IPv6 nexthop for IPv4 routes: 0
  Inbound updates with attribute errors:
    Resulting in removal of all paths in update (treat-as-withdraw): 0
    Resulting in AFI/SAFI disable: 0
    Resulting in attribute ignore: 0
  Inbound paths dropped by reason:
    IPv4 labeled-unicast NLRIs dropped due to excessive labels: 0
    IPv6 labeled-unicast NLRIs dropped due to excessive labels: 0
  Outbound paths dropped by reason:
    IPv4 local address not available: 0
    IPv6 local address not available: 0
    Inbound policy
    Outbound policy
Local AS is 65000, local router ID 10.0.1.0
TTL is 255, BGP neighbor may be up to None hops away
Local TCP address is 10.2.1.0, local port is 179
Remote TCP address is 10.2.1.1, remote port is 41135
Auto-Local-Addr is disabled
MD5 authentication is enabled
Private AS numbers removed from outbound updates to this neighbor if only private AS numbers are present
TCP Socket Information:
  TCP state is ESTABLISHED
  Recv-Q: 0/32768
  Send-Q: 0/32768
  Outgoing Maximum Segment Size (MSS): 1440
  Total Number of TCP retransmissions: 0
  Options:
    Timestamps enabled: yes
    Selective Acknowledgments enabled: yes
    Window Scale enabled: yes
    Explicit Congestion Notification (ECN) enabled: no
  Socket Statistics:
    Window Scale (wscale): 7,7
    Retransmission Timeout (rto): 244.0ms
    Round-trip Time (rtt/rtvar): 41.1ms/2.4ms
    Delayed Ack Timeout (ato): 40.0ms
    Congestion Window (cwnd): 10
    TCP Throughput: 2.80 Mbps
    Advertised Recv Window (rcv_space): 14600


BGP neighbor is 10.2.1.3, remote AS 65000, internal link
  BGP version 4, remote router ID 10.0.0.2, VRF default
  Inherits configuration from and member of peer-group dc1
  Negotiated BGP version 4
  Member of update group 2
  Last read never, last write never
  Hold time is 9, keepalive interval is 3 seconds
  Configured hold time is 9, keepalive interval is 3 seconds
  Connect timer is inactive
  Idle-restart timer is inactive
  BGP state is Established, up for    2d01h
  Number of transitions to established: 1
  Last state was OpenConfirm
  Last event was RecvKeepAlive
  Neighbor is a route reflector client
  Neighbor Capabilities:
    Multiprotocol IPv4 Unicast: advertised and received and negotiated
    Four Octet ASN: advertised and received and negotiated
    Route Refresh: advertised and received and negotiated
    Send End-of-RIB messages: advertised and received and negotiated
    Additional-paths recv capability:
      IPv4 Unicast: advertised
    Additional-paths send capability:
      IPv4 Unicast: received
  Restart timer is inactive
  End of rib timer is inactive
  Message Statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent      Rcvd
    Opens:                  1         1
    Notifications:          0         0
    Updates:                6         2
    Keepalives:         59020     59019
    Route-Refresh:          0         0
    Total messages:     59027     59022
  Prefix Statistics:
                         Sent      Rcvd     Best Paths     Best ECMP Paths
    IPv4 Unicast:           5         2              2                   0
    IPv6 Unicast:           0         0              0                   0
    IPv4 SR-TE:             0         0              0                   0
    IPv6 SR-TE:             0         0              0                   0
  Inbound updates dropped by reason:
    AS path loop detection: 0
    Enforced First AS: 0
    Originator ID matches local router ID: 0
    Nexthop matches local IP address: 0
    Unexpected IPv6 nexthop for IPv4 routes: 0
  Inbound updates with attribute errors:
    Resulting in removal of all paths in update (treat-as-withdraw): 0
    Resulting in AFI/SAFI disable: 0
    Resulting in attribute ignore: 0
  Inbound paths dropped by reason:
    IPv4 labeled-unicast NLRIs dropped due to excessive labels: 0
    IPv6 labeled-unicast NLRIs dropped due to excessive labels: 0
  Outbound paths dropped by reason:
    IPv4 local address not available: 0
    IPv6 local address not available: 0
    Inbound policy
    Outbound policy
Local AS is 65000, local router ID 10.0.1.0
TTL is 255, BGP neighbor may be up to None hops away
Local TCP address is 10.2.1.2, local port is 179
Remote TCP address is 10.2.1.3, remote port is 42485
Auto-Local-Addr is disabled
MD5 authentication is enabled
Private AS numbers removed from outbound updates to this neighbor if only private AS numbers are present
TCP Socket Information:
  TCP state is ESTABLISHED
  Recv-Q: 0/32768
  Send-Q: 0/32768
  Outgoing Maximum Segment Size (MSS): 1440
  Total Number of TCP retransmissions: 0
  Options:
    Timestamps enabled: yes
    Selective Acknowledgments enabled: yes
    Window Scale enabled: yes
    Explicit Congestion Notification (ECN) enabled: no
  Socket Statistics:
    Window Scale (wscale): 7,7
    Retransmission Timeout (rto): 228.0ms
    Round-trip Time (rtt/rtvar): 25.1ms/16.5ms
    Delayed Ack Timeout (ato): 40.0ms
    Congestion Window (cwnd): 10
    TCP Throughput: 4.60 Mbps
    Recv Round-trip Time (rcv_rtt): 464338.6ms
    Advertised Recv Window (rcv_space): 64313


BGP neighbor is 10.2.1.5, remote AS 65000, internal link
  BGP version 4, remote router ID 10.0.0.3, VRF default
  Inherits configuration from and member of peer-group dc1
  Negotiated BGP version 4
  Member of update group 2
  Last read 00:00:01, last write 00:00:02
  Hold time is 9, keepalive interval is 3 seconds
  Configured hold time is 9, keepalive interval is 3 seconds
  Connect timer is inactive
  Idle-restart timer is inactive
  BGP state is Established, up for 01:37:43
  Number of transitions to established: 1
  Last state was OpenConfirm
  Last event was RecvKeepAlive
  Neighbor is a route reflector client
  Neighbor Capabilities:
    Multiprotocol IPv4 Unicast: advertised and received and negotiated
    Four Octet ASN: advertised and received and negotiated
    Route Refresh: advertised and received and negotiated
    Send End-of-RIB messages: advertised and received and negotiated
    Additional-paths recv capability:
      IPv4 Unicast: advertised
    Additional-paths send capability:
      IPv4 Unicast: received
  Restart timer is inactive
  End of rib timer is inactive
  Message Statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent      Rcvd
    Opens:                  1         1
    Notifications:          0         0
    Updates:                6         2
    Keepalives:          1956      1956
    Route-Refresh:          0         0
    Total messages:      1963      1959
  Prefix Statistics:
                         Sent      Rcvd     Best Paths     Best ECMP Paths
    IPv4 Unicast:           5         2              2                   0
    IPv6 Unicast:           0         0              0                   0
    IPv4 SR-TE:             0         0              0                   0
    IPv6 SR-TE:             0         0              0                   0
  Inbound updates dropped by reason:
    AS path loop detection: 0
    Enforced First AS: 0
    Originator ID matches local router ID: 0
    Nexthop matches local IP address: 0
    Unexpected IPv6 nexthop for IPv4 routes: 0
  Inbound updates with attribute errors:
    Resulting in removal of all paths in update (treat-as-withdraw): 0
    Resulting in AFI/SAFI disable: 0
    Resulting in attribute ignore: 0
  Inbound paths dropped by reason:
    IPv4 labeled-unicast NLRIs dropped due to excessive labels: 0
    IPv6 labeled-unicast NLRIs dropped due to excessive labels: 0
  Outbound paths dropped by reason:
    IPv4 local address not available: 0
    IPv6 local address not available: 0
    Inbound policy
    Outbound policy
Local AS is 65000, local router ID 10.0.1.0
TTL is 255, BGP neighbor may be up to None hops away
Local TCP address is 10.2.1.4, local port is 179
Remote TCP address is 10.2.1.5, remote port is 37645
Auto-Local-Addr is disabled
MD5 authentication is enabled
Private AS numbers removed from outbound updates to this neighbor if only private AS numbers are present
TCP Socket Information:
  TCP state is ESTABLISHED
  Recv-Q: 0/32768
  Send-Q: 0/32768
  Outgoing Maximum Segment Size (MSS): 1440
  Total Number of TCP retransmissions: 0
  Options:
    Timestamps enabled: yes
    Selective Acknowledgments enabled: yes
    Window Scale enabled: yes
    Explicit Congestion Notification (ECN) enabled: no
  Socket Statistics:
    Window Scale (wscale): 7,7
    Retransmission Timeout (rto): 228.0ms
    Round-trip Time (rtt/rtvar): 24.2ms/14.3ms
    Delayed Ack Timeout (ato): 40.0ms
    Congestion Window (cwnd): 10
    TCP Throughput: 4.77 Mbps
    Advertised Recv Window (rcv_space): 14600
```
## Вывод команд show ip bgp, show ip route и show ip bgp neighbor dc1-leaf3
```
dc1-leaf3#show ip bgp
BGP routing table information for VRF default
Router identifier 10.0.0.3, local AS number 65000
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >Ec   10.0.0.1/32            10.2.2.4              0       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 *  ec   10.0.0.1/32            10.2.1.4              0       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 * >Ec   10.0.0.2/32            10.2.1.4              0       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec   10.0.0.2/32            10.2.2.4              0       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 * >     10.0.0.3/32            -                     0       0       -       i
 * >     10.0.1.0/32            10.2.1.4              0       100     0       i
 * >     10.0.2.0/32            10.2.2.4              0       100     0       i
 * >Ec   10.4.1.0/24            10.2.2.4              0       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.2.0
 *  ec   10.4.1.0/24            10.2.1.4              0       100     0       i Or-ID: 10.0.0.1 C-LST: 10.0.1.0
 * >Ec   10.4.2.0/24            10.2.1.4              0       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.1.0
 *  ec   10.4.2.0/24            10.2.2.4              0       100     0       i Or-ID: 10.0.0.2 C-LST: 10.0.2.0
 * >     10.4.3.0/24            -                     1       0       -       i

dc1-leaf3#show ip route

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

 B I      10.0.0.1/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 B I      10.0.0.2/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 C        10.0.0.3/32 is directly connected, Loopback1
 B I      10.0.1.0/32 [200/0] via 10.2.1.4, Ethernet1
 B I      10.0.2.0/32 [200/0] via 10.2.2.4, Ethernet2
 C        10.1.0.3/32 is directly connected, Loopback2
 C        10.2.1.4/31 is directly connected, Ethernet1
 C        10.2.2.4/31 is directly connected, Ethernet2
 B I      10.4.1.0/24 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 B I      10.4.2.0/24 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 C        10.4.3.0/24 is directly connected, Vlan30


dc1-leaf3#show ip bgp neighbors
BGP neighbor is 10.2.1.4, remote AS 65000, internal link
  BGP version 4, remote router ID 10.0.1.0, VRF default
  Inherits configuration from and member of peer-group dc1
  Negotiated BGP version 4
  Member of update group 2
  Last read 00:00:01, last write 00:00:01
  Hold time is 9, keepalive interval is 3 seconds
  Configured hold time is 9, keepalive interval is 3 seconds
  Connect timer is inactive
  Idle-restart timer is inactive
  BGP state is Established, up for 01:43:52
  Number of transitions to established: 1
  Last state was OpenConfirm
  Last event was RecvKeepAlive
  Neighbor Capabilities:
    Multiprotocol IPv4 Unicast: advertised and received and negotiated
    Four Octet ASN: advertised and received and negotiated
    Route Refresh: advertised and received and negotiated
    Send End-of-RIB messages: advertised and received and negotiated
    Additional-paths recv capability:
      IPv4 Unicast: advertised
    Additional-paths send capability:
      IPv4 Unicast: received
  Restart timer is inactive
  End of rib timer is inactive
  Message Statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent      Rcvd
    Opens:                  1         1
    Notifications:          0         0
    Updates:                2         6
    Keepalives:          2079      2079
    Route-Refresh:          0         0
    Total messages:      2082      2086
  Prefix Statistics:
                         Sent      Rcvd     Best Paths     Best ECMP Paths
    IPv4 Unicast:           2         5              3                   4
    IPv6 Unicast:           0         0              0                   0
    IPv4 SR-TE:             0         0              0                   0
    IPv6 SR-TE:             0         0              0                   0
  Inbound updates dropped by reason:
    AS path loop detection: 0
    Enforced First AS: 0
    Originator ID matches local router ID: 0
    Nexthop matches local IP address: 0
    Unexpected IPv6 nexthop for IPv4 routes: 0
  Inbound updates with attribute errors:
    Resulting in removal of all paths in update (treat-as-withdraw): 0
    Resulting in AFI/SAFI disable: 0
    Resulting in attribute ignore: 0
  Inbound paths dropped by reason:
    IPv4 labeled-unicast NLRIs dropped due to excessive labels: 0
    IPv6 labeled-unicast NLRIs dropped due to excessive labels: 0
  Outbound paths dropped by reason:
    IPv4 local address not available: 0
    IPv6 local address not available: 0
    Inbound policy
    Outbound policy
Local AS is 65000, local router ID 10.0.0.3
TTL is 255
Local TCP address is 10.2.1.5, local port is 37645
Remote TCP address is 10.2.1.4, remote port is 179
Auto-Local-Addr is disabled
MD5 authentication is enabled
Private AS numbers removed from outbound updates to this neighbor if only private AS numbers are present
TCP Socket Information:
  TCP state is ESTABLISHED
  Recv-Q: 0/32768
  Send-Q: 0/32768
  Outgoing Maximum Segment Size (MSS): 1460
  Total Number of TCP retransmissions: 0
  Options:
    Timestamps enabled: yes
    Selective Acknowledgments enabled: yes
    Window Scale enabled: yes
    Explicit Congestion Notification (ECN) enabled: no
  Socket Statistics:
    Window Scale (wscale): 7,7
    Retransmission Timeout (rto): 212.0ms
    Round-trip Time (rtt/rtvar): 11.8ms/1.2ms
    Delayed Ack Timeout (ato): 40.0ms
    Congestion Window (cwnd): 10
    TCP Throughput: 9.94 Mbps
    Advertised Recv Window (rcv_space): 14600


BGP neighbor is 10.2.2.4, remote AS 65000, internal link
  BGP version 4, remote router ID 10.0.2.0, VRF default
  Inherits configuration from and member of peer-group dc1
  Negotiated BGP version 4
  Member of update group 2
  Last read 00:00:02, last write 00:00:02
  Hold time is 9, keepalive interval is 3 seconds
  Configured hold time is 9, keepalive interval is 3 seconds
  Connect timer is inactive
  Idle-restart timer is inactive
  BGP state is Established, up for 01:43:50
  Number of transitions to established: 1
  Last state was OpenConfirm
  Last event was RecvKeepAlive
  Neighbor Capabilities:
    Multiprotocol IPv4 Unicast: advertised and received and negotiated
    Four Octet ASN: advertised and received and negotiated
    Route Refresh: advertised and received and negotiated
    Send End-of-RIB messages: advertised and received and negotiated
    Additional-paths recv capability:
      IPv4 Unicast: advertised
    Additional-paths send capability:
      IPv4 Unicast: received
  Restart timer is inactive
  End of rib timer is inactive
  Message Statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent      Rcvd
    Opens:                  1         1
    Notifications:          0         0
    Updates:                2         4
    Keepalives:          2078      2078
    Route-Refresh:          0         0
    Total messages:      2081      2083
  Prefix Statistics:
                         Sent      Rcvd     Best Paths     Best ECMP Paths
    IPv4 Unicast:           2         5              3                   4
    IPv6 Unicast:           0         0              0                   0
    IPv4 SR-TE:             0         0              0                   0
    IPv6 SR-TE:             0         0              0                   0
  Inbound updates dropped by reason:
    AS path loop detection: 0
    Enforced First AS: 0
    Originator ID matches local router ID: 0
    Nexthop matches local IP address: 0
    Unexpected IPv6 nexthop for IPv4 routes: 0
  Inbound updates with attribute errors:
    Resulting in removal of all paths in update (treat-as-withdraw): 0
    Resulting in AFI/SAFI disable: 0
    Resulting in attribute ignore: 0
  Inbound paths dropped by reason:
    IPv4 labeled-unicast NLRIs dropped due to excessive labels: 0
    IPv6 labeled-unicast NLRIs dropped due to excessive labels: 0
  Outbound paths dropped by reason:
    IPv4 local address not available: 0
    IPv6 local address not available: 0
    Inbound policy
    Outbound policy
Local AS is 65000, local router ID 10.0.0.3
TTL is 255
Local TCP address is 10.2.2.5, local port is 45947
Remote TCP address is 10.2.2.4, remote port is 179
Auto-Local-Addr is disabled
MD5 authentication is enabled
Private AS numbers removed from outbound updates to this neighbor if only private AS numbers are present
TCP Socket Information:
  TCP state is ESTABLISHED
  Recv-Q: 0/32768
  Send-Q: 0/32768
  Outgoing Maximum Segment Size (MSS): 1460
  Total Number of TCP retransmissions: 0
  Options:
    Timestamps enabled: yes
    Selective Acknowledgments enabled: yes
    Window Scale enabled: yes
    Explicit Congestion Notification (ECN) enabled: no
  Socket Statistics:
    Window Scale (wscale): 7,7
    Retransmission Timeout (rto): 212.0ms
    Round-trip Time (rtt/rtvar): 11.4ms/2.2ms
    Delayed Ack Timeout (ato): 40.0ms
    Congestion Window (cwnd): 10
    TCP Throughput: 10.23 Mbps
    Advertised Recv Window (rcv_space): 14600
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

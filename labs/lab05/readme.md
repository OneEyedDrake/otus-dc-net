### **VxLAN. L3VNI**



Настроить маршрутизацию в рамках Overlay между клиентами

Для настройки VxLAN. L3 VNI будет использован стенд из 4-ой лабораторной работы, с добавлением дополнительной подсети и клиента рамещенного в нем.

### **План работы:**
1. Настройки будут произвоиться только на leaf, т.к. настройка идёт в Overlay сети;
2. На leaf3 создадим 70-й vlan и подключим в него нового клиента;
3. На всех leaf:
   
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
   
**Готово!** можно проверять доступность хостов host1 и host4

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
      redistribute connected
!

```
## Конфигурация dc1-leaf2


```

      

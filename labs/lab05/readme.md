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

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab05/eve-ng-scheme.png)


      

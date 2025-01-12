### **VXLAN. Multihoming**

Цель:
Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming

Для настройки VxLAN. L3 VNI будет использован стенд из 5-ой лабораторной работы, с добавлением дополнительного клиента подключено в leaf1 и leaf2 с использованием **lacp**.

### **План работы:**
1. Добавим дополнительное клиентское устройство и подключим к leaf1 и leaf2 портами 1 и 2 к 5-м портам(leaf) соответственно.
   
2. Настроим клиентское устройство **node1**:
   
   2.1 создадим vlan 100
   2.2 создадим svi 100 назначим адрес 10.4.100.21/24
   2.3 добавим машрут по умолчанию ip route 0.0.0.0/0 10.4.100.1
   2.4 добавим порты eth1-2 в портгруппу 1, **channel-group 1 mode active** с использованием протокола lacp
   2.5 настроим interface port-chanel1

3. Настроим leaf1 и 2(настройки будут идентичны)
   3.1 создадим interface port-channel1, сделаем его trunk и разрешим vlan 100.
   3.2 Сделаем настройки для работы **ESI LAG** *evpn ethernet-segment*:
      - добавим уникальный идентификатор для ESI командой: identifier 0000:2222:3333:4444:0000
      - добавим route-target import 55:55:55:55:55:55 для импорта маршрутов
      - зададим lacp system-id 7777.7777.7777 для корректной работы lacp.
   3.3 Добавим интерфейс eth5 в port-channel1 
   
**Готово!** можно проверять доступность хостов node1 и host4 и host3. 


**Таблица 1 Loopback интерфейсов**
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab06/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab06/ptp%20network.png)

**Таблица 3 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab06/host%20add.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab06/as%2065000.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab06/eve-ng-scheme.png)

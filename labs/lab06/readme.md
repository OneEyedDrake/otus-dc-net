### **VXLAN. Multihoming**

Цель:
Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming

Для настройки VxLAN. L3 VNI будет использован стенд из 5-ой лабораторной работы, с добавлением дополнительного клиента подключено в leaf1 и leaf2 с использованием **lacp**.

### **План работы:**

   
**Готово!** можно проверять доступность хостов host4 и host1

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

### **Проектирование адресного пространства**

1. Собрать схему CLOS;
2. Распределить адресное пространство.


![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/stand.jpg)

### **Первично подготовлена схема CLOS в draw.io, с наименованием устройств:**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/draw.io2.png)

В соотвествиями с рекомендациями полученными в лекции, были выделены адресные пространства:
1. **PTP** интерфесов для подключения **Spine** и **Leaf** маршрутизаторов;
2. Двух **Loopback** интерфейсов для каждого **Spine** и **Leaf** маршрутизатора;

Для удобства администрирования была использована сдедующая логика:

**IP = 10.Dn.Sn.X/31**, где:

**Dn** – диапазон в зависимости от номера ЦОД (в работе использован дипазон 10.0.0.0/13): 
- 0 - диапазон для **loopback 1**,
- 1 - диапазон для **loopback 2**,
- 2 - диапазон для **ptp**,
- 3 - диапазон разеревирован,
- 4-7 - диапазон под сервисы **(host and VM)**

Sn – номер spine, для looback на Leaf ипользуется **0**

X – значение по порядку, 

Ниже представлены таблицы адресации:

**Таблица 1 Loopback интерфейсов**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/addres%20loopback.png)


**Таблица 2 подсетей ptp интерфейсов**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/ptp%20network.png)

**Таблица 3 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/address%20ptp.png)

### **Итоговая схема с адресацией**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/scheme.png)
---

### **Реализация схемы в виртуальной лаборатории *EVE NG***
1. Добавлены *node* на схему, с ипользованием образа VEOS;
2. Переименованы *node* на схеме и внтури устройства;
3. Подключены **eth** интерфесы, каждый leaf подключен к каждому spine;
4. На интерфейсах ptp настроены ip адреса в соотвествии с *таблицей 3*;
5. На всех устройствах настроены loopback адреса в соотвествии с *таблицей 1*;
6. Провереена доступность устройств командой **ping**.

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/eve-ng-scheme.png)

## Пример конфигурации dc1-spine1
```
dc1-spine1#show running-config
! Command: show running-config
! device: dc1-spine1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname dc1-spine1
!
spanning-tree mode mstp
!
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
!
interface Loopback2
   ip address 10.1.1.0/32
!
interface Management1
!
no ip routing
!
end
dc1-spine1#
```
## Пример конфигурации dc1-leaf3

```
dc1-leaf3#show running-config
! Command: show running-config
! device: dc1-leaf3 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname dc1-leaf3
!
spanning-tree mode mstp
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
!
interface Loopback1
   ip address 10.0.0.3/32
!
interface Loopback2
   ip address 10.1.0.3/32
!
interface Management1
!
no ip routing
!
end
```

## Пример доступности с dc1-leaf3 узла dc1-spine1
```
dc1-leaf3#ping 10.2.1.4
PING 10.2.1.4 (10.2.1.4) 72(100) bytes of data.
80 bytes from 10.2.1.4: icmp_seq=1 ttl=64 time=47.8 ms
80 bytes from 10.2.1.4: icmp_seq=2 ttl=64 time=41.5 ms
80 bytes from 10.2.1.4: icmp_seq=3 ttl=64 time=37.8 ms
80 bytes from 10.2.1.4: icmp_seq=4 ttl=64 time=25.9 ms
80 bytes from 10.2.1.4: icmp_seq=5 ttl=64 time=9.75 ms

--- 10.2.1.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 9.754/32.572/47.853/13.455 ms, pipe 4, ipg/ewma 21.191/39.201 ms
```


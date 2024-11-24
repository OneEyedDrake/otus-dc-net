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

Sn – номер spine,

X – значение по порядку, 

Ниже представлены таблицы адресации:

**Таблица Loopback интерфейсов**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/addres%20loopback.png)


**Таблица подсетей ptp интерфейсов**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/ptp%20network.png)

**Таблица *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/address%20ptp.png)

### **Итоговая схема с адресацией**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab01/scheme.png)
---

### **Реализация схемы в виртуальной лаборатории *EVE NG***


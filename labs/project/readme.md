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

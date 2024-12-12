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
В IBGP чтоб соседи поместили маршрут в таблицу маршрутизации необходимо чтобы у получателя такого анонса был маршрут до Next-Hop, сделать это можно командой **neighbor dc1 next-hop-self**
Настроим автоматическое обнаружение соседсва в соотвествии с префиксом подсети и настроек *peer group dc1* **bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000**
Анансируем адрес Loopback1 в в таблицу маршрутизаци;
**Настрока spine завершена!**
На Leaf сделаем настройки peer group аналогичные **Spine**
```
neighbor dc1 peer group
   neighbor dc1 remote-as 65000
   neighbor dc1 timers 3 9
   neighbor dc1 password 7
```

 


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

## Конфигурация OSPF dc1-spine1

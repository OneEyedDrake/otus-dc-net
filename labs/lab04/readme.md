### **VxLAN. L2 VNI**

-Настроить VxLAN. L2 VNI

Для настройки VxLAN. L2 VNI будет использован стенд из 3-ой лабораторной работы, с iBGP в качестве Underlay сети.

### **План работы:**
1. Настроить **ptp** между spine и leaf в соответствии со адресацией и схемой;
2. Настроить **Loopback** интерфейсы **lo1** и **lo2** на spine и leaf (для использования в Underlay и Overlay сетях соответсвенно)
4. Создадим ip-префикc для анонса адресов **lo1-2** интефейсов *ip prefix-list loopback seq 10 permit 10.0.0.0/15 le 32* и route-map и приматчим **prefix-list loopback**(в дальнейшем будет использован для редистрибьюции loopback1-2);
5. Включим поддержку EVPN на Arista командой **service routing protocols model multi-agent**
6. Используем ранее созданный **BGP процесс 65000**, добавим настройки для Overlay сети
7. На Spine, воспользуемся **peer group**, для упрощения конфигруации (все настройки принадлежащие одной **peer group** будут применяться для установления соседсва; **neighbor *overlay* peer group**
9. Зададим номер AS из приватного дипазона  neighbor overlay **remote-as 65000**
10. Зададим таймеры *Keepalive Timer* и *Hold* neighbor overlay **timers 3 9**
11. Зададим пароль который будет использован для поднятия сессии с соседом neighbor overlay **password 7 arista-overlay**
12. Для передачи EVPN маршрутов между leaf включим функционал Route Reflector **neighbor overlay route-reflector-client**, все полученные маршруты будут переданы на все остальные маршрутизаторы в сехеме.
13. Настроим автоматическое обнаружение соседсва в соотвествии с префиксом подсети и настроек *peer group overlay* **bgp listen range 10.2.0.0/16 peer-group dc1 remote-as 65000**
14. Включим поддрежку расширенного комьюнити для передачи EVPN информации **neighbor overlay send-community extended**
15. В качестве интерфейса для BGP сессии в overlay используем lo2 **neighbor overlay update-source Loopback2**
16. Активируем возможноть передачи EVPN информации:
17. ```
18. address-family evpn
19.    neighbor overlay activate
20.```
21. Анонсируем адреса Loopback1-2 в в таблицу маршрутизаци **redistribute connected route-map redistrib_connect**;
22. **Настрока spine завершена!**

На Leaf:
1. Используем ранее созданный **BGP процесс 65000**, добавим настройки для Overlay сети
3. Cделаем настройки peer group *overlay* аналогичные **Spine**
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
  
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/addres%20loopback.png)

**Таблица 2 *IP адресов* ptp интерейсов leaf и spine**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/ptp%20network.png)

**Таблица 3 адреса хостовых машин**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/add%20hosts.png)

### **Cхема с адресацией и указанием AS BGP**

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/as%2065000.png)

### **Cхема eve-ng стенда**
![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/lab04/eve-ng-scheme.png)

# **Проектная работа**
## **Тема: "Построение сетевой фабрики на основе VxLAN EVPN с фильтрацией трафика на внешнем firewall"**

## **Цель:**
Реализовать разграничение на обмен информацией между сегментами сети, с возможнотью масштабирования и учетом миграции имеющегося оборудования и сервисов.  

# **Используемые технологии:**
**VxLAN, EVPN, BGP, MLAG, LACP, OSPF, ESI LAG**

### Cхема до перехода на VxLAN EVPN с Firewall 

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/Scheme%20do1.png)

### Cхема "промежуточная" до полного перехода на VxLAN EVPN с Firewall

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/Scheme%20after1.png)

### Cхема итоговая все сервисы ходят через фабрику VxLAN EVPN с фильтрацией трафика на Firewall

![](https://github.com/OneEyedDrake/otus-dc-net/blob/main/labs/project/scheme/scheme%20final1.png)

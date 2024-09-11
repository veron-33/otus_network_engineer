# EIGRP

## Цель:
- Настроить EIGRP в С.-Петербург;
- Использовать named EIGRP

## Описание/Пошаговая инструкция выполнения домашнего задания:
- В офисе С.-Петербург настроить EIGRP.
- R32 получает только маршрут по умолчанию.
- R16-17 анонсируют только суммарные префиксы.
- Использовать EIGRP named-mode для настройки сети.

## Выполнение

Настроим роутер R18.  
Создадим именованный EIGRP, добавим router-id и сеть:
```
R18(config)#router eigrp SPB
R18(config-router)#address-family ipv4 unicast autonomous-system 2042
R18(config-router-af)#eigrp router-id 18.18.18.18
R18(config-router-af)#network 10.2.0.0 255.255.0.0
```
Также дополнительно пропишем на этом роутере маршрут по-умолчанию и включим в EIGRP его распространение:
```
R18(config)#ip route 0.0.0.0 0.0.0.0 10.4.10.1
R18(config-router-af-topology)#redistribute static
```

Настроим R17, в том числе включим суммирование маршрутов:
```
R17(config)#router eigrp SPB
R17(config-router)#address-family ipv4 unicast autonomous-system 2042
R17(config-router-af)#eigrp router-id 17.17.17.17
R17(config-router-af)#network 10.2.0.0 255.255.0.0
R17(config-router-af)#topology base
R17(config-router-af-topology)#auto-summary
```

Настроим R16 аналогично как и R17, но дополнительно настроим фильтрацию для распространения на R32 только маршрута по-умолчанию:
```
R16(config)#router eigrp SPB
R16(config-router)#address-family ipv4 unicast autonomous-system 2042
R16(config-router-af)#eigrp router-id 16.16.16.16
R16(config-router-af)#network 10.2.0.0 255.255.0.0
R16(config-router-af)#topology base
R16(config-router-af-topology)#auto-summary

R16(config)#ip prefix-list only-def seq 10 permit 0.0.0.0/0
R16(config-router-af-topology)#distribute-list prefix only-def out Ethernet0/3
```

Настроим R32:
```
R32(config)#router eigrp SPB
R32(config-router)#address-family ipv4 unicast autonomous-system 2042
R32(config-router-af)#eigrp router-id 32.32.32.32
R32(config-router-af)#network 10.2.0.0 255.255.0.0
```

## Проверка

Получим EIGRP-соседей на роутрере R16
```
R16#sh ip ei ne
EIGRP-IPv4 VR(SPB) Address-Family Neighbors for AS(2042)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
3   10.2.5.2                Et0/3                    14 00:11:11    6   100  0  5
2   10.2.2.1                Et0/2                    10 00:20:29   12   100  0  23
1   10.2.3.2                Et0/0                    11 00:20:31    8   100  0  24
0   10.2.1.1                Et0/1                    12 00:24:45  526  3156  0  13
```

Таблица маршрутизации на нем же:
```
R16#sh ip ro
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.2.1.1 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/1536000] via 10.2.1.1, 00:01:45, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
D        10.2.0.0/30 [90/1536000] via 10.2.3.2, 00:01:45, Ethernet0/0
                     [90/1536000] via 10.2.2.1, 00:01:45, Ethernet0/2
                     [90/1536000] via 10.2.1.1, 00:01:45, Ethernet0/1
C        10.2.1.0/30 is directly connected, Ethernet0/1
L        10.2.1.2/32 is directly connected, Ethernet0/1
C        10.2.2.0/24 is directly connected, Ethernet0/2
L        10.2.2.2/32 is directly connected, Ethernet0/2
C        10.2.3.0/24 is directly connected, Ethernet0/0
L        10.2.3.1/32 is directly connected, Ethernet0/0
C        10.2.5.0/30 is directly connected, Ethernet0/3
L        10.2.5.1/32 is directly connected, Ethernet0/3
```


На роутере R32 проверим, что он получил только маршрут по-умолчанию:
```
R32#sh ip ro
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.2.5.1 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/2048000] via 10.2.5.1, 00:00:39, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.2.5.0/30 is directly connected, Ethernet0/0
L        10.2.5.2/32 is directly connected, Ethernet0/0
```
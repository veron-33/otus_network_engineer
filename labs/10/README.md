# iBGP

## Цель:
- Настроить iBGP в офисе Москва
- Настроить iBGP в сети провайдера Триада
- Организовать полную IP связанность всех сетей


## Описание/Пошаговая инструкция выполнения домашнего задания:


- Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.
- Настроите iBGP в провайдере Триада, с использованием RR.
- Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.
- Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
- Все сети в лабораторной работе должны иметь IP связность.

## Выполнение

### Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.

Добавим на роутерах друг друга в настройках BGP:
```
R14(config-router)#neighbor 10.1.0.6 remote-as 1001
R14(config-router)#neighbor 10.1.0.6 next-hop-self

R15(config-router)#neighbor 10.1.0.5 remote-as 1001
R15(config-router)#neighbor 10.1.0.5 next-hop-self
```

Выведем результат:
```
R15(config-router)#do sh ip bgp
BGP table version is 14, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.1.0.0/21      10.1.0.5                 0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 10.1.8.0/30      10.1.0.5                 0    100      0 101 i
 *                    10.1.9.2                               0 301 101 i
 r>  10.1.9.0/30      10.1.9.2                 0             0 301 i
 * i 10.1.10.0/30     10.1.0.5                 0    100      0 101 i
 *>                   10.1.9.2                 0             0 301 i
 *>  10.2.0.0/16      10.1.9.2                               0 301 520 2042 i
 *>i 10.4.4.0/30      10.1.0.5                 0    100      0 101 i
 *                    10.1.9.2                               0 301 101 i
 *>  10.4.5.0/30      10.1.9.2                 0             0 301 i
 *>  10.4.10.0/30     10.1.9.2                               0 301 520 i
```


### Настроите iBGP в провайдере Триада, с использованием RR.

В данном случае обмениваться маршрутной информацией будем от интерфейса Loopback, чтобы не потерять связности в случае отключения любого из физических интерфейсов. Также, т.к. этот интерфейс ранее не использовался включим на нем протокол ISIS, чтобы адреса loopback-интерфейсов в этой АС "видели" также друг друга.  
В качесвте RR выберем роутер R24.  

Настроим R23:
```
R23(config-if)#ip address 23.23.23.23 255.255.255.0
R23(config-if)#ip router isis 1
R23(config)#router bgp 520
R23(config-router)#bgp router-id 23.23.23.23
R23(config-router)#network 10.4.4.0 mask 255.255.255.252
R23(config-router)#network 10.4.3.0 mask 255.255.255.252
R23(config-router)#network 10.4.0.0 mask 255.255.255.252
R23(config-router)#neighbor 24.24.24.24 remote-as 520
R23(config-router)#neighbor 24.24.24.24 next-hop-self
R23(config-router)#neighbor 24.24.24.24 update-source loopback 0
R23(config-router)#neighbor 10.4.4.2 remote-as 101
```

Настроим R25:
```
R25(config-if)#ip address 25.25.25.25 255.255.255.0
R25(config-if)#ip router isis 1
R25(config)#router bgp 520
R25(config-router)#bgp router-id 25.25.25.25
R25(config-router)#network 10.4.0.0 mask 255.255.255.252
R25(config-router)#network 10.4.1.0 mask 255.255.255.252
R25(config-router)#neighbor 24.24.24.24 remote-as 520
R25(config-router)#neighbor 24.24.24.24 update-source loopback 0
R25(config-router)#neighbor 24.24.24.24 next-hop-self
```

Настроим R26:
```
R26(config-if)#ip address 26.26.26.26 255.255.255.0
R26(config-if)#ip router isis 1
R26(config)#router bgp 520
R26(config-router)#bgp router-id 26.26.26.26
R26(config-router)#network 10.4.9.0 mask 255.255.255.252
R26(config-router)#network 10.4.2.0 mask 255.255.255.252
R26(config-router)#network 10.4.1.0 mask 255.255.255.252
R26(config-router)#neighbor 24.24.24.24 remote-as 520
R26(config-router)#neighbor 24.24.24.24 update-source loopback 0
R26(config-router)#neighbor 24.24.24.24 next-hop-self
R26(config-router)#neighbor 10.4.9.2 remote-as 2042
```


Настроим R24:
```
R24(config-if)#ip address 24.24.24.24 255.255.255.0
R24(config-if)#ip router isis 1
R24(config)#router bgp 520
R24(config-router)#neighbor 23.23.23.23 remote-as 520
R24(config-router)#neighbor 23.23.23.23 update-source loopback 0
R24(config-router)#neighbor 23.23.23.23 route-reflector-client
R24(config-router)#neighbor 23.23.23.23 next-hop-self
R24(config-router)#neighbor 26.26.26.26 remote-as 520
R24(config-router)#neighbor 26.26.26.26 route-reflector-client
R24(config-router)#neighbor 26.26.26.26 update-source loopback 0
R24(config-router)#neighbor 26.26.26.26 next-hop-self
R24(config-router)#neighbor 25.25.25.25 remote-as 520
R24(config-router)#neighbor 25.25.25.25 route-reflector-client
R24(config-router)#neighbor 25.25.25.25 update-source loopback 0
R24(config-router)#neighbor 25.25.25.25 next-hop-self
```

Проверим на R25 обмен маршрутной информацией:
```
R25(config-router)#do sh ip bgp
BGP table version is 73, local router ID is 25.25.25.25
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.1.0.0/21      24.24.24.24              0    100      0 301 1001 i
 *>i 10.1.8.0/30      23.23.23.23              0    100      0 101 i
 *>i 10.1.9.0/30      24.24.24.24              0    100      0 301 i
 *>i 10.1.10.0/30     24.24.24.24              0    100      0 301 i
 *>i 10.2.0.0/16      24.24.24.24              0    100      0 2042 i
 * i 10.4.0.0/30      23.23.23.23              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 * i 10.4.1.0/30      26.26.26.26              0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 r>i 10.4.2.0/30      24.24.24.24              0    100      0 i
 r>i 10.4.3.0/30      24.24.24.24              0    100      0 i
 *>i 10.4.4.0/30      23.23.23.23              0    100      0 i
 *>i 10.4.5.0/30      24.24.24.24              0    100      0 i
 *>i 10.4.9.0/30      26.26.26.26              0    100      0 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.4.10.0/30     24.24.24.24              0    100      0 i
```





### Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.

На роутере R15 Ламас итак будет приоритетным провайдером. Для настройки приоритетного провайдера Ламас на роутере R14, повысим вес полученной от R15 маршрутной информации:
```
R14(config-router)#neighbor 10.1.0.6 weight 500
```

Проверим применимость приоритизации маршрутов:
```
R14#sh ip bgp
BGP table version is 14, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.1.0.0/21      0.0.0.0                  0         32768 i
 * i                  10.1.0.6                 0    100    500 i
 r   10.1.8.0/30      10.1.8.2                 0             0 101 i
 r>i                  10.1.0.6                 0    100    500 301 101 i
 *   10.1.9.0/30      10.1.8.2                               0 101 301 i
 *>i                  10.1.0.6                 0    100    500 301 i
 *   10.1.10.0/30     10.1.8.2                 0             0 101 i
 *>i                  10.1.0.6                 0    100    500 301 i
 *   10.2.0.0/16      10.1.8.2                               0 101 520 2042 i
 *>i                  10.1.0.6                 0    100    500 301 520 2042 i
 *   10.4.0.0/30      10.1.8.2                               0 101 520 i
 *>i                  10.1.0.6                 0    100    500 301 520 i
 *   10.4.1.0/30      10.1.8.2                               0 101 520 i
...
```
Теперь с помощью трассировки удостоверимся, что трафик пошел через Ламас (АС 301):
```
R14#traceroute 10.4.0.2
Type escape sequence to abort.
Tracing the route to 10.4.0.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.0.6 1 msec 0 msec 0 msec
  2 10.1.9.2 [AS 301] 0 msec 1 msec 0 msec
  3 10.4.5.1 [AS 301] 0 msec 1 msec 0 msec
  4 10.4.3.2 [AS 520] 0 msec 1 msec 0 msec
  5 10.4.0.2 [AS 520] 0 msec *  1 msec
```



### Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.

Сначала добавим второго соседа из АС 520 для обмена маршрутами BGP (на предыдущей лабы добавляли только одного):
```
R18(config-router)#neighbor 10.4.9.1 remote-as 520
```
Теперь для балансировки установим параметр:
```
R18(config-router)#maximum-paths 2
```


### Все сети в лабораторной работе должны иметь IP связность.

Для полной связности добавим маршруты в Чокурдах на смежных роутерах, а затем добавим распространение недостающих маршрутов в протокол BGP:
```
R25(config)#ip route 10.3.0.0 255.255.0.0 10.4.7.2
R25(config-router)#network 10.4.6.0 mask 255.255.255.252
R25(config-router)#network 10.4.7.0 mask 255.255.255.252
R25(config-router)#network 10.3.0.0 mask 255.255.0.0

R26(config)#ip route 10.3.0.0 255.255.0.0 10.4.8.2
R26(config-router)#network 10.4.8.0 mask 255.255.255.252
R26(config-router)#network 10.3.0.0 mask 255.255.0.0
```


Проверим из Москвы доступность адреса в Чокурдахе:
```
R14#ping 10.3.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


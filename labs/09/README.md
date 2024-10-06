# BGP. Основы

## Цель:
- Настроить BGP между автономными системами
- Организовать доступность между офисами Москва и С.-Петербург


## Описание/Пошаговая инструкция выполнения домашнего задания:

- Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
- Настроите eBGP между провайдерами Киторн и Ламас.
- Настроите eBGP между Ламас и Триада.
- Настроите eBGP между офисом С.-Петербург и провайдером Триада.
- Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.

## Выполнение

Настроим BGP на маршрутизаторах. В каждом укажем соседей и сети, которые надо странслировать:

R14:
```
R14(config)#router bgp 1001
R14(config-router)#bgp router-id 14.14.14.14
R14(config-router)#neighbor 10.1.8.2 remote-as 101
R14(config-router)#network 10.1.0.0 mask 255.255.248.0
R14(config)#ip route 10.1.0.0 255.255.248.0 Null 0
```
R14:
```
R15(config)#router bgp 1001
R15(config-router)#bgp router-id 15.15.15.15
R15(config-router)#neighbor 10.1.9.2 remote-as 301
R15(config-router)#network 10.1.0.0 mask 255.255.248.0
R15(config)#ip route 10.1.0.0 255.255.248.0 Null 0
```
R22:
```
R22(config)#router bgp 101
R22(config-router)#bgp router-id 22.22.22.22
R22(config-router)#neighbor 10.1.8.1 remote-as 1001
R22(config-router)#neighbor 10.1.10.2 remote-as 301
R22(config-router)#neighbor 10.4.4.1 remote-as 520
R22(config-router)#network 10.1.8.0 mask 255.255.255.252
R22(config-router)#network 10.1.10.0 mask 255.255.255.252
R22(config-router)#network 10.4.4.0 mask 255.255.255.252
```

R21
```
R21(config)#router bgp 301
R21(config-router)#bgp router-id 21.21.21.21
R21(config-router)#neighbor 10.1.9.1 remote-as 1001
R21(config-router)#neighbor 10.1.10.1 remote-as 101
R21(config-router)#neighbor 10.4.5.1 remote-as 520
R21(config-router)#network 10.1.9.0 mask 255.255.255.252
R21(config-router)#network 10.4.5.0 mask 255.255.255.252
R21(config-router)#network 10.1.10.0 mask 255.255.255.252
```

R24
```
R24(config)#router bgp 520
R24(config-router)#bgp router-id 24.24.24.24
R24(config-router)#neighbor 10.4.5.2 remote-as 301
R24(config-router)#neighbor 10.4.10.2 remote-as 2042
R24(config-router)#network 10.4.10.0 mask 255.255.255.252
R24(config-router)#network 10.4.5.0 mask 255.255.255.252
```

R18
```
R18(config)#router bgp 2042
R18(config-router)#bgp router-id 18.18.18.18
R18(config-router)#neighbor 10.4.10.1 remote-as 520
R18(config-router)#network 10.2.0.0 mask 255.255.0.0
R18(config)#ip route 10.2.0.0 255.255.0.0 Null 0
```




## Проверка
Состояние BGP:
```
R18#sh ip bgp
BGP table version is 12, local router ID is 18.18.18.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.1.0.0/21      10.4.10.1                              0 520 301 1001 i
 *>  10.1.8.0/30      10.4.10.1                              0 520 301 101 i
 *>  10.1.9.0/30      10.4.10.1                              0 520 301 i
 *>  10.1.10.0/30     10.4.10.1                              0 520 301 i
 *>  10.2.0.0/16      0.0.0.0                  0         32768 i
 *>  10.4.4.0/30      10.4.10.1                              0 520 301 101 i
 *>  10.4.5.0/30      10.4.10.1                0             0 520 i
 r>  10.4.10.0/30     10.4.10.1                0             0 520 i
```

Проверим связь с пограничного маршрутизатора АС С.-Петербурга R18 до пограничного маршрутищзатора Москвы R15:
```
R18#ping 10.1.9.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.9.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


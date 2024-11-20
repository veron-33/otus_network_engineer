# VPN. GRE. DmVPN

## Цель:
- Настроить GRE между офисами Москва и С.-Петербург
- Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги


## Описание/Пошаговая инструкция выполнения домашнего задания:

- Настроите GRE между офисами Москва и С.-Петербург.
- Настроите DMVMN между Москва и Чокурдах, Лабытнанги.

## Выполнение

### 1. Настроите GRE между офисами Москва и С.-Петербург.

Выполним настройки на маршрутизаторах в Москве (R15) и С.-Петербурге (R18):
```
R15(config)#int tunnel 00
R15(config-if)#tunnel mode gre ip
R15(config-if)#ip address 10.100.0.1 255.255.255.0
R15(config-if)#tunnel destination 10.4.10.2
R15(config-if)#tunnel source 10.1.9.1
R15(config-if)#ip mtu 1400
R15(config-if)#ip tcp adjust-mss 1360
```

```
R18(config)#int tun00
R18(config-if)#tunnel mode gre ip
R18(config-if)#ip address 10.100.0.2 255.255.255.0
R18(config-if)#tunnel destination 10.1.9.1
R18(config-if)#tunnel source 10.4.10.2
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#ip mtu 1400
```

Сразу же проверим, что туннель поднялся. Пинганем протвоположную сторону:
```
R15#ping 10.100.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
Чтобы маршрутизация между сетями этих АС происходила через туннель, добавим на маршрутизаторах доп. маршруты:
```
R15(config)#ip route 10.2.0.0 255.255.0.0 10.100.0.2

R18(config)#ip route 10.1.0.0 255.255.248.0 10.100.0.1
```

Теперь проверим доступность с R15 устройства в С.-Петербурге за маршрутизатором R18 и убедимся что трафик прошели именно через туннель:
```
R15#traceroute 10.2.0.2
Type escape sequence to abort.
Tracing the route to 10.2.0.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.100.0.2 [AS 301] 5 msec 5 msec 0 msec
  2 10.2.0.2 [AS 2042] 1 msec *  5 msec
```


### 2. Настроите DMVMN между Москва и Чокурдах, Лабытнанги.

Настроим DMVPN Phase 2, чтобы трафик ходил между spoke-маршрутизаторами напрямую, не через hub.  
В качестве hub выберем маршрутизатор R14:
```
R14(config)#int tun 00
R14(config-if)#tunnel mode gre multipoint
R14(config-if)#ip address 10.64.0.1 255.255.255.0
R14(config-if)#tunnel source e0/2
R14(config-if)#ip mtu 1400
R14(config-if)#ip tcp adjust-mss 1360
R14(config-if)#ip nhrp network-id 100
R14(config-if)#ip nhrp map multicast dynamic
```

Настроим spoke R28:
```
R28(config)#int tun 00
R28(config-if)#tunnel mode gre multipoint
R28(config-if)#ip address 10.64.0.2 255.255.255.0
R28(config-if)#tunnel source e0/0
R28(config-if)#ip mtu 1400
R28(config-if)#ip tcp adjust-mss 1360
R28(config-if)#ip nhrp network-id 100
R28(config-if)#ip nhrp map multicast 10.1.8.1
R28(config-if)#ip nhrp nhs 10.64.0.1
R28(config-if)#ip nhrp map 10.64.0.1 10.1.8.1
```

Сразу проверим, что туннель поднялся:
```
R28#ping 10.64.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.64.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

Настроим spoke R27:
```
R27(config)#int tun 00
R27(config-if)#tunnel mode gre multipoint
R27(config-if)#ip address 10.64.0.3 255.255.255.0
R27(config-if)#tunnel source e0/0
R27(config-if)#ip mtu 1400
R27(config-if)#ip tcp adjust-mss 1360
R27(config-if)#ip nhrp network-id 100
R27(config-if)#ip nhrp map multicast 10.1.8.1
R27(config-if)#ip nhrp nhs 10.64.0.1
R27(config-if)#ip nhrp map 10.64.0.1 10.1.8.1
```

Проверим, что доступны остальные участники DMVPN-сети:
```
R27#ping 10.64.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.64.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R27#ping 10.64.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.64.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

Продемонстрируем статус DMVPN на R14 (hub):
```
R14#sh dmvp
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.4.8.2              10.64.0.2    UP 00:07:34     D
     1 10.4.6.2              10.64.0.3    UP 00:02:21     D
```

Как и в первой задаче добавим маршруты, чтобы трафик ходил через туннели:
```
R14(config)#ip route 10.3.0.0 255.255.0.0 10.64.0.2

R27(config)#ip route 10.3.0.0 255.255.0.0 10.64.0.2
R27(config)#ip route 10.1.0.0 255.255.248.0 10.64.0.1

R28(config)#ip route 10.1.0.0 255.255.248.0 10.64.0.1
```

# Основные протоколы сети интернет

## Цель:
- Настроить DHCP в офисе Москва
- Настроить синхронизацию времени в офисе Москва
- Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах

## Описание/Пошаговая инструкция выполнения домашнего задания:

- Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
- Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
- Настроите статический NAT для R20.
- Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
- _Настроите статический NAT(PAT) для офиса Чокурдах._
- Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
- Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.



## Выполнение

### 1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.

Настроим на R14 и R15 NAT(PAT):
```
R14(config)#access-list 1 permit 10.1.0.0 0.0.7.255
R14(config)#ip nat inside source list 1 interface e0/2 overload
R14(config)#int range e0/0-1, e0/3, e1/0
R14(config-if-range)#ip nat inside
R14(config-if-range)#int e0/2
R14(config-if)#ip nat outside

R15(config)#access-list 1 permit 10.1.0.0 0.0.7.255
R15(config)#ip nat inside source list 1 interface e0/2 overload
R15(config)#int range e1/0,e0/0-1,e0/3
R15(config-if-range)#ip nat inside
R15(config-if-range)#int e0/2
R15(config-if)#ip nat outside
```

Для проверки создадим трафик из локальной сети во внешнюю. Запустим пинг с R12 до Киторна:
```
R12#ping 10.1.8.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.8.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

После этого на R14 появилась запись
```
R14#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.1.8.1:0        10.1.1.2:0         10.1.8.2:0         10.1.8.2:0
```


### 2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.

На R18 создадим пул из 5 адресов "внешнего" интерфейса и применим к настройке NAT(PAT):
```
R18(config)#ip nat pool NAT_POOL1 10.4.10.5 10.4.10.9 netmask 255.255.255.0
R18(config)#access-list 1 permit 10.2.0.0 0.0.255.255
R18(config)#ip nat inside source list 1 pool NAT_POOL1
R18(config)#int range e0/0-1
R18(config-if-range)#ip nat inside
R18(config-if-range)#int e0/2
R18(config-if)#ip nat outside
```

Проверим:
```
R18#sh ip nat tra
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.4.10.5:0       10.2.0.2:0         10.4.10.1:0        10.4.10.1:0
icmp 10.4.10.5:1       10.2.0.2:1         10.4.10.1:1        10.4.10.1:1
--- 10.4.10.5          10.2.0.2           ---                ---
```


### 3. Настроите статический NAT для R20.

На R15 выделим отдельный внешний адрес и создадим запись для статического NAT, указав внутренний адрес R20:
```
R15(config)#ip nat inside source static 10.1.5.2 10.1.9.20
```

Проверка:
```
R15#sh ip nat tra
Pro Inside global      Inside local       Outside local      Outside global
--- 10.1.9.20          10.1.5.2           ---                ---
```

### 4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
Для этого на R14 у интерфейса e0/3 отключим настройку NAT, тем самым адрес роутера R19 будет маршрутизироваться без преобразования адресов:
```
R14(config-if)#no ip nat inside
```
Для проверки проверим доступность R19 с роутера R24:
```
R24#ping 10.1.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


### 5*. Настроите статический NAT(PAT) для офиса Чокурдах.

Для этого на R28 в дополнение к обычной настройке NAT добавим route-map для корректной трансляции NAT в связке с PBR:

```
R28(config)#int range e0/0-1
R28(config-if-range)#ip nat outside

R28(config)#int e0/2.10
R28(config-subif)#ip nat inside
R28(config-subif)#int e0/2.20
R28(config-subif)#ip nat inside

R28(config)#access-list 1 permit 10.3.1.0 0.0.0.255
R28(config)#access-list 1 permit 10.3.2.0 0.0.0.255

R28(config)#route-map TO_ISP1 permit 10
R28(config-route-map)#match ip address 1
R28(config-route-map)#match interface e0/1
R28(config-route-map)#exit
R28(config)#route-map TO_ISP2 permit 10
R28(config-route-map)#match ip address 1
R28(config-route-map)#match interface e0/0

R28(config)#ip nat inside source route-map TO_ISP1 interface e0/1 overload
R28(config)#ip nat inside source route-map TO_ISP2 interface e0/0 overload

```


### 5. Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.

Настроим на R12 DHCP-сервер, исключив из пула выдаваемых адресов уже занятые адреса, в т.ч. адрес х.х.х.254, используемый для HSRP:
```
R12(config)#ip dhcp excluded-address 10.1.7.1 10.1.7.5
R12(config)#ip dhcp excluded-address 10.1.7.254 10.1.7.254
R12(config)#ip dhcp pool LAN-POOL-7
R12(dhcp-config)#network 10.1.7.0 255.255.255.0
R12(dhcp-config)#default-router 10.1.7.254
```
Проверим на VPC1:
```
VPCS> ip dhcp
DORA IP 10.1.7.6/24 GW 10.1.7.254
```

Сделаем тоже самое на R13:
```
R13(config)#ip dhcp excluded-address 10.1.6.1 10.1.6.5
R13(config)#ip dhcp excluded-address 10.1.6.254 10.1.6.254
R13(config)#ip dhcp pool LAN-POOL-6
R13(dhcp-config)#network 10.1.6.0 255.255.255.0
R13(dhcp-config)#default-router 10.1.6.254
```
Проверим на VPC7:
```
VPCS> ip dhcp
DORA IP 10.1.6.6/24 GW 10.1.6.254
```

### 6. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.

Настроим R12 и R13 как NTP-серверы:
```
R12(config)#ntp master 3

R13(config)#ntp master 3
```

Настроим остальные устройства как NTP-клиенты, указав в качестве сервер роутеры R12 и R13. Приведем пример такой настройки на R14:
```
R14(config)#ntp server 10.1.1.2
R14(config)#ntp server 10.1.2.2
```

Проверим работу NTP:
```
R14(config)#do sh ntp stat
Clock is synchronized, stratum 4, reference is 10.1.1.2
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 2200 (1/100 of seconds), resolution is 4000
reference time is EADF7021.D0A3D948 (18:36:17.815 UTC Wed Nov 13 2024)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 193.00 msec, peer dispersion is 189.44 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 64, last update was 13 sec ago.

R14(config)#do sh ntp ass
  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.1.1.2        127.127.1.1      3     23     64     1  0.000   0.000 189.44
+~10.1.2.2        127.127.1.1      3     14     64     1  0.000   0.000 189.45
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
```

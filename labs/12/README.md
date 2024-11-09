
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
- Все офисы в лабораторной работе должны иметь IP связность.


## Выполнение

### 1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
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



проверка

пинганем с r12
```
R12#ping 10.1.8.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.8.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

на р14 появилась запись
```
R14#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.1.8.1:0        10.1.1.2:0         10.1.8.2:0         10.1.8.2:0
```

### 2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.


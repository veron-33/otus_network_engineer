# IS-IS

## Цель:
- Настроить IS-IS офисе Триада


## Описание/Пошаговая инструкция выполнения домашнего задания:
- Настроите IS-IS в ISP Триада.
- R23 и R25 находятся в зоне 2222.
- R24 находится в зоне 24.
- R26 находится в зоне 26.

## Выполнение

На каждом роутере настроим IS-IS с указанными в задании номерами зон. Все устройсвта будут иметь тип L2. Затем включим протокол на интерфейсах.

R23:
```
R23(config)#router isis 1
R23(config-router)#net 49.2222.0000.0000.0023.00
R23(config-router)#is-type level-2-only
R23(config-router)#int range e0/1-2
R23(config-if-range)#ip  router isis 1
```
R25:
```
R25(config)#router isis 1
R25(config-router)#net 49.2222.0000.0000.0025.00
R25(config-router)#is-type level-2-only
R25(config-router)#int range e0/0,e0/2
R25(config-if-range)#ip router isis 1
```
R24:
```
R24(config)#router isis 1
R24(config-router)#net 49.0024.0000.0000.0024.00
R24(config-router)#is-type level-2-only
R24(config-router)#int range e0/1-2
R24(config-if-range)#ip router isis 1
```
R26:
```
R26(config)#router isis 1
R26(config-router)#net 49.0026.0000.0000.0026.00
R26(config-router)#is-type level-2-only
R26(config-router)#int range e0/0,e0/2
R26(config-if-range)#ip router isis 1
```

## Проверка
Выполним проверку доступности с роутера R23 на адрес интерфейса е0/0 роутера R26:
```
R23#ping 10.4.2.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.4.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
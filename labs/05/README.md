# Маршрутизация на основе политик (PBR)

## Цель:
- Настроить политику маршрутизации в офисе Чокурдах
- Распределить трафик между 2 линками


## Описание/Пошаговая инструкция выполнения домашнего задания:

Необходимо:  
- Настроить политику маршрутизации для сетей офиса.
- Распределить трафик между двумя линками с провайдером.
- Настроить отслеживание линка через технологию IP SLA.(только для IPv4)
- Настройте для офиса Лабытнанги маршрут по-умолчанию.

## Выполнение

### Настройка политики и распределение трафика между двумя провайдерами

Выбрана следующая политика: Трафик для AS 101 (Киторн),301 (Ламас) и 1001 (Москва) должен ходить через R25, а весь остальной через R26. Все настройки выполняем на R28.

1. Устанавливаем шлюз по умолчанию до R26
```  
R28(config)#ip route 0.0.0.0 0.0.0.0 10.4.8.1
```

2. Для Москвы, Ламаса и Киторна устанавливаем второй маршрут до R25:  
Создаем аксес лист для сети назначения:  
```
R28(config)#ip access-list standard MSK
R28(config-std-nacl)#permit 10.1.0.0 0.0.255.255
```
Создаем маршрутную карту:
```
R28(config)#route-map to_msk permit 10
R28(config-route-map)#match ip address MSK
R28(config-route-map)#set ip  next-hop 10.4.7.1
```
Применяем политику:
```
R28(config)#int e0/2
R28(config-if)#ip policy route-map to_msk
```

Проверяем:
```
R28#sh ip access-lists 
Standard IP access list MSK
    10 permit 10.1.0.0, wildcard bits 0.0.255.255

R28#sh route-map 
route-map to_msk, permit, sequence 10
  Match clauses:
    ip address (access-lists): MSK 
  Set clauses:
    ip next-hop 10.4.7.1
  Policy routing matches: 0 packets, 0 bytes

R28#sh ip policy
Interface      Route map
Ethernet0/2    to_msk
```


### Настройка остлеживания трафика (IP SLA)

По умолчанию настроен шлюз R26. Проверяем его доступность, и если недоступен уходим на R25.

Создаем SLA
```
R28(config)#ip sla 1
R28(config-ip-sla)#icmp-echo 10.4.8.1 source-interface e0/0

R28(config)#ip sla schedule 1 life forever start-time now
R28(config)#track 1 ip sla 1 reachability 
```

Создадим маршрут с большей метрикой:
```
R28(config)#ip route 0.0.0.0 0.0.0.0 10.4.7.1 120 track 1 
```

Проверка
```
R28#sh ip sla configuration 
IP SLAs Infrastructure Engine-III
Entry number: 1
Owner: 
Tag: 
Operation timeout (milliseconds): 5000
Type of operation to perform: icmp-echo
Target address/Source interface: 10.4.8.1/Ethernet0/0
Type Of Service parameter: 0x0
Request size (ARR data portion): 28
Verify data: No
Vrf Name: 
Schedule:
   Operation frequency (seconds): 60  (not considered if randomly scheduled)
   Next Scheduled Start Time: Start Time already passed
   Group Scheduled : FALSE
   Randomly Scheduled : FALSE
   Life (seconds): Forever
   Entry Ageout (seconds): never
   Recurring (Starting Everyday): FALSE
   Status of entry (SNMP RowStatus): Active
Threshold (milliseconds): 5000
Distribution Statistics:
   Number of statistic hours kept: 2
   Number of statistic distribution buckets kept: 1
   Statistic distribution interval (milliseconds): 20
Enhanced History:
History Statistics:
   Number of history Lives kept: 0
   Number of history Buckets kept: 15
   History Filter Type: None
```

Статистика:
```
R28#sh ip sla statistics 
IPSLAs Latest Operation Statistics

IPSLA operation id: 1
	Latest RTT: 1 milliseconds
Latest operation start time: 10:17:04 UTC Thu Aug 15 2024
Latest operation return code: OK
Number of successes: 2
Number of failures: 1
Operation time to live: Forever
```

### Настройка шлюза в Лабытнаги:
Добавялем маршрут по-умолчанию на R27:
```
R27(config)#ip route 0.0.0.0 0.0.0.0 10.4.6.1
```

Проверка:
```
R27#ping 10.4.1.1      
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.4.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
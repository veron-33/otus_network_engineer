# DHCPv4/v6 и SLAAC 

## 1. Лабораторная работа DHCPv4

### Часть 1. Построение сети и базованя настройка устройств
В рамках этой работы на первом этапе была установлена адресная схема для сегментов сети. Результат представлен в таблице:

<!-- rowspan в markdown нельзя... -->
<table>
<thead>
<tr>
<th>Устройство</th>
<th>Интерфейс</th>
<th>IP адрес</th>
<th>Маска</th>
<th>Шлюз</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="5">R1</td>
<td>e0/0</td>
<td>10.0.0.1</td>
<td>255.255.255.252</td>
<td rowspan="5">-</td>
</tr>

<tr>
<td>e0/1</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>e0/1.100</td>
<td>192.168.1.1</td>
<td>255.255.255.192</td>
</tr>

<tr>
<td>e0/1.200</td>
<td>192.168.1.65</td>
<td>255.255.255.224</td>
</tr>
 <tr>
<td>e0/1.1000</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td rowspan="2">R2</td>
<td>e0/0</td>
<td>10.0.0.2</td>
<td>255.255.255.252</td>
<td rowspan="2">-</td>
</tr>

<tr>
<td>e0/1</td>
<td>192.168.1.97</td>
<td>255.255.255.240</td>
</tr>

<tr>
<td>S1</td>
<td>VLAN 200</td>
<td>192.168.1.66</td>
<td>255.255.255.224</td>
<td>192.168.1.65</td>
</tr>

<tr>
<td>S2</td>
<td>VLAN 1</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>PC-A</td>
<td>eth</td>
<td>DHCP</td>
<td>DHCP</td>
<td>DHCP</td>
</tr>

<tr>
<td>PC-B</td>
<td>eth</td>
<td>DHCP</td>
<td>DHCP</td>
<td>DHCP</td>
</tr>
</tbody>
</table>

Далее сконфигурирован роутер R1:
```
R1(config)#do sh ip int br  
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  administratively down down    
Ethernet0/1                unassigned      YES unset  up                    up      
Ethernet0/1.100            192.168.1.1     YES manual up                    up      
Ethernet0/1.200            192.168.1.65    YES manual up                    up      
Ethernet0/1.1000           unassigned      YES unset  up                    up      
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down 
```

Далее назначен адрес интерфейсу на R2 и проверена связь между роутерами:

```
R1(config)#do ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
Затем был настроен коммутатор S1 с назначением необходимых VLAN:

```
S1#sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0
100  clients                          active    Et0/1
200  mngnt                            active    
999  parking                          active    Et0/2, Et0/3
1000 native                           active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

Порт e0/0 этого коммутатора, идущий к роутеру переводим в trunk и разрешаем в нем необходимые VLAN:

```
S1#sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/0       100,200,1000

Port        Vlans allowed and active in management domain
Et0/0       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       100,200,1000
```

### Часть 2. Настройка серверов DHCPv4

В этой части на роутере R1 были настроены два пула DHCPv4 для двух пордсетей и настроены основные параметры DHCP сервера. В виду отсутствия на данном этапе запущенных клиентов, которые могли бы запросить сетевые настройки, то пока наши сервера не выдали адресов. Настройки пула и статистика выданных адресов представлена ниже:
```
R1#sh ip dhcp pool

Pool clients_pool :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.1          192.168.1.1      - 192.168.1.62      0

Pool r2_clients_pool :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.97         192.168.1.97     - 192.168.1.110     0
```
```
R1#sh ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/	 	    Lease expiration        Type
		    Hardware address/
		    User name
```
```
R1#sh ip dhcp server statistics 
Memory usage         25176
Address pools        2
Database agents      0
Automatic bindings   0
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         0
DHCPREQUEST          0
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            0
DHCPACK              0
DHCPNAK              0
```

После запуска компьютера PC-A, он успешно получил сетевые настройки. Пинг до роутера это подтвердил:
```
VPCS> ip dhcp
DDORA IP 192.168.1.6/26 GW 192.168.1.1

VPCS> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=255 time=0.274 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=255 time=0.487 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=255 time=0.431 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=255 time=0.441 ms
```

### Часть 3. Настройка DHCP Relay

Для того, чтобы компьютер PC-B получил настройки от DHCP сервера, расположенного на R1, необходимо на роутере R2 настроить DHCP Relay.  
После настройки R2 и включения компьютера PC-B, последний успешно получил сетевые настройки, о чем также свидетельствует вывод списка выданных адресов на R1:
```
R1#show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/	 	    Lease expiration        Type
		    Hardware address/
		    User name
192.168.1.6         0100.5079.6668.05       Jul 19 2024 11:31 PM    Automatic
192.168.1.102       0100.5079.6668.06       Jul 20 2024 12:53 AM    Automatic
```
Как можно увидеть адрес выдан из второго пула, как и должно быть.  
Стастистика запросов к DHCP серверу теперь такая:
```
R1#show ip dhcp server statistics 
Memory usage         42094
Address pools        2
Database agents      0
Automatic bindings   2
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         4
DHCPREQUEST          2
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            2
DHCPACK              2
DHCPNAK              0
```


## 2. Лабораторная работа DHCPv6
### Часть 1. Построение сети и базованя настройка устройств
В рамках этой работы на первом этапе была установлена адресная схема для сегментов сети. Результат представлен в таблице:

<!-- rowspan в markdown нельзя... -->
<table>
<thead>
<tr>
<th>Устройство</th>
<th>Интерфейс</th>
<th>IPм6 адрес</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="4">R1</td>
<td rowspan="2">e0/0</td>
<td>2001:db8:acad:2::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>
<tr>
<td rowspan="2">e0/1</td>
<td>2001:db8:acad:1::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>

<tr>
<td rowspan="4">R2</td>
<td rowspan="2">e0/0</td>
<td>2001:db8:acad:2::2/64</td>
</tr>
<tr>
<td>fe80::2</td>
</tr>
<tr>
<td rowspan="2">e0/1</td>
<td>2001:db8:acad:3::1/64</td>
</tr>
<tr>
<td>fe80::1</td>
</tr>

<tr>
<td>PC-A</td>
<td>eth</td>
<td>DHCP</td>
</tr>
<tr>
<td>PC-B</td>
<td>eth</td>
<td>DHCP</td>
</tr>
</tbody>
</table> 

После включения и первоначальной настройки роутеров R1 и R2 их интерфейсам были присвоены адреса из таблицы и проверена связь между роутерами:
```
R1#ping 2001:db8:acad:3::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:ACAD:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### Часть 2. Проверка присвоения адреса методом SLAAC
Включив компьютер PC-A мы удостоверились, что он настроил себе IPv6-адрес исходя из адреса интерфейса на роутере R1:
```
PC-A> show ipv6

NAME              : PC-A[1]
LINK-LOCAL SCOPE  : fe80::250:79ff:fe66:6805/64
GLOBAL SCOPE      : 2001:db8:acad:1:2050:79ff:fe66:6805/64
DNS               :
ROUTER LINK-LAYER : aa:bb:cc:00:10:10
MAC               : 00:50:79:66:68:05
LPORT             : 20000
RHOST:PORT        : 127.0.0.1:30000
MTU:              : 1500
```

### Часть 3. Настройка DHCPv6 сервера (без сохранения состояния) на маршрутизаторе R1
На этом этапе был настроен DHCPv6 сервер на роутере R1 в режиме _other-config-flag_. Далее после того как компьютер PC-A перезагрузился и получил новые настройки была проверена связь с него до второго маршрутизатора R2:
```
PC-A> ping 2001:db8:acad:3::1

2001:db8:acad:3::1 icmp6_seq=1 ttl=63 time=9.265 ms
2001:db8:acad:3::1 icmp6_seq=2 ttl=63 time=0.610 ms
2001:db8:acad:3::1 icmp6_seq=3 ttl=63 time=0.571 ms
2001:db8:acad:3::1 icmp6_seq=4 ttl=63 time=0.593 ms
```

### Часть 4. Настройка DHCPv6 сервера (с сохранением состояния) на маршрутизаторе R1
Теперь был настроен второй пул и настроен полноценный DHCPv6 сервер с выдачей адресов (без использования SLAAC).

### Часть 5. Настройка и проверка DHCPv6 Relay на маршрутизаторе R2

Для того, чтобы компьютер PC-B смог получить настройки от DHCPv6 сервера, настроенного на R1, необходимо, чтобы на маршрутизаторе R2 был настроен DHCPv6 Relay.  
После такой настройки компьютер PC-B был перезагружен. После того, как он получил настройки была проверена связь до роутера R1: 

```
PC-B> ping 2001:db8:acad:1::1

2001:db8:acad:1::1 icmp6_seq=1 ttl=63 time=9.964 ms
2001:db8:acad:1::1 icmp6_seq=2 ttl=63 time=0.552 ms
2001:db8:acad:1::1 icmp6_seq=3 ttl=63 time=0.802 ms
```


#
К отчёту также прикладываю [выгрузки лабораторных стендов EVE-NG](eve-ng/)
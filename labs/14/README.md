# IPSec over DmVPN

## Цель:
- Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
- Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


## Описание/Пошаговая инструкция выполнения домашнего задания:

- Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
- Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
- Все узлы в офисах в лабораторной работе должны иметь IP связность.

## Выполнение


### 1. Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.

Выполним настройку двух фаз IPSEC на R18 и применим это шифорвание к созданному на прошлой лабораторной работе GRE-туннелю:

```
R18(config)#crypto isakmp policy 10
R18(config-isakmp)#encryption aes
R18(config-isakmp)#hash md5
R18(config-isakmp)#authentication pre-share
R18(config-isakmp)#group 2
R18(config-isakmp)#exit
R18(config)#crypto isakmp key MY_PASS address 10.1.9.1

R18(config)#crypto ipsec transform-set IPSEC_TS esp-aes esp-md5-hmac
R18(cfg-crypto-trans)#mode transport
R18(cfg-crypto-trans)#exit

R18(config)#crypto map IPSEC 10 ipsec-isakmp
R18(config-crypto-map)#set peer 10.100.0.1
R18(config-crypto-map)#set transform-set IPSEC_TS
R18(config-crypto-map)#match address IPSEC_MSK
R18(config-crypto-map)#exit

R18(config)#ip access-list extended IPSEC_MSK
R18(config-ext-nacl)#permit ip 10.2.0.0 0.0.255.255 10.1.0.0 0.0.7.255
R18(config-ext-nacl)#permit ip 10.1.0.0 0.0.7.255 10.2.0.0 0.0.255.255
R18(config-ext-nacl)#exit

R18(config)#crypto ipsec profile IPSEC_PROFILE
R18(ipsec-profile)#set transform-set IPSEC_TS

R18(config)#int tu0
R18(config-if)#tunnel protection ipsec profile IPSEC_PROFILE

```
Выполним аналогичные настройки со стороны Москвы на R15. После 
этого проверим, что обе фазы поднялись и работают:

```
R18#sh crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
10.1.9.1        10.4.10.2       QM_IDLE           1001 ACTIVE


R18#sh cryp sess
Crypto session current status

Interface: Tunnel0
Session status: UP-ACTIVE
Peer: 10.1.9.1 port 500
  Session ID: 0
  IKEv1 SA: local 10.4.10.2/500 remote 10.1.9.1/500 Active
  IPSEC FLOW: permit 47 host 10.4.10.2 host 10.1.9.1
        Active SAs: 2, origin: crypto map
```
Также проверим, что связность между сетями Москва и С.-Петербург не пропала. Из Москвы запустим тест до устройства сети С.-Петербурга:
```
R15#ping 10.4.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.4.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```


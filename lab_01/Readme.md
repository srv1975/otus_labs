# Лабораторная работа "VLAN и маршрутизация между VLAN"

### Цель лабораторной работы
Т.к. четких требований к выполнению задания сформулировано не было, попробую сформулировать их самостоятельно:
 * В среде EVE-NG собрать схему из маршрутизатора R1 , 2-х коммутаторов SW1, SW2 и нескольких рабочих станций. От
себя добавлю управляющую сеть, подключенную к бриджу pnet1 хостовой машины (сделано для того, чтобы иметь telnet
доступ к оборудованию и возможность сливать конфиги на tftp сервер хостовой машины.
 * Выполнить предварительную настройку оборудования (установить пароль 'cisco' на доступ telnet, console, задать пароль
'cisco' на enable, задать hostname).
 * Пробросить Vlan на соответствющие порты коммутаторов согласно таблице.
 * Поднять сабинтерфейсы на маршрутизаторе согласно плану адресации.
 * На маршрутизаторе сабинтерфейс e0/0.99 поместить в VRF, на коммутаторах поднять Vlan интерфейсы с соответствующими адресами.
 * Задать IP-адреса на VPC.
 * Проверить связность узлов сети. VPC1 и VPC2 должны видеть друг друга и интерфейсы маршрутизатора e0/0.10 и e0/0.20, адреса из сети 
172.16.0.0/24 не должны быть доступны. VPC_MNG должна видеть только хосты из сети 172.16.0.0/24.
 * Выгрузить на TFTP сервер конфиги коммутаторов и маршрутизаторов.


### Схема лабораторной работы

![Схема лабораторной работы](/lab_01/lab_01.png)


### План IP адресации


| Device | Interface | Ip Address                | Subnet Mask    | Default Gateway |
|--------|-----------|---------------------------|----------------|-----------------|
|R1      |e0/0.10    |192.168.10.1               |255.255.255.0   |N/A              |
|        |           |fd00:000a:000b:0001::/64   |                |                 |
|        |e0/0.20    |192.168.20.1               |255.255.255.0   |                 |
|        |           |fd00:000a:000b:0002::/64   |                |                 |
|        |e0/0.99 vrf MNG   |172.16.0.1          |255.255.255.0   |                 |
|SW1     |vlan99     |172.16.0.2                 |255.255.255.0   |172.16.0.1       |
|SW2     |vlan99     |172.16.0.3                 |255.255.255.0   |172.16.0.1       |
|VPC1    |NIC        |192.168.10.2               |255.255.255.0   |192.168.10.1     |
|VPC2    |NIC        |192.168.20.2               |255.255.255.0   |192.168.20.1     |
|VPC_MNG |NIC        |172.16.0.4                 |255.255.255.0   |172.16.0.1       |
|eve-ng  |pnet1      |172.16.0.254               |255.255.255.0   |N/A              |


### VLAN's

| VLAN | Name      | Interface Assigned |
|------|-----------|--------------------|
|2     |Native|N/A |                    |
|10    |LocalNet_1 |SW1:e0/0(trunk), e0/1(access)|
|20    |LocalNet_2 |SW1:e0/0,e0/3(trunk), </br> SW2:e0/3(trunk), e0/1(access)|
|99    |Managment  |SW1:e0/0, e0/3(trunk), e0/2(access), VLAN99 </br>SW2:e0/3(trunk), e0/2(access),VLAN99|



### Выводы

Подробно расписывать настройку оборудования не вижу смысла, она тривиальна. К томуже текстовые конфиги
активного оборудования прилагаются, поэтому перейду сразу к проверке работоспособности.

#### VPC1
```
VPCS> show

NAME   IP/MASK              GATEWAY                             GATEWAY
VPCS1  192.168.10.2/24      192.168.10.1
       fe80::250:79ff:fe66:6805/64
       fd00:a:b:1:250:79ff:fe66:6805/64 
```

Проверим доступность шлюза
```
VPCS> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=255 time=0.417 ms
. . .
84 bytes from 192.168.10.1 icmp_seq=5 ttl=255 time=0.625 ms
```

Проверим доступность VPC2
```
VPCS> ping 192.168.20.2

84 bytes from 192.168.20.2 icmp_seq=1 ttl=63 time=1.871 ms
. . .
84 bytes from 192.168.20.2 icmp_seq=5 ttl=63 time=1.027 ms
```

Проверим доступность VPC_MNG
```
VPCS> ping 172.16.0.4  

*192.168.10.4 icmp_seq=1 ttl=255 time=0.616 ms (ICMP type:3, code:1, Destination host unreachable)
. . .
*192.168.10.4 icmp_seq=5 ttl=255 time=0.702 ms (ICMP type:3, code:1, Destination host unreachable)

VPCS>                
```
Что и требовалось доказать. Управляющая сеть не доступна из сети 192.168.10.0/24


Проверим доступность VPC2 по IPv6
```
VPCS> ping fd00:a:b:2:250:79ff:fe66:6801

fd00:a:b:2:250:79ff:fe66:6801 icmp6_seq=1 ttl=62 time=19.300 ms
. . .
fd00:a:b:2:250:79ff:fe66:6801 icmp6_seq=5 ttl=62 time=1.034 ms

VPCS> 
```

#### VPC2
```
VPCS> show

NAME   IP/MASK              GATEWAY                             GATEWAY
VPCS1  192.168.20.2/24      192.168.20.1
       fe80::250:79ff:fe66:6801/64
       fd00:a:b:2:250:79ff:fe66:6801/64 
```


Проверим доступность шлюза
```
VPCS> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=255 time=0.537 ms
. . . 
84 bytes from 192.168.20.1 icmp_seq=5 ttl=255 time=0.969 ms
```


Проверим доступность VPC1
```
VPCS> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=63 time=2.093 ms
. . .
84 bytes from 192.168.10.2 icmp_seq=5 ttl=63 time=1.107 ms
```


Проверим доступность VPC_MNG
```
VPCS> ping 172.16.0.1  

*192.168.20.1 icmp_seq=1 ttl=255 time=0.959 ms (ICMP type:3, code:1, Destination host unreachable)
. . .
*192.168.20.1 icmp_seq=5 ttl=255 time=0.900 ms (ICMP type:3, code:1, Destination host unreachable)
```


Проверим доступность VPC1 по IPv6
```
VPCS> ping fd00:a:b:1:250:79ff:fe66:6805

fd00:a:b:1:250:79ff:fe66:6805 icmp6_seq=1 ttl=62 time=1.492 ms
. . .
fd00:a:b:1:250:79ff:fe66:6805 icmp6_seq=5 ttl=62 time=0.997 ms
```

#### VPC_MNG
```
VPCS> show 

NAME   IP/MASK              GATEWAY                             GATEWAY
VPCS1  172.16.0.4/24        172.16.0.1
       fe80::250:79ff:fe66:6806/64
```

Проверим доступность R1, SW1, SW2
```
VPCS> ping 172.16.0.1

84 bytes from 172.16.0.1 icmp_seq=1 ttl=255 time=0.685 ms
. . .
84 bytes from 172.16.0.1 icmp_seq=5 ttl=255 time=0.912 ms


VPCS> ping 172.16.0.2

84 bytes from 172.16.0.2 icmp_seq=1 ttl=255 time=0.405 ms
. . .
84 bytes from 172.16.0.2 icmp_seq=5 ttl=255 time=0.642 ms


VPCS> ping 172.16.0.3

84 bytes from 172.16.0.3 icmp_seq=1 ttl=255 time=0.261 ms
. . .
84 bytes from 172.16.0.3 icmp_seq=5 ttl=255 time=0.307 ms
```

Проверим доступность VPC1 и VPC2
```
VPCS> ping 192.168.10.2

*172.16.0.1 icmp_seq=1 ttl=255 time=0.896 ms (ICMP type:3, code:1, Destination host unreachable)
. . .
*172.16.0.1 icmp_seq=5 ttl=255 time=0.952 ms (ICMP type:3, code:1, Destination host unreachable)


VPCS> ping 192.168.20.2

*172.16.0.1 icmp_seq=1 ttl=255 time=0.967 ms (ICMP type:3, code:1, Destination host unreachable)
. . .
*172.16.0.1 icmp_seq=5 ttl=255 time=0.948 ms (ICMP type:3, code:1, Destination host unreachable)
```
Что и требовалось доказать



#### Выгрузка конфигов на примере R1

```
User Access Verification

Password: 
R1>en
Password: 
R1#
R1#copy startup-config tftp:
Address or name of remote host []? 172.16.0.254
Destination filename [r1-confg]? 2023-09-14_R1.conf
.!!
1414 bytes copied in 4.014 secs (352 bytes/sec)

R1#
```
 
### Материалы

[Лабораторная работа](/lab_01/Lab_01.zip)

[Файл конфигурации R1](/lab_01/2023-09-14_R1.conf)

[Файл конфигурации SW1](/lab_01/2023-09-14_SW1.conf)

[Файл конфигурации SW2](/lab_01/2023-09-14_SW2.conf)

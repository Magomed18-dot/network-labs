# Лаба 01: VLAN, trunk и маршрутизация между VLAN

Учебная лабораторная работа в Cisco Packet Tracer: настройка VLAN на двух коммутаторах, trunk-линк между ними и маршрутизация между VLAN (Router-on-a-stick).

## Цель

- Разделить сеть на две VLAN (SALES и IT) на двух коммутаторах.
- Настроить trunk-порт между коммутаторами.
- Проверить, что устройства в одной VLAN видят друг друга, а в разных VLAN не видят.
- Настроить маршрутизацию между VLAN через подинтерфейсы маршрутизатора (802.1Q).

## Инструменты

- Cisco Packet Tracer (версия: _укажите вашу_)
- Коммутаторы Cisco 2960, маршрутизатор Cisco 2911

## Топология

![Топология](images/topology.png)

| Устройство | Порт | VLAN | IP-адрес | Шлюз |
|---|---|---|---|---|
| PC1 → SW1 | Fa0/1 | 10 (SALES) | 192.168.10.11/24 | 192.168.10.1 |
| PC2 → SW1 | Fa0/2 | 20 (IT) | 192.168.20.11/24 | 192.168.20.1 |
| PC3 → SW2 | Fa0/1 | 10 (SALES) | 192.168.10.12/24 | 192.168.10.1 |
| PC4 → SW2 | Fa0/2 | 20 (IT) | 192.168.20.12/24 | 192.168.20.1 |
| SW1 ↔ SW2 | Gi0/1 | trunk (10, 20) | — | — |
| SW1 ↔ R1 | Fa0/3 → Gi0/0 | trunk | — | — |

## Конфигурация

### SW1

```
enable
configure terminal
hostname SW1
vlan 10
 name SALES
vlan 20
 name IT
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface gi0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
interface fa0/3
 switchport mode trunk
end
write memory
```

### SW2

```
enable
configure terminal
hostname SW2
vlan 10
 name SALES
vlan 20
 name IT
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface gi0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
end
write memory
```

### R1 (Router-on-a-stick)

```
enable
configure terminal
hostname R1
interface gi0/0
 no shutdown
interface gi0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface gi0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
end
write memory
```

## Проверка

### 1. VLAN и trunk на коммутаторах

Команды: `show vlan brief` и `show interfaces trunk`.

![show vlan brief](images/show-vlan-brief.png)
![show interfaces trunk](images/show-trunk.png)

### 2. Связь внутри одной VLAN (до настройки маршрутизатора)

`ping 192.168.10.12` с PC1: успешно, трафик идёт через trunk между SW1 и SW2.

![Ping внутри VLAN 10](images/ping-same-vlan.png)

### 3. Изоляция между VLAN (до настройки маршрутизатора)

`ping 192.168.20.11` с PC1: не проходит, так как устройства в разных VLAN.

![Ping между VLAN до маршрутизатора](images/ping-different-vlan-fail.png)

### 4. Маршрутизация между VLAN (после настройки R1)

`ping 192.168.20.11` с PC1: успешно, трафик идёт через подинтерфейсы R1.

![Ping между VLAN через маршрутизатор](images/ping-different-vlan-ok.png)

## Результаты

- VLAN 10 и 20 созданы на обоих коммутаторах, порты распределены по VLAN.
- Trunk между SW1 и SW2 передаёт трафик VLAN 10 и 20.
- Устройства в одной VLAN видят друг друга через оба коммутатора.
- Без маршрутизатора устройства в разных VLAN изолированы.
- После настройки подинтерфейсов на R1 связь между VLAN работает.

## Что я узнал

- Как работают access- и trunk-порты и зачем нужна разметка 802.1Q.
- Как VLAN разделяют широковещательные домены на одном физическом оборудовании.
- Как маршрутизатор с подинтерфейсами связывает VLAN между собой.

## Файлы

- `lab01-vlan.pkt`: файл Packet Tracer с готовой топологией
- `images/`: скриншоты топологии и проверок

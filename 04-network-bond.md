# 04. Сеть: Bond LACP + Jumbo Frames

[← Назад к оглавлению](README.md)

## 1. Создание Bond через `nmtui`

```bash
sudo nmtui
```
Edit a connection → Add → Bond:

Поле	Значение
Profile name	bond0-lacp
Device	bond0
Slaves (Ports)	eno5, eno6
Mode	802.3ad (LACP)
Link monitoring	MII (recommended)
Monitoring frequency	100 ms
IPv4	<Manual>
Addresses	<IP-адрес-сервера>/<маска>
⚠️ На коммутаторе порты eno5 и eno6 должны быть в одном LAG с LACP.

### 2. Проверка профилей
```bash
nmcli -f NAME,DEVICE,TYPE connection show
```
Ожидаемый результат:

text
NAME              DEVICE   TYPE
bond0-lacp        bond0    bond
bond-slave1       eno5     ethernet
bond-slave2       eno6     ethernet

### 3. Настройка MTU 9000
В nmtui нет поля MTU — используем nmcli:

```bash
sudo nmcli connection modify bond0-lacp 802-3-ethernet.mtu 9000
sudo nmcli connection modify bond-slave1 802-3-ethernet.mtu 9000
sudo nmcli connection modify bond-slave2 802-3-ethernet.mtu 9000
sudo nmcli connection up bond0-lacp
```

⚠️ MTU 9000 нужен и на bond, и на slave'ах. Иначе ядро откажется поднимать bond.

### 4. Проверка
```bash
ip link show bond0 | grep mtu
ip link show eno5  | grep mtu
ip link show eno6  | grep mtu

cat /proc/net/bonding/bond0
```

Ожидаемые поля:

Bonding Mode: IEEE 802.3ad Dynamic link aggregation
MII Status: up
LACP active: on
Number of ports: 2

Partner Key: <N> (не 0)

### 5. Тест Jumbo Frames
```bash
ping -M do -s 8972 -c 3 <IP-адрес-цели>
```
Результат	Значение
✅ Reply	MTU 9000 работает end-to-end
❌ Message too long	Где-то на пути MTU 1500
❌ Timeout	Партнёр не отвечает
⚠️ Jumbo Frames работают только если все устройства на пути (NIC, коммутатор, ESXi) настроены на MTU 9000.

### 6. Частые проблемы
Бонд не поднимается:

Проверьте LACP на коммутаторе
```bash
cat /proc/net/bonding/bond0 покажет, какой slave не up
```
MTU 9000 не применяется:

Убедитесь, что установлен на всех slave-интерфейсах

Проверьте, не перезаписывает ли NetworkManager

Далее: Пересборка RAID →

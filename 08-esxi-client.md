### 08. Подключение со стороны VMware ESXi

### 1. Включение iSCSI-инициатора

vSphere Client:

- Host → Configure → Storage Adapters → iSCSI Software Adapter → Enable

CLI:
```bash
esxcli iscsi software set --enabled=true
```

### 2. Добавление портала
vSphere Client:

Host → Configure → Storage Adapters → iSCSI → Dynamic Discovery → Add

Address: <IP-адрес-сервера>:3260

CLI:

```bash
# Узнать имя адаптера (обычно vmhbaXX)
esxcli iscsi adapter list

esxcli iscsi adapter discovery sendtarget add \
    -a <adapter> \
    -A <IP-адрес-сервера>:3260
```

### 3. Пересканирование
```bash
esxcli storage core adapter rescan --all
```

### 4. Создание VMFS
vSphere Client:

Storage → New Datastore → VMFS → выбрать устройство → задать имя

CLI:

```bash
# Найти NAA-идентификатор (полный, вида naa.6001405...)
esxcli storage core device list | grep -i naa

# Создать VMFS (подставить полный NAA-идентификатор)
esxcli storage vmfs volume create -l datastore-sas1 -d naa.<полный-идентификатор>
```

### 5. Multipath I/O (MPIO)
Важно: ESXi не включает multipath автоматически при добавлении второго портала в Dynamic Discovery. Для программного iSCSI-адаптера необходимо настроить Port Binding (привязку VMkernel-адаптеров).

**5.1. Подготовка на стороне сервера (target)**
Добавить второй IP на бонд:
```bash
sudo nmcli connection modify bond0-lacp +ipv4.addresses <IP-2>/28
sudo nmcli connection up bond0-lacp
```
Добавить второй портал в targetcli:
```bash
sudo targetcli /iscsi/iqn.../tpg1/portals create <IP-2> 3260
sudo targetcli saveconfig
```
**5.2. Настройка на стороне ESXi**

Сценарий А: оба IP-адреса таргета в одной подсети с ESXi
Создайте два VMkernel-адаптера (например, vmk1 и vmk2) в той же подсети, что и порталы таргета.
В vSphere Client: Host → Configure → Storage Adapters → iSCSI Software Adapter → Network Port Binding → добавьте оба VMkernel-адаптера.
Добавьте оба портала в Dynamic Discovery.
Пересканируйте: esxcli storage core adapter rescan --all.
Проверьте пути: esxcli storage core path list — должно быть несколько путей в состоянии Active к одному устройству.

Сценарий Б: IP-адреса таргета в разных подсетях
Port Binding не используется. Вместо этого настройте статические маршруты через нужные VMkernel-адаптеры:

```bash
esxcli network ip route ipv4 add --gateway <шлюз> --network <подсеть-таргета>
```
Затем добавьте порталы в Dynamic Discovery и пересканируйте.

**5.3. Проверка MPIO**
```bash
# Список путей к устройству
esxcli storage core path list

# Политика MPIO (по умолчанию Round Robin для iSCSI)
esxcli storage nmp device list
```
### 6. Проверка на ESXi
```bash
# Список iSCSI-устройств
esxcli storage core device list | grep -A5 naa

# Состояние путей
esxcli storage core path list

# Список VMFS
esxcli storage vmfs extent list
```
### 7. Производительность
Проверьте скорость на датасторе:

```bash
# На ESXi (SSH)
cd /vmfs/volumes/<datastore>
dd if=/dev/zero of=test.bin bs=1M count=10240 conv=fdatasync
rm test.bin
```
Ориентировочные показатели (зависят от RAID, сети и нагрузки; при MTU 9000 и 10GbE):

SSD датастор: 400+ МБ/с

SAS датастор: 300+ МБ/с

Для сети 1GbE потолок составит ~110–120 МБ/с на один путь.

Далее: Мониторинг →
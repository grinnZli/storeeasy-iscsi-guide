# 08. Подключение со стороны VMware ESXi

[← Назад к оглавлению](README.md)

## 1. Включение iSCSI-инициатора

**vSphere Client:**
- Host → Configure → Storage Adapters → iSCSI Software Adapter → **Enable**

**CLI:**
```bash
esxcli iscsi software set --enabled=true
2. Добавление портала
vSphere Client:

Host → Configure → Storage Adapters → iSCSI → Dynamic Discovery → Add

Address: <IP-адрес-сервера>:3260

CLI:

bash
esxcli iscsi adapter discovery sendtarget add \
    -a <adapter> \
    -A <IP-адрес-сервера>:3260
3. Пересканирование
bash
esxcli storage core adapter rescan --all
4. Создание VMFS
vSphere Client:

Storage → New Datastore → VMFS → выбрать устройство → задать имя

CLI:

bash
# Найти NAA-идентификатор
esxcli storage core device list | grep -i naa

# Создать VMFS
esxcli storage vmfs volume create -l datastore-sas1 -d naa.<xxx>
5. Multipath I/O (опционально)
Добавить второй IP на бонд
bash
sudo nmcli connection modify bond0-lacp +ipv4.addresses <IP-2>/28
sudo nmcli connection up bond0-lacp
Добавить второй портал в targetcli
bash
sudo targetcli /iscsi/iqn.../tpg1/portals create <IP-2> 3260
sudo targetcli saveconfig
На ESXi
Добавьте второй портал в Dynamic Discovery. ESXi настроит multipath автоматически.

6. Проверка на ESXi
bash
# Список iSCSI-устройств
esxcli storage core device list | grep -A5 naa

# Состояние путей
esxcli storage core path list

# Список VMFS
esxcli storage vmfs extent list
7. Производительность
Проверьте скорость на датасторе:

bash
# На ESXi (SSH)
cd /vmfs/volumes/<datastore>
dd if=/dev/zero of=test.bin bs=1M count=10240 conv=fdatasync
rm test.bin
Ожидаемые показатели (при MTU 9000):

SSD датастор: 400+ МБ/с

SAS датастор: 300+ МБ/с

Далее: Мониторинг →
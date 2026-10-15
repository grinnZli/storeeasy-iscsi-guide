
# 11. Приложения

[← Назад к оглавлению](README.md)

## A. Схема хранения
┌─────────────────────────────────────────────────────┐
│ HPE Smart Array P816i-a SR Gen10 (Embedded) │
│ │
│ ┌──────────────────┐ ┌──────────────────┐ │
│ │ Array A │ │ Array B │ │
│ │ RAID 10 │ │ RAID 10 │ │
│ │ 4× SSD 480 ГБ │ │ 8× SAS HDD 2.4 ТБ│ │
│ │ = 894 ГБ │ │ = 8.73 ТБ │ │
│ └────────┬─────────┘ └────────┬─────────┘ │
│ │ │ │
│ /dev/sdc /dev/sdd │
└───────────┼─────────────────────────┼───────────────┘
│ │
▼ ▼
┌──────────┐ ┌──────────┐
│ vg_ssd │ │ vg_sas │
│ 894 ГБ │ │ 8.73 ТБ │
└────┬─────┘ └────┬─────┘
│ │
▼ ▼
┌──────────┐ ┌──────────────┐
│ lun_ssd1 │ │ lun_sas1 │
│ 750 ГБ │ │ lun_sas2 │
└────┬─────┘ │ (4 ТБ + 4 ТБ)│
│ └──────┬───────┘
│ │
└──────────┬───────────┘
▼
┌────────────────┐
│ iSCSI Target │
│ iqn.2026-10... │
│ :vmware │
│ 192.168.200.2 │
└────────┬───────┘
│
┌────────┴────────┐
▼ ▼
┌──────────┐ ┌──────────┐
│ ESXi-1 │ │ ESXi-2 │
│ (ACL) │ │ (ACL) │
└──────────┘ └──────────┘

text

## B. Итоговая конфигурация

| Компонент | Значение |
|---|---|
| Сервер | HPE StoreEasy 1860 (DL380 Gen10) |
| ОС | AlmaLinux 9 |
| Система | mdadm RAID 1 (2× Micron 240 ГБ) |
| Хранилище 1 | P816i-a Array A — RAID 10, 4× SSD 480 ГБ, 894 ГБ |
| Хранилище 2 | P816i-a Array B — RAID 10, 8× SAS HDD 2.4 ТБ, 8.73 ТБ |
| LVM | `vg_ssd`, `vg_sas` |
| LUN'ы | `lun_ssd1` (750 ГБ), `lun_sas1` (4 ТБ), `lun_sas2` (4 ТБ) |
| Сеть | `bond0` (802.3ad), MTU 9000 |
| IP | `<IP-адрес-сервера>/28` |
| iSCSI | targetcli, порт 3260 |
| Клиенты | 2× VMware ESXi с ACL |

## C. Быстрые команды

### Сеть
```bash
nmcli -f NAME,DEVICE,TYPE connection show
ip -br a
cat /proc/net/bonding/bond0
RAID
bash
sudo ssacli ctrl all show config
sudo ssacli ctrl slot=0 ld all show
sudo ssacli ctrl slot=0 pd all show
LVM
bash
sudo pvs && sudo vgs && sudo lvs
iSCSI
bash
sudo targetcli ls
sudo targetcli sessions list
sudo ss -tlnp | grep 3260
D. Полезные ссылки
AlmaLinux

HPE MCP репозиторий

targetcli документация

VMware iSCSI guide

E. Лицензия
MIT. Используйте на свой страх и риск.

F. Версия
Версия: 1.0

Дата: 2026-10-14

Автор: grinnzli
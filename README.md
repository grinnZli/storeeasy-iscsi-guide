# HPE StoreEasy 1860 (DL380 Gen10) → AlmaLinux 9 → iSCSI для VMware ESXi

Полное руководство по превращению штатного NAS **HPE StoreEasy 1860** в iSCSI-хранилище для кластера **VMware ESXi** на базе **AlmaLinux 9**.

## Оглавление

| #  | Раздел | Описание |
|----|--------|----------|
| 01 | [Исходное состояние](01-hardware.md) | Характеристики сервера, дисков, сети |
| 02 | [Установка AlmaLinux 9](02-install-almalinux.md) | Подготовка, разметка, установка |
| 03 | [Утилиты HPE](03-hpe-tools.md) | `ssacli`, `amsd`, связь с iLO |
| 04 | [Сеть: Bond + Jumbo Frames](04-network-bond.md) | LACP-бонд, MTU 9000 |
| 05 | [Пересборка RAID](05-raid-rebuild.md) | Смена RAID-уровня, проверка |
| 06 | [Настройка LVM](06-lvm.md) | PV, VG, LV |
| 07 | [Настройка iSCSI-таргета](07-iscsi-target.md) | `targetcli`, backstores, ACL, CHAP |
| 08 | [Подключение ESXi](08-esxi-client.md) | iSCSI-инициатор, VMFS, MPIO / Port Binding |
| 09 | [Мониторинг](09-monitoring.md) | RAID, iSCSI, скрипты |
| 10 | [Troubleshooting](10-troubleshooting.md) | Частые проблемы |
| 11 | [Приложения](11-appendix.md) | Схемы, команды, конфиги |

## Важное предупреждение

Замена штатной ОС (Windows Storage Server IoT 2019) на неподдерживаемую производителем (AlmaLinux 9) **лишает вас гарантийной поддержки HPE**. Используйте эту инструкцию только для:

- лабораторий,
- тестовых стендов,
- сценариев, где поддержка HPE не критична.

## Что в итоге получится

- **Сервер:** HPE StoreEasy 1860 (DL380 Gen10) на AlmaLinux 9
- **Хранилище:**
  - 894 ГБ SSD (RAID 10) → 750 ГБ LUN
  - 8.73 ТБ SAS (RAID 10) → 2 × 4 ТБ LUN
- **Сеть:** LACP-бонд, MTU 9000 (jumbo frames)
- **iSCSI-таргет** для 2+ хостов VMware ESXi с ACL и (опционально) CHAP

## Требования

- Сервер HPE DL380 Gen10 / StoreEasy 1860
- iLO 5 с доступом
- AlmaLinux 9 DVD ISO
- Коммутатор с поддержкой LACP и jumbo frames
- Кластер VMware ESXi (опционально — для базового подключения достаточно одного хоста; для проверки MPIO желательно два VMkernel-адаптера или два хоста)

## Используемые обозначения

- `<IP-адрес-сервера>` — IP-адрес бонда
- `<IQN-ESXi-N>` — IQN инициатора ESXi
- `<adapter>` — имя iSCSI-адаптера ESXi (например, `vmhba65`)
- `<hostname>` — имя хоста
- `<UUID-...>` — UUID раздела

## Безопасность и резервное копирование

- Для production-среды рекомендуется включить **CHAP-аутентификацию** (см. раздел 07).
- После изменения конфигурации таргета выполняйте `targetcli saveconfig` — конфигурация сохраняется в `/etc/target/saveconfig.json`. Рекомендуется хранить резервную копию этого файла.

## Лицензия

Этот проект распространяется под лицензией MIT. Подробности в файле [LICENSE](LICENSE).

## Благодарности

HPE, AlmaLinux Foundation, сообщество LIO/Datera, сообщество Linux.
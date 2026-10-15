# 01. Исходное состояние оборудования

[< Назад к оглавлению](README.md)

## Сервер

| Компонент | Значение |
|---|---|
| Модель | HPE StoreEasy 1860 (DL380 Gen10) |
| CPU | Intel Xeon Scalable |
| RAM | ?32 ГБ |
| iLO | iLO 5, прошивка актуальная |
| BIOS | UEFI, актуальная версия |
| RAID | P816i-a SR Gen10 (Embedded) + P408e-p SR Gen10 (Slot 1) |

## Дисковая конфигурация (исходная)

### Array A — SSD RAID 10
- 4? SATA SSD, 480 ГБ
- Порты: `1I:1:13` – `1I:1:16`
- Итог: **894 ГБ**

### Array B — SAS RAID 60 (до пересборки)
- 8? SAS HDD, 2.4 ТБ
- Порты: `1I:1:1` – `1I:1:8`
- Итог: **8.73 ТБ**

### Системный массив
- 2? SATA SSD Micron 240 ГБ
- Программный RAID 1 (`mdadm`)

## Сетевая конфигурация

| Интерфейс | Назначение |
|---|---|
| `eno1`, `eno2` | Management (1 Гбит/с) |
| `eno3`, `eno4` | Свободны (link down) |
| `eno5`, `eno6` | Bond LACP (10G/25G) |

## Что было сделано с железом

1. Штатная Windows Storage Server удалена
2. Установлена AlmaLinux 9
3. Системные диски оставлены в mdadm RAID 1
4. Оба массива данных пересобраны через `ssacli`
5. Настроен LACP-бонд и jumbo frames
6. Поверх RAID создан LVM
7. Поверх LVM развёрнут iSCSI-таргет

[Далее: Установка AlmaLinux >](02-install-almalinux.md)
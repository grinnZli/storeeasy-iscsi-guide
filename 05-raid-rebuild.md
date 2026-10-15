# 05. Пересборка RAID-массивов

[← Назад к оглавлению](README.md)

## 1. Инвентаризация

```bash
sudo ssacli ctrl slot=0 pd all show
12 дисков:

1I:1:1 – 1I:1:8 — SAS HDD 2.4 ТБ

1I:1:13 – 1I:1:16 — SATA SSD 480 ГБ

2. Смена RAID-уровня (опционально)
⚠️ ВНИМАНИЕ: Удаление Logical Drive приводит к полной потере данных на массиве.

Удаление старого массива
bash
# Посмотреть LD
sudo ssacli ctrl slot=0 ld all show

# Удалить LD 2 (RAID 60 из SAS)
sudo ssacli ctrl slot=0 ld 2 delete forced
Создание нового RAID 10
bash
sudo ssacli ctrl slot=0 create type=ld \
    drives=1I:1:1-1I:1:8 \
    raid=1+0 \
    ss=128
Параметр ss=128 — strip size 128 КБ (оптимально для iSCSI/БД).

3. Проверка
bash
sudo ssacli ctrl slot=0 ld all show
Ожидаемый результат:

text
Array A
  Logical Drive: 1
    Size: 894.20 GB
    Fault Tolerance: 1+0
    Status: OK

Array B
  Logical Drive: 2
    Size: 8.73 TB
    Fault Tolerance: 1+0
    Status: OK
4. Идентификация в Linux
bash
lsblk -o NAME,SIZE,TYPE,MODEL,SERIAL
sudo blkid
⚠️ Имена /dev/sdX нестабильны. Используйте UUID или LABEL.

5. Сравнение RAID-уровней
Параметр	RAID 60	RAID 10
Ёмкость (8× 2.4 ТБ)	8.73 ТБ (6 дисков)	8.73 ТБ (4 диска)
Отказоустойчивость	2 диска / пара	1 диск / пара
Запись	Средняя	Высокая
Чтение	Хорошая	Отличная
Для чего	Архивы	БД, VM, iSCSI
Далее: LVM →

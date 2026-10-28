# 02. Установка AlmaLinux 9

[← Назад к оглавлению](README.md)

## 1. Подготовка загрузочного носителя

Скачайте **AlmaLinux 9 DVD ISO** (~14 ГБ): https://almalinux.org/

Запишите образ на USB (≥16 ГБ):
```bash
sudo dd if=AlmaLinux-9.x-x86_64-dvd.iso of=/dev/sdX bs=4M status=progress
sync
```

💡 Альтернатива: Minimal ISO (~1.5 ГБ) — быстрее, но требует интернета при установке.

### 2. Настройка RAID-контроллера до установки
Через iLO Remote Console → Intelligent Provisioning → Smart Storage Administrator:

Проверьте, что массивы Array A и Array B созданы

Убедитесь, что все диски в статусе OK

Запишите конфигурацию

### 3. Разметка дисков
Используйте программный RAID 1 (mdadm) на двух системных SSD Micron (223 ГБ каждый):

Раздел	Размер	ФС	Точка монтирования
/boot/efi	600 МБ	vfat	/boot/efi
/boot	1 ГБ	xfs	/boot
/	70 ГБ	xfs	/
/var	85 ГБ	xfs	/var
/home	50 ГБ	xfs	/home
swap	16 ГБ	swap	—

### 4. Сетевые настройки при установке
На этапе установки не настраивайте bond — оставьте дефолтный интерфейс (eno1 или eno2). Bond будет настроен позже.

### 5. Hostname и пользователь
Hostname: alma-storage

Пользователь: создайте root и рабочего пользователя

### 6. Проверка установки
```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
cat /proc/mdstat
```

Ожидаемый результат:
text
md127 : active raid1 sda1[0] sdb1[1]
md122 : active raid1 sda2[0] sdb2[1]

### 7. Первичное обновление

```bash
sudo dnf update -y
sudo reboot
```

Далее: Драйверы и утилиты HPE →
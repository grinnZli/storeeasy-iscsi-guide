
## 📄 `06-lvm.md`

```markdown
# 06. Настройка LVM

[← Назад к оглавлению](README.md)

## 1. Очистка подписей

```bash
sudo wipefs -a /dev/sdc   # SSD массив (894 ГБ)
sudo wipefs -a /dev/sdd   # SAS массив (8.7 ТБ)
2. Создание PV
bash
sudo pvcreate /dev/sdc /dev/sdd
3. Создание VG
bash
sudo vgcreate vg_ssd /dev/sdc
sudo vgcreate vg_sas /dev/sdd
4. Проверка
bash
sudo vgs
sudo pvs
Ожидаемый результат:

text
VG      #PV #LV #SN Attr   VSize   VFree
vg_sas    1   2   0 wz--n- 8.73t   <750.34g
vg_ssd    1   1   0 wz--n- <894.20g <144.20g
5. Создание LV
bash
sudo lvcreate -L 750G -n lun_ssd1 vg_ssd

sudo lvcreate -L 4T -n lun_sas1 vg_sas
sudo lvcreate -L 4T -n lun_sas2 vg_sas
💡 Если появилось xfs signature detected ... Wipe it? [y/n] — отвечайте y. Для iSCSI LUN не должно быть ФС на стороне сервера.

6. Проверка
bash
sudo lvs
Ожидаемый результат:

text
LV        VG      Attr       LSize
lun_sas1  vg_sas  -wi-a----- 4.00t
lun_sas2  vg_sas  -wi-a----- 4.00t
lun_ssd1  vg_ssd  -wi-a----- 750.00g
7. Схема
text
RAID 10 (SSD)             RAID 10 (SAS)
/dev/sdc                  /dev/sdd
    │                         │
    ▼                         ▼
  vg_ssd                    vg_sas
    │                         │
    ▼                    ┌────┴────┐
  lun_ssd1               ▼         ▼
  750 ГБ              lun_sas1  lun_sas2
                      4 ТБ      4 ТБ
Далее: iSCSI-таргет →

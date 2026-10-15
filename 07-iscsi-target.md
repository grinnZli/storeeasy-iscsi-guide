# 07. Настройка iSCSI-таргета

[← Назад к оглавлению](README.md)

## 1. Установка targetcli

```bash
sudo dnf install -y targetcli
sudo systemctl enable --now target
sudo firewall-cmd --permanent --add-port=3260/tcp
sudo firewall-cmd --reload
⚠️ Если зеркала AlmaLinux отдают 502 — см. Troubleshooting.

2. Создание backstores
bash
sudo targetcli /backstores/block create name=sas1 dev=/dev/vg_sas/lun_sas1
sudo targetcli /backstores/block create name=sas2 dev=/dev/vg_sas/lun_sas2
sudo targetcli /backstores/block create name=ssd1 dev=/dev/vg_ssd/lun_ssd1
3. Создание iSCSI-таргета
bash
sudo targetcli /iscsi create iqn.2026-10.local.<hostname>:vmware
4. Привязка LUN'ов
bash
sudo targetcli /iscsi/iqn.2026-10.local.<hostname>:vmware/tpg1/luns create /backstores/block/sas1
sudo targetcli /iscsi/iqn.2026-10.local.<hostname>:vmware/tpg1/luns create /backstores/block/sas2
sudo targetcli /iscsi/iqn.2026-10.local.<hostname>:vmware/tpg1/luns create /backstores/block/ssd1
5. Настройка портала
bash
# Удалить дефолтный 0.0.0.0
sudo targetcli /iscsi/iqn.../tpg1/portals delete 0.0.0.0 3260

# Привязать к IP бонда
sudo targetcli /iscsi/iqn.../tpg1/portals create <IP-адрес-сервера> 3260
6. Настройка ACL
bash
sudo targetcli /iscsi/iqn.../tpg1/acls create <IQN-ESXi-1>
sudo targetcli /iscsi/iqn.../tpg1/acls create <IQN-ESXi-2>
Отключение аутентификации (лаборатория)
bash
sudo targetcli /iscsi/iqn.../tpg1 set attribute authentication=0
sudo targetcli /iscsi/iqn.../tpg1 set attribute demo_mode_write_protect=0
sudo targetcli /iscsi/iqn.../tpg1 set attribute generate_node_acls=1
CHAP (продакшен)
bash
sudo targetcli /iscsi/iqn.../tpg1 set attribute authentication=1
sudo targetcli /iscsi/iqn.../tpg1/acls/<IQN-ESXi-1> set auth userid=<user>
sudo targetcli /iscsi/iqn.../tpg1/acls/<IQN-ESXi-1> set auth password=<password>
7. Сохранение конфигурации
bash
sudo targetcli saveconfig
sudo cp /etc/target/saveconfig.json /root/target-saveconfig-$(date +%F).json
8. Проверка
bash
sudo targetcli ls
sudo ss -tlnp | grep 3260
sudo targetcli sessions list
9. (Опционально) write-back кэш
Если батарея Smart Array в статусе OK:

bash
sudo ssacli ctrl slot=0 show detail | grep -i -E 'cache|battery|capacitor'

sudo targetcli /backstores/block/sas1 set attribute emulate_write_cache=1
sudo targetcli /backstores/block/sas2 set attribute emulate_write_cache=1
sudo targetcli /backstores/block/ssd1 set attribute emulate_write_cache=1
sudo targetcli saveconfig
Далее: Подключение ESXi →
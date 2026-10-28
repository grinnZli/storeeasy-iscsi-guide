# 09. Мониторинг и обслуживание

[← Назад к оглавлению](README.md)

### 1. RAID

```bash
# Статус логических дисков
sudo ssacli ctrl slot=0 ld all show status

# Статус физических дисков
sudo ssacli ctrl slot=0 pd all show status

# Детали диска
sudo ssacli ctrl slot=0 pd 1I:1:1 show detail

# Baseline конфигурации
sudo ssacli ctrl all show config > ~/raid-config-$(date +%F).txt
sudo ssacli ctrl slot=0 pd all show detail > ~/disk-inventory-$(date +%F).txt
```

### 2. iSCSI
```bash
sudo targetcli sessions list
sudo ss -tnp | grep 3260
sudo journalctl -u target -f
```

### 3. LVM
```bash
sudo pvs
sudo vgs
sudo lvs
sudo vgdisplay vg_sas
```
###4. Сеть
```bash
cat /proc/net/bonding/bond0
ip -br a
ip -s link show bond0
```
### 5. Скрипт мониторинга RAID
/usr/local/bin/raid-check.sh:

```bash
#!/bin/bash
STATUS=$(ssacli ctrl slot=0 ld all show status 2>/dev/null | grep -v 'OK')
if [ -n "$STATUS" ]; then
    echo "RAID WARNING on $(hostname):" | mail -s "RAID Alert" root
    echo "$STATUS" | mail -s "RAID Alert" root
fi
```

```bash
sudo chmod +x /usr/local/bin/raid-check.sh
echo "0 * * * * root /usr/local/bin/raid-check.sh" | sudo tee /etc/cron.d/raid-check
```
### 6. Email-оповещения от RAID
```bash
sudo ssacli ctrl slot=0 modify alertlevel=on
sudo ssacli ctrl slot=0 modify alertemail=<email-админ>
sudo ssacli ctrl slot=0 modify alerthostname=<hostname>
```
###7. SMART на дисках за RAID
```bash
sudo dnf install -y smartmontools
```
### Device ID берётся из ssacli pd ... show detail
```bash
sudo smartctl -i -d sat+megaraid,13 /dev/sdc
sudo smartctl -i -d sat+megaraid,1  /dev/sdd
```
### 8. Полезные команды
Команда	Что показывает
```bash
ssacli ctrl all show config	Вся конфигурация контроллеров
ssacli ctrl slot=0 ld all show	Логические диски
ssacli ctrl slot=0 pd all show	Физические диски
ssacli ctrl slot=0 show detail	Кэш, батарея контроллера
targetcli ls	Вся конфигурация iSCSI
targetcli sessions list	Активные сессии
```
Далее: Troubleshooting →

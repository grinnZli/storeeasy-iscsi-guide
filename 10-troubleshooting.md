### 10. Troubleshooting

[← Назад к оглавлению](README.md)

### AMS = `Not available` в iLO

** Причина:** 
`amsd` не работает или сессия с iLO зависла.

** Решение:**
```bash
sudo systemctl status amsd
sudo systemctl restart smad amsd
```
Если не помогло — сбросьте iLO (Information → Diagnostics → Reset iLO) и подождите 2–3 минуты.

### hpilo: Open could not dequeue a packet

** Причина:** 
Известная проблема совместимости hpilo ↔ iLO.

** Решение:**
Обновить amsd до 2.4.1+
Обновить прошивку iLO до актуальной
Сбросить iLO

### 502 от зеркал AlmaLinux
** Причина: **
Временные проблемы зеркал.

** Решение: **
добавить репозиторий Яндекса:

```bash
sudo tee /etc/yum.repos.d/yandex-almalinux.repo <<'EOF'
[yandex-baseos]
name=AlmaLinux $releasever - BaseOS (Yandex)
baseurl=https://mirror.yandex.ru/almalinux/$releasever/BaseOS/$basearch/os/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9

[yandex-appstream]
name=AlmaLinux $releasever - AppStream (Yandex)
baseurl=https://mirror.yandex.ru/almalinux/$releasever/AppStream/$basearch/os/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9

[yandex-extras]
name=AlmaLinux $releasever - Extras (Yandex)
baseurl=https://mirror.yandex.ru/almalinux/$releasever/extras/$basearch/os/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9

[yandex-crb]
name=AlmaLinux $releasever - CRB (Yandex)
baseurl=https://mirror.yandex.ru/almalinux/$releasever/CRB/$basearch/os/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9
EOF

sudo dnf clean all && sudo dnf makecache
```

### targetcli saveconfig: Command not found

** Решение:**
Выйти из интерактивной оболочки (exit)

Выполнить
```bash 
sudo targetcli saveconfig
```
Или проверить /etc/target/saveconfig.json — конфигурация может сохраняться автоматически.

### iSCSI-клиент не видит LUN'ы
** Решение ** 
Проверьте по порядку:
Firewall: 
```bash 
sudo firewall-cmd --list-ports | grep 3260
```
Служба: 
```bash
systemctl status target
```

Портал: 
```bash
sudo targetcli ls /iscsi/.../tpg1/portals
```
ACL: 
```bash
sudo targetcli ls /iscsi/.../tpg1/acls
```
LUN'ы: 
```bash 
sudo targetcli ls /iscsi/.../tpg1/luns
```
MTU 9000 на клиенте

MTU 9000 не работает end-to-end
Проверьте:
Коммутатор: порты с jumbo frames (mtu 9216 / mtu 9000)
ESXi: vSwitch и vmkernel с MTU 9000
Тест: ping -M do -s 8972 с обеих сторон

### mount -a не находит UUID

** Причина:** 
UUID вставлен вместе с угловыми скобками < > из документации.

**Решение: **
Уберите < и >:

text
UUID=b8db36f8-...  /mnt/ssd-data  xfs  defaults,noatime,nodiratime  0 0
Проверка:

```bash
sudo findmnt --verify
```
### Бонд не поднимается
** Решение ** 
LACP на коммутаторе
MTU на всех slave'ах
```bash
cat /proc/net/bonding/bond0
```

### Медленная работа iSCSI
** Возможные причины:**
MTU 1500 вместо 9000
write-thru вместо write-back
Один путь (нет MPIO)
Перегруженный RAID
Проверьте:

```bash
# Кэш контроллера
sudo ssacli ctrl slot=0 show detail | grep -i cache

# iSCSI-сессии
sudo targetcli sessions list
```

Далее: Приложения →

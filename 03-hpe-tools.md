## `03-hpe-tools.md`

# 03. Драйверы и утилиты HPE

[← Назад к оглавлению](README.md)

## 1. Проверка драйвера RAID-контроллера

```bash
sudo dnf install -y pciutils
lspci -nnk | grep -i -A3 -E 'raid|storage|scsi|sas'
```

Ожидаемый результат:

text
11:00.0 Serial Attached SCSI controller [0107]: Adaptec Smart Storage PQI SAS [9005:028f]
    Subsystem: Hewlett-Packard Company Smart Array P408e-p SR Gen10 [103c:0601]
    Kernel driver in use: smartpqi
    Kernel modules: smartpqi
b1:00.0 Serial Attached SCSI controller [0107]: Adaptec Smart Storage PQI SAS [9005:028f]
    Subsystem: Hewlett-Packard Company Smart Array P816i-a SR Gen10 [103c:1100]
    Kernel driver in use: smartpqi
    Kernel modules: smartpqi
Драйвер smartpqi уже в ядре AlmaLinux 9 — установка не требуется.

### 2. Подключение репозитория HPE MCP
```bash
sudo tee /etc/yum.repos.d/mcp.repo <<'EOF'
[mcp]
name=Management Component Pack
baseurl=https://downloads.linux.hpe.com/SDR/repo/mcp/Alma/9/x86_64/current/
enabled=1
gpgcheck=0
EOF

sudo dnf clean all
sudo dnf makecache
```

### 3. Установка утилит управления
```bash
sudo dnf install -y ssacli amsd
```
Пакет	Назначение
ssacli	Управление Smart Array (RAID)
amsd	Agentless Management Service (iLO)

### 4. Настройка связи iLO ↔ ОС
```bash
sudo systemctl enable --now amsd
systemctl status amsd
```
Проверка в iLO
Откройте веб-интерфейс iLO → System Information → Summary:

Пункт	Ожидаемый статус
Agentless Management Service	✅ OK
Network	✅ OK
BIOS/Hardware Health	✅ OK
Storage	✅ OK
Если AMS = Not available
Перезапустите службы:

```bash
sudo systemctl restart smad amsd
```

Если не помогло — сбросьте iLO:

iLO → Information → Diagnostics → Reset iLO

Подождите 2–3 минуты.

### 5. Проверка RAID-конфигурации
```bash
sudo ssacli ctrl all show config
sudo ssacli ctrl slot=0 ld all show
sudo ssacli ctrl slot=0 pd all show
```
Все диски должны быть в статусе OK.

Далее: Сеть Bond + Jumbo →

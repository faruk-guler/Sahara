# Modül 03: Wazuh Agent Kurulumu ve Yönetimi

Wazuh Agent, uç noktaları izlemek için kullanılan temel uç noktadır.

## Kurulum Yöntemleri

### 1. Linux (Ubuntu/Debian)

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
apt-get update
WAZUH_MANAGER='192.168.1.100' apt-get install wazuh-agent
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

### 2. Windows (PowerShell)

```powershell
invoke-webrequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.2-1.msi -OutFile ${env:tmp}\wazuh-agent.msi; msiexec.exe /i ${env:tmp}\wazuh-agent.msi /q WAZUH_MANAGER='192.168.1.100'
```

## Agent Kayıt (Registration)

Agent'lar yöneticiye kendilerini iki şekilde tanıtabilir:

- **Password-based:** `authd` üzerinden şifre ile.
- **Certificate-based:** SSL sertifikası ile.

### Agent Durum Kontrolü (Server Tarafında)

```bash
/var/ossec/bin/agent_control -l
```

## Agent Konfigürasyonu (`ossec.conf`)

Agent tarafındaki ayarlar `/var/ossec/etc/ossec.conf` (Linux) veya `C:\Program Files (x86)\ossec-agent\ossec.conf` (Windows) dosyasından yönetilir.

- `<client>`: Manager IP ve Port bilgileri.
- `<syscheck>`: FIM ayarları.
- `<rootcheck>`: Rootkit tarama ayarları.

## Merkezi Yapılandırma (Agent Groups)

Binlerce ajanı tek tek yönetmek yerine, Manager üzerinde gruplar oluşturabilir ve `/var/ossec/etc/shared/agent.conf` dosyası ile ortak ayarlar basabilirsiniz.

```bash
# Ajanı bir gruba atama
/var/ossec/bin/agent_groups -a -i 001 -g web_servers
```

## Manuel Kayıt (CLI)

Eğer otomatik kayıt (authd) kapalıysa, ajan tarafında manuel kayıt yapılabilir:

```bash
/var/ossec/bin/agent-auth -m 192.168.1.100 -P root_sifresi
```

---
[README'ye Dön](README.md)

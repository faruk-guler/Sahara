# Zabbix Master Cheat Sheet (2026)

Bu doküman, Zabbix yönetiminde en sık kullanılan komutları ve konfigürasyon detaylarını içerir.

## 🛠️ Servis Yönetimi

### Zabbix Server

```bash
systemctl restart zabbix-server
systemctl status zabbix-server
tail -f /var/log/zabbix/zabbix_server.log
```

### Zabbix Agent 2

```bash
systemctl restart zabbix-agent2
zabbix_agent2 -t "system.cpu.load"
zabbix_agent2 -p
```

## 📂 Önemli Dosya Yolları

- **Server Konfig:** `/etc/zabbix/zabbix_server.conf`
- **Agent 2 Konfig:** `/etc/zabbix/zabbix_agent2.conf`
- **Frontend Konfig:** `/etc/zabbix/web/zabbix.conf.php`
- **Loglar:** `/var/log/zabbix/` (Server), `/var/log/httpd/error_log` (Apache)

## ⌨️ Zabbix Get & Sender

- **Metric Çekme:** `zabbix_get -s 192.168.1.10 -k "system.cpu.load"`
- **Metric Gönderme:** `zabbix_sender -z 127.0.0.1 -s "Host" -k "traps" -o "1"`

## 🏆 HA Cluster & Status (7.0+)

- **HA Durumu:** `zabbix_server -R ha_status`
- **Failover Tetikle:** `zabbix_server -R ha_remove_node=node_name`
- **Database Status:** `SELECT * FROM timescaledb_information.hypertables;`

## ♻️ Cache ve Senkronizasyon (Runtime Control)

- **Server Config Yenile:** `zabbix_server -R config_cache_reload`
- **Proxy Config Yenile:** `zabbix_proxy -R config_cache_reload`
- **Log Seviyesi Artır:** `zabbix_server -R log_level_increase="poller"`

## 🔒 Otomasyon & Güvenlik (2026)

- **API Token Sorgusu:** `curl -X POST -H 'Authorization: Bearer <TOKEN>' -d '{"jsonrpc":"2.0","method":"host.get","id":1}' http://127.0.0.1/api_jsonrpc.php`
- **Vault Token Testi:** `curl -H "X-Vault-Token: $VAULT_TOKEN" https://vault.local:8200/v1/secret/data/zabbix`

---
[README'ye Dön](./README.md)

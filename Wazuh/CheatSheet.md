# Wazuh Master Series: Cheat Sheet (Hızlı Referans)

En sık kullanılan dosya yolları, komutlar ve konfigürasyonlar.

## Dosya Yolları (Paths)

| Bileşen | Dosya Yolu | Açıklama |
| :--- | :--- | :--- |
| **Konfigürasyon** | `/var/ossec/etc/ossec.conf` | Ana ayarlar |
| **Log Dosyası** | `/var/ossec/logs/ossec.log` | Hata ve işlem logları |
| **Alarmer** | `/var/ossec/logs/alerts/alerts.json` | Üretilen tüm alarmler |
| **Özel Kurallar** | `/var/ossec/etc/rules/local_rules.xml` | Kendi kurallarınız |
| **Özel Dekoderlar** | `/var/ossec/etc/decoders/local_decoder.xml` | Kendi dekoderlarınız |

## Servis Komutları

### Linux (Systemd)

- `systemctl restart wazuh-manager`
- `systemctl status wazuh-agent`
- `systemctl restart wazuh-dashboard`

### Windows (Powershell)

- `Restart-Service -Name Wazuh`
- `Get-Service -Name Wazuh`

## Faydalı Komutlar

- **Logları Canlı İzleme:** `tail -f /var/ossec/logs/alerts/alerts.json | jq`
- **Agent Listesi:** `/var/ossec/bin/agent_control -l`
- **Agent Detayı:** `/var/ossec/bin/agent_control -i 001`
- **Kural Test Etme:** `/var/ossec/bin/wazuh-logtest`
- **API Testi:** `curl -u user:pass -k https://localhost:55000`
- **API Token:** `curl -u user:pass -k -X POST https://localhost:55000/security/user/authenticate?raw=true`

## Kural Seviye Rehberi

- **Seviye 3:** Çok düşük (Bilgilendirme)
- **Seviye 5-7:** Orta (Giriş hataları vb.)
- **Seviye 10-12:** Yüksek (Kritik hatalar)
- **Seviye 15:** Çok Yüksek (Aktif saldırılar)

---

[README'ye Dön](README.md)

# Modül 09: Log Verisi Toplama ve Analizi

Wazuh'un temel işlevi, farklı kaynaklardan log toplamak ve bunları merkezi bir yerde analiz etmektir.

## Log Kaynakları

### 1. Yerel Log Dosyaları (Local Files)

Sistemdeki düz metin (text) loglarını izler.

```xml
<localfile>
  <location>/var/log/nginx/access.log</location>
  <log_format>syslog</log_format>
</localfile>
```

### 2. Windows Event Logs

Windows servis ve sistem loglarını okur.

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

### 3. Komut Çıktıları (Command Output)

Belirli bir komutun çıktısını log olarak kabul eder.

```xml
<localfile>
  <log_format>full_command</log_format>
  <command>netstat -tulpn</command>
  <frequency>360</frequency>
</localfile>
```

### 4. Uzak Loglama (Syslog)

Ajan kurulamayan cihazlardan (firewall, switch vb.) logları 514 portu (UDP/TCP) üzerinden alır.

## Log Analiz Süreci

1. **Pre-decoding:** Logun geldiği tarih, hostname ve program adı ayrıştırılır.
2. **Decoding:** Log içeriğindeki spesifik alanlar (IP, User) Regex ile çıkarılır.
3. **Rule Matching:** Çıkarılan alanlar kurallarla karşılaştırılır.
4. **Alerting:** Kural eşleşirse alarm seviyesine göre Indexer'a yazılır.

## Arşivleme vs Alarmlar

Wazuh Manager varsayılan olarak sadece alarm üreten olayları saklar.

- **alerts.json:** Sadece kuralla eşleşen ve alarm seviyesi yeterli olan olaylar.
- **archives.json:** (Opsiyonel) Gelen tüm ham loglar. Yasal zorunluluklar için açılabilir.

```xml
<ossec_config>
  <global>
    <logall>yes</logall> <!-- Ham logları archives.json'a yaz -->
    <logall_json>yes</logall_json>
  </global>
</ossec_config>
```

## Çok Satırlı Loglar (Multi-line Logs)

Java stack trace gibi loglar için `out_format` veya özel dekoder ayarları gerekebilir.

---

[README'ye Dön](README.md)

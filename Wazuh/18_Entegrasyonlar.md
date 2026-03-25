# Modül 18: Entegrasyonlar (Webhook, Slack, Jira)

Wazuh alarmlerini Manager dışına aktararak (External Integration) anlık bildirimler alabilirsiniz.

## 1. Hazır Entegrasyonlar

Wazuh şu araçlarla doğrudan (native) konuşabilir:

- **Slack:** Belirli seviyedeki alarmleri Slack kanalına gönderir.
- **PagerDuty:** Olay müdahale süreçlerini tetikler.
- **Jira:** Kritik alarmler için otomatik ticket açar.
- **Email:** Geleneksel SMTP bildirimleri.

## 2. Slack Entegrasyonu Kurulumu

Manager üzerindeki `ossec.conf` dosyasına:

```xml
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/YOUR/WEBHOOK</hook_url>
  <level>10</level> <!-- 10 ve üzeri alarmleri gönder -->
  <alert_format>json</alert_format>
</integration>
```

## 3. Özel Script Entegrasyonu (Webhook)

Eğer kendi uygulamanıza veri göndermek istiyorsanız, bir Python scripti yazarak integrasyonu özelleştirebilirsiniz. Betik Manager üzerindeki `/var/ossec/integrations/` dizininde bulunmalıdır.

## 4. VirusTotal Entegrasyonu

FIM tarafından algılanan yeni bir dosyanın hash değerini otomatik olarak VirusTotal'e sorabilir ve zararlıysa alarm üretebilirsiniz.

```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_VT_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
```

## Entegrasyon Servisi

External integration'lar Manager üzerindeki `ossec-integratord` servisi tarafından yönetilir.

## Hata Ayıklama (Debug)

Eğer bildirimler gitmiyorsa, `/var/ossec/logs/integrations.log` dosyasını kontrol ederek hatanın kaynağını (Örn: geçersiz API key, network hatası) görebilirsiniz.

---

[README'ye Dön](README.md)

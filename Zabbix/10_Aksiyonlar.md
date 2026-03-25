# Modül 10: Aksiyonlar (Actions), Bildirimler ve Otonom İyileştirme

Sensör (Item) ölçümü yaptı, Trigger (Tetikleyici) eşiği aşınca kızardı. Peki sonra ne olacak? Sadece web arayüzünde "Kırmızı" bir bar görmek sistemi yönetmeye yetmez. Çözümü uygulayan ya da sizi haberdar eden kısım **Action (Aksiyon)**'dır.

## 1. Otonom Hata Giderme (Remote Commands / Auto-Remediation)

2026 yılı Sysadmin standartlarında, gece 03:00'te çöken bir Nginx servisi için insana SMS atılmaz. Zabbix'in bunu kendi kendine düzeltmesi istenir.

1. `Template App Nginx` içindeki süreç izlemesi (proc.num[nginx]) eğer 0'a düşerse `Nginx is down` trigger'ı üretilir.
2. Yazdığımız bir `Action`, bu trigger'ı tespit eder. Olay (Operation) eylemi olarak `Send Message` yerine **`Remote Command`** seçilir.
3. Hedef Sistem (Target): Current Host (Problemin yaşandığı zombi sunucu).
4. Tür: `Custom script` -> `systemctl restart nginx`

**Güvenlik Engeli Yıkımı:** 
Eskiden Ajanlar güvenlik sebebiyle root seviyesinde yetkisiz komut çalıştıramazlardı. 
1. Zabbix Agent 2 ayar dosyasında `AllowKey=system.run[*]` kilidi açılmalıdır.
2. Zabbix Unix kullanıcısına (zabbix objesi), ajan makinesinde `/etc/sudoers.d/zabbix` içinden Nginx restart yetkisi tanınmalıdır:
`zabbix ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx`

Dev devasa bir veri merkezinde çöken IIS veya Apache havuzlarının tamamı %90 Zabbix Otonom İyileştirme ile otomatik ayağa kaldırılıyor olmalıdır.

## 2. Escalations (Seviyeli Tırmandırma Mimarisi)

Problemi (Diyelim ki Disk tamamen doldu ve veritabanı çöktü) Zabbix'in uzaktan çözemeyeceğini varsayalım. O zaman insana haber vermelidir. Fakat kime? Mimarisine göre escalation yazılır.

Her Step (Adım) süresini örneğin 1 saat (`1h`) belirlediniz:

- **Adım 1:** Sadece Operasyon (L1 Destek) Ekibine Telemetri e-postası veya Slack mesajı fırlat.
- **Adım 2 (1 Saat Sonra hala Kırmızıysa):** Problemi Senior Veritabanı Yöneticisine (DBA) SMS ve Jira Ticket olarak at.
- **Adım 3 (2 Saat Sonra):** Gece yarısı nöbetçiden yanıt yoksa ve problem sürüyorsa, CTO (Teknoloji Müdürü) ve Veri Merkezi Yöneticisine doğrudan acil "Call (Arama) PagerDuty" Alarmı gönder.

## 3. Webhook (Entegrasyon) Kullanarak Bildirim Paylaşımı

2026'da hiç kimse SMTP sunucusundan e-posta beklemiyor. Tüm BT dünyası Slack, Telegram, MS Teams, ve Discord etrafında dönüyor.

Zabbix'in **Media Types** menüsünde yer alan yerleşik Webhook altyapısı doğrudan JavaScript çalıştırır. Zabbix'i, Microsoft Teams'in sunduğu Webhook adresine (POST URL) şu şekilde entegre edebilirsiniz:

```javascript
// Örnek Bir MS Teams veya Slack Payload Gönderimi (Media Type)
var req = new HttpRequest();
req.addHeader('Content-Type: application/json');

var payload = {
    title: "ZABBİX ALARM: " + params.event_name,
    text: "Mevcut Durum: " + params.event_status + "\nSunucu IP: " + params.host_ip,
    color: (params.event_value == 1) ? "#FF0000" : "#00FF00" // Kırmızı: Sorun var, Yeşil: Çözüldü
};

req.post(params.webhook_url, JSON.stringify(payload));
```
Böylelikle, BT ekibi Teams veya Slack üzerinde çalışırken odaya düşen renkli (Resolved olduğunda yeşile dönen interaktif) Zabbix kartlarını görerek anında aksiyon alır.

---
[Önceki Modül](./09_Triggerlar.md) | [README'ye Dön](./README.md) | [Sonraki Modül: SLA ve Servisler](./11_SLA_Servisler.md)

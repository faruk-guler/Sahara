# Modül 21: Kurumsal PDF Raporlama ve Zabbix Web Service Mimarisi

C-Level (Yönetim Kurulu), Şube Müdürleri veya Finans birimleri Zabbix paneline asla girmez, Gözlemlenebilirlik grafiklerine göz atmaz ve SLA panelleri onlara hiçbir şey ifade etmez. İstedikleri tek şey; her Pazartesi Sabah 08:30'da E-posta adreslerine düşen şık (Logolu), temiz, basitleştirilmiş bir **"Sistem Sağlık Özet (PDF) Raporu"** almaktır.

Eskiden Python ile Selenium scriptleri yazılarak ekran fotoğrafı çekilip manuel maillenen bu yapı; 2026'da Zabbix'in yerleşik PDF motoruyla standartlaşmıştır.

## 1. Mimaride Yeni Bileşen: Zabbix Web Service

Zabbix Server programlama dili olarak (C) HTML/JavaScript yorumlama veya PDF render etme yeteneğine doğal olarak sahip değildir. Ekranda gördüğünüz güzel Dashboard'lar (Tarayıcı tabanlıdır, İstemcide/Sizde render edilir).

Zabbix bu HTML tablolarını PDF'e çevirmek için **Google Chrome** veya **Chromium** motoruna ihtiyaç duyar:

- Harici ve C# / Go dilleriyle sıfırdan yapılmış **`zabbix-web-service`** paketi ortama yüklenir.
- Bu servis arkada Chromium tarayıcısını "Headless" (Ekransız) modda çalıştırır.

### Başlarken (Konfigürasyon Etkileşimi)

`zabbix_server.conf` dosyasında Zabbix ile Web Servis'in tanıştırılması şarttır:

```ini
StartReportWriters=3
WebServiceURL=http://localhost:10053/report
```

*Note: Sunucu 3 Adet (Tavsiye edilen başlangıç) Yazar Süreci (Writer Thread) üretir ve PDF oluşturma işlemlerini asenkron olarak WebService portuna fırlatıp saniye tasarrufu (Performans) yapar.*

## 2. Zamanlanmış Rapor (Scheduled Reports) Yapılandırması

Rapor üretecini devreye aldıktan sonra işin Arayüz kısmı kalır (`Reports` menüsü -> `Scheduled reports`).

1. **Rapor Çerçevesi (Dashboard):** Seçilen Dashboard neyse Zabbix onun ekran resmini anlık alıp sayfalarca PDF çıkartır. C-Level Raporlama Dashboard'unda "Honeycomb" veya Yüzlerce "Top Hosts" listesi olamaz. İçerik sadece dev *SLA Yüzdeleri*, *Aylık Biten Harddisk Miktar Tablosu* ve 1-2 Adet sade Pasta Grafik'ten oluşmalı, bu temiz tasarlanmış "Executive Dashboard" klasörü rapora zimmetlenmelidir.
2. **Kullanıcı Hedefi (User/User Groups):** Rapor kimlere (CEO_Group) atılacak?
3. **Peryodik (Takvim):** Daily (Günlük), Weekly (Haftalık) Pazartesi Saat 08:00 vb.

## 3. Sıklıkla Yapılan 2 Master Hatası (Kritik Nokta)

- **Hatırlatma 1 (`Frontend URL` Sorunu):** Zabbix'e "Admin, Site_URL" adı altında kim olduğunu belirtmeniz hayati önem taşır. `Administration` -> `General` -> `Other` ayarlarında "Frontend URL" bölümü boş kalmamalı (Örn: `https://zabbix.sirket.com`). Boş bırakırsanız Zabbix Web Service Chrome üzerinden neye bağlanıp da ekran görüntüsü çekeceğini (DNS'ini) bilemez ve Rapora boş PDF'ler gelir veya PDF oluşturma süreci (TimeOut) iptal olur.
- **Hatırlatma 2 (Yetki Sorunu):** Rapor motorunun sahibi (Report Owner/Generator user) olarak, Dashboard üzerindeki grafikleri göremeyen, Cihazların Host grubuna yetkisi olmayan bir NOC personeli seçerseniz, rapordaki görsellere veriler yansımaz, "No permissions" pencerelerinden ibaret bir rapor CEO'ya ulaşır! Raporu daima geniş yetkili API Token / Scheduled User profiliyle tetikleyin.

## 4. Alternatif: Grafana Export Çözümleri

Mimaride Zabbix UI yerine %100 Grafana ekosistemi kullanan firmalar (Modül 13), Raporlamayı da Grafana Image Renderer (Veya Grafana Enterprise PDF eklentisi) üzerinden kurgulayabilirler. İki platformun raporları da mantık olarak birebir Headless Chrome üzerine temellenir. Olay tamamen görselliğin nerede duracağına karar vermektir.

---
[Önceki Modül](./20_Guvenlik_Hardening.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Zabbix Master Performans Tuning](./22_Performans_Tuning.md)

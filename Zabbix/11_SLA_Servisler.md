# Modül 11: Zabbix ITIL, SLA, ve BSM (İş Hizmeti Yönetimi)

Bir CTO'nun (Teknoloji Yöneticisinin) umrunda olan şey "Veritabanının portu kapanmış" veya "Switchin RAM'i şişmiş" değildir. Onun umrunda olan: **"Müşterilerimiz E-Ticaret sitemizden alışveriş yapabiliyor mu?"** sorusudur.

Zabbix'teki "Services" (Eskiden IT Services, v6.0 ile yeniden tasarlandı, v7.0 ile güçlendirildi) menüsü tam olarak bu Business Service Management (BSM) vizyonu etrafında tasarlanmıştır.

## 1. Ağaç Mimarisi (Service Tree) Doğrulaması

BSM yaklaşımıyla en tepede (Root) firmanızın sattığı Ana Hizmet durur: **"Online Alışveriş Platformu"**.
Platformun çatısı altında 3 ana sütun (dal) vardır:
- Web Ön Yüzü (Nginx ve CDN)
- Ödeme API'leri (Banka entegrasyonu)
- Veritabanı Kümesi (Master-Slave PostgreSQL)
- Arkaplan Sistemleri (Active Directory, DNS, Ağ Bağlantısı)

### Parent-Child Bağıntıları:
DNS sunucusunun Down (Kapalı) olması, Web ön yüzüne erişimi bozar. Zabbix'te bunları birbirine Parent/Child olarak eklersiniz.

*   Her düğüme bir **"Weight" (Ağırlık/Önem Derecesi)** atanır. Core Switch'in Weight'i 100 iken, Test Sunucusununki 1'dir.
*   **Root Cause Analysis (Kök Neden Analizi):** E-Ticaret Ana Servisi çöktüğünde Zabbix otomatik olarak alt dalları tarar, problemi Veritabanı dalından aşağı kadar indirir ve CTO'nun ekranına şunu çıkartır: "E-Ticaret Sitesi Çöktü -> Kök Neden: Oracle Switch-31 Enerji Kesintisi".

## 2. Status Calculation Rule (Mantıksal Durum Hesaplama)

Hizmetlerin (Services) hangi koşullarda çökük kabul edileceğini belirleyen esnek matematiktir.

- **Veritabanı Kümesi Örneği (Clustering):** Altyapıda 3 adet DB sunucusu var ve Load Balancer üzerinden hizmet veriyorlar. Bu durumda "Ödeme API Veritabanları" isimli Ana Servisin durumunu `Kısmi Çökme` veya `Kritik Hata` yapması için;
  - *Kural Seçimi:* At least `2` child services have `Disaster` status. 
  - *Mantık:* "Eğer sadece 1 DB çökerse hizmeti kırmızı yapma (Çünkü diğer ikisi yükü kaldırmaya devam ediyor). Ancak 2 tane DB aynı anda çökerse İşletme risk altındadır, servisi Kızart."

## 3. SLA (Service Level Agreement) Yönetimi ve Raporlama

Şirketin IT departmanı "Yıllık %99.9" çalışma sözü (SLA Hedefi) verdi diyelim.

Zabbix Services menüsü altında bir **SLA Sözleşmesi** tanımlanır.
- **Schedule (Çalışma Takvimi):** Destek ekibinin sözleşmesi sadece hafta içi 08:00 - 18:00 arası (Business Rules) ise SLA sadece bu takvimi sayar. Cumartesi gece çöken sunucu Uptime (Hizmet süresi) istatistiğini düşürmez!
- **Downtime Calculation:** Planlı bakımlar (Bkz. Modül 17) SLA kaybından çıkartılır. Ekibin yüzdesi düşmez.
- Zabbix bu periyotta Uptime oranının 99.8'e (%99.9 hedefinin altına) düştüğünü fark ederse, o ay ceza yenmemesi için Yöneticilere `Service Actions` üzerinden SMS gönderir.

### 4. Service Actions (Servis Aksiyonları)
Standart Triggerların (Modül 10) dışında, tüm bu BSM mantığı kendi bildirim mekanizmasına (`Service Actions`) sahiptir.
Örnek kullanım:
"E-Ticaret Platformu SLA'i %98'e düştüyse CTO'ya E-posta fırlat. Switch-A CPU Triggerı çalıştıysa L1 Ekibine Telegram at." 

İşte bu yüzden, şirket üst kademesine verilecek izleme ekranı Hostlara değil, yalnızca **Servislere (Services)** yönlendirilmelidir.

---
[Önceki Modül](./10_Aksiyonlar.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Proxy ve Dağıtık Yapı](./12_Zabbix_Proxy.md)

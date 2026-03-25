# Modül 01: Zabbix'e Giriş ve 2026 Observability Evrimi

Bu modül, Zabbix'in temel felsefesini, mimari vizyonunu ve nasıl sadece bir "Monitoring" aracı olmaktan çıkıp "Observability" (Gözlemlenebilirlik) ekosistemine dönüştüğünü incelemektedir.

## 1. Monitoring vs Observability (İzleme ve Gözlemlenebilirlik)

Geleneksel "Monitoring" yaklaşımında sistemlerin **"Up" (Açık)** veya **"Down" (Kapalı)** olduğu kontrol edilir. CPU %90 olduysa alarm üretilir. 
Ancak modern 2026 altyapılarında (K8s, Mikroservisler, Serverless) bu yaklaşım yetersizdir.

**Observability (Gözlemlenebilirlik):**
- Sistemin "Neden" hata verdiğini log (kayıt), metric (ölçüm) ve trace (iz) (Three Pillars of Observability) verilerini birleştirerek analiz etme sanatıdır.
- Zabbix 7.0+ ile birlikte bu üç sütunun tamamına (özellikle OpenTelemetry ile trace desteği gelerek) hakimiyet kurmuştur.

## 2. Zabbix vs Prometheus & Grafana

Çoğu modern mimaride "Zabbix mi, Prometheus mu?" sorusu sorulmaktadır. Gerçekte her iki araç da farklı mimari yaklaşımlara sahiptir:

| Özellik | Zabbix (2026) | Prometheus |
| :--- | :--- | :--- |
| **Mimari Model** | Push (Active Agent) ve Pull | Sadece Pull (Scrape) |
| **Veri Tutma (Retention)** | Trendler ile yıllarca (Long-Term, TimescaleDB) | Kısa Süreli Bellek İçi (TSDB) |
| **Problem Analizi** | Triggerlar, Mantıksal İfadeler, SLA Yönetimi | PromQL Alertmanager |
| **Görselleştirme** | Kendi yerleşik Dashboard'ları (Honeycomb, Geomap) | Grafana'ya bağımlılık |
| **Uygunluk** | Windows, Network, Legacy donanım, Veri Merkezleri | Cloud-Native, Podlar, Ephemeral (Geçici) Ortamlar |

*Not: Zabbix 2026'da doğrudan Prometheus Data Source üzerinden de veri okuyabilir (Prometheus to Zabbix).*

## 3. 2026 (v7.0/v8.0) ile Gelen Kritik Yenilikler

Geleceğin altyapısını yönetmek için Zabbix'in sunduğu "Game Changer" (Oyun Değiştirici) özellikler şunlardır:

### 3.1. Proxy High Availability (Yüksek Erişilebilirlik)
Eskiden on binlerce cihazı izleyen bir Zabbix Proxy çökerse, sistem köre düşerdi. Artık **Proxy Load Balancing** sayesinde:
- Proksiler bir "Group" içine alınır.
- Proxy-A çökerse, yük anında Proxy-B'ye aktarılır. Bütün hostlar otomatik failover yaşar.

### 3.2. AIOps ve Makine Öğrenimi (Machine Learning)
Zabbix, gelen statik datayı sadece threshold'a (eşik değeri) göre vurmaz. `trendstl()` fonksiyonu ile **Seasonality (Mevsimsellik)** hesabı yapar.
- *Örnek:* Bir sunucunun CPU'su her sabah 09:00'da yoğunlaşıyorsa, Zabbix bunu "Normal" kabul eder. Ancak gece 03:00'te aniden yükselirse "Anomaly" (Anormallik) alarmı üretir.

### 3.3. Browser Monitoring (Web Senaryoları)
Geleneksel HTTP izleme (Sitenin 200 OK dönmesi) yeterli değildir. Müşterinin siteye girip ürün sepete atıp atamadığını test etmeniz gerekir.
- Zabbix artık **Playwright** motorunu kullanır.
- Tarayıcıyı Headless (Arayüzsüz) modda çalıştırır, JavaScript yükler, butonlara tıklar ve süreci uçtan uca milisaniye bazında raporlar.

### 3.4. Native MFA (Multi-Factor Authentication)
VPN olmadan Zabbix arayüzüne dışarıdan erişim vermek intihardır. Zabbix artık yerleşik olarak:
- **TOTP (Time-based One-Time Password):** Google Authenticator, Authy vb.
- **Duo:** Push notification tabanlı çift aşamalı doğrulamayı tek tıkla destekler.

## 4. Sistem Mühendisleri İçin Gözlemlenebilirlik Mantalitesi

Binlerce alarm oluşturmak "İyi İzleme" yaptığınızı göstermez; aksine **Alert Fatigue (Alarm Yorgunluğu)** yaratır. L2/L3 ekipler bir süre sonra gelen uyarıları ciddiye almamaya başlar. 
Zabbix'te temel felsefe:
> "Sistemde bir arıza çıktığında alarm ver; ancak o arıza işletmeyi etkilemiyorsa kimseyi gece 03:00'te uyandırma!"

Modül 09 (Triggerlar) ve Modül 11 (BSM) bu sorunu çözmek için tasarlanmıştır.

---
[README'ye Dön](./README.md) | [Sonraki Modül: Modern Zabbix Mimarisi](./02_Mimari.md)

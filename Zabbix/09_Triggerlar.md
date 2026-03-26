# Modül 09: Modern Trigger ve Anomali Tespiti (AIOps)

Zabbix'te veriyi topladıktan sonra, "Bu veri normal mi?" veya "Problem teşkil ediyor mu?" sorusunu soran, sistem yöneticilerini SMS veya e-posta ile ayağa kaldıran yapı **Trigger (Tetikleyici)** mekanizmasıdır. Triggerlar doğrudan Item'lara (ölçüm değerlerine) bağlıdır.

## 1. Mimaride Trigger İfade (Expression) Yapısı

Yeni nesil (v5.4 ve sonrası) Zabbix sürümleri daha yetenekli ve matematikteki fonksiyon çağrılarına benzeyen bir syntax (Söz dizimi) üretir. 

Eski sürüm syntax: `{Linux_Server:system.cpu.load[all,avg1].last()} > 5`
**2026 Sürümü Syntax:** `last(/Linux_Server/system.cpu.load[all,avg1]) > 5`

### 1.1 En Sık Kullanılan Genel Fonksiyonlar
*   **`last()`:** Tam olarak anlık, alınan en son veri.
*   **`avg(5m)`, `max(15m)`, `min(1h)`:** Dalgalanmaları engellemek. Bir ping yanıtı anlık 2000ms olduysa alarm çalsın istemeyiz. "*Son 5 dakikadaki* ortalama ping 1000'den büyükse alarm çal" `avg(/Server/icmppingsec,5m) > 1` mantığı false-positive (yanıltıcı) alarmları önler.
*   **`nodata(10m)`:** Veri akışının durduğunu (Zabbix Agent'ın koptuğunu veya sunucunun yandığını) anlar. `nodata(/Server/system.cpu.load,10m) = 1` şeklinde çalışır.
*   **`diff()` veya `change()`:** Verideki anormal değişimi anlar (Örn: Cihaz konfigürasyon dosyası MD5 checksum hash'i değiştiyse).

## 2. AIOps, Forecasting ve Makine Öğrenimi Sensörleri

Modern altyapıda, disk %90 olduğunda alarm göndermek gelenekselleşmiştir. Trend analizleri ile bunu gelecekte haber vermek (Predictive Analytics) Zabbix'i güçlü kılar.

### 2.1 Predictive (Tahminsel) Fonksiyonlar
- **`timeleft()`:** "/ (Root) Diskimiz bu performansla yazılmaya devam ederse kaç saate veya güne dolar (tamamen tükenir)?" 
  - Kural örneği: "Eğer diskin dolmasına kalan süre (timeleft) 24 saatten az ise alarm ver." Böylece sorun sistem yöneticisine gece değil de, mesai saatinde önceden "Diskte yer bitecek" uyarısı verir.
- **`forecast()`:** Eğilim (lineer, kök veya logaritmik uydurma). "Bu Pazar gecesi veritabanı yükü CPU % kaçlara dayanacak?" sorusuna karşılık hesaplama yapar ve belirlenen eşiği aşacağı tahmini çıkarsa alarm (Warning) çalar.

### 2.2 Anomali Fonksiyonları (Seasonality)
- **`trendstl()`:** Makine öğrenimine benzer bir fonksiyondur (Seasonal-Trend Decomposition using Loess). Her Cuma akşamüstü e-ticaret sitenizin işlem kapasitesinin 5 kat artması Zabbix için artık bir anomali (alarm) değildir çünkü rutindir. Ancak işlem Salı sabahı aynı orana fırlarsa `trendstl` anında alarm mekanizmasını tetikler.

## 3. Tag (Etiket) Sistemi: Event Correlation

Her olay (trigger) belirli etiketlerle donatılmalıdır (Tagging). 
*Örnek Etiketler:*
- `Service: MySQL`
- `Target: Production`
- `Team: Database Admins`

Eskiden (Eski Zabbix'lerde) alarm, gruplara göre (Linux Sunucuları grubu) gönderilirdi. Artık alarmlar (Action'lar) tamamen Taglara göre yönlendirilir (Modül 10). "İçinde Team=Database Admins" tag'ı geçen bir trigger tetiklenirse bunu DB ekibine at.

## 4. Trigger Dependency (Alarm Bağımlılığı - Master Seviyesi)

**Senaryo:** Ankara Veri Merkezinde 500 Tane ESXi, Router ve Storage var. Ankara'daki Core Switch'in enerjisi giderse Zabbix Data Center'a ulaşamayacak.

Eğer önlem almazsanız; önce Switch Down alarmı, sonra art arda ESXi-1, ESXi-2... Node-500 toplamda +500 adet arka arkaya "Makineye Erişilemiyor (Agent Unreachable) - Disaster" SMS'i Telefonunuza gelir. (Alert Storm - Alarm Fırtınası).

**Çözüm:** Bağlı (dependent) trigger kurgusudur. 
Tüm o 500 cihazın "Agent Unreachable" triggerları açılır ve `Dependencies` (Bağımlılıklar) alanından Core Switch'teki "ICMP Ping Down" trigger'ına bağlanır.

**Sonuç:** Core Switch çöktüğü zaman sistem tek 1 tane SMS atacaktır: "Core Switch Down". Geri kalan 500 makinenin durumunun ne olduğu artık Zabbix için ikincil kalır.

---
[Önceki Modül](./08_Veri_Toplama.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Aksiyonlar](./10_Aksiyonlar.md)

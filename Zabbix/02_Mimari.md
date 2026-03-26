# Modül 02: Modern Zabbix Mimarisi Derinlemesine Bakış

Zabbix'in devasa ölçekte (saniyede yüzbinlerce veri - NVPS) çalışabilmesi, mikroservislere benzer şekilde kendi içinde bölümlendirilmiş "Process (Düğüm)" yapısından kaynaklanır.

## 1. Mimarinin Ana Kalbi: Zabbix Server

Herkesin tek bir servis sandığı `zabbix_server`, aslında arka planda onlarca farklı alt iş parçacığıdır (thread). Sadece bu alt süreçleri anlamak sistem performansını (tuning) yönetmenin %80'idir:

### ⚙️ Kritik Zabbix İşletim Süreçleri (Processes):

*   **Poller:** Pasif (Passive) ajanlara, SNMP cihazlarına veya JMX servislerine gidip "Bana metriklerini ver" diyen işçilerdir. Sisteminizde çok fazla pasif host varsa, Poller yetersiz kalıp darboğaz yapar.
*   **Trapper:** Aktif (Active) ajanların, Zabbix Sender verilerinin ve Zabbix Proxy'lerden gelen paketlerin karşılandığı porttur (10051). Dinleyici (Listener) görevi görür.
*   **History Syncer:** Veritabanına (DB) yazma işleminden sorumludur. NVPS (New Values Per Second) arttıkça en çok RAM (`HistoryCacheSize`) kullanan bileşendir.
*   **Timer:** Zaman tabanlı fonksiyonların (`nodata`, `timeleft`) hesaplanmasını yapan süreçtir.
*   **Escalator:** Modül 10'da anlatılan Aksiyonların (Slack'e mesaj at, SSH ile bağlan) yürütülmesini sağlar.

## 2. Zabbix Server HA Cluster (Native High Availability)

Zabbix v7.0 öncesi Sunucu çöktüğünde HA (Yüksek Erişilebilirlik) sağlamak için Pacemaker, Corosync gibi 3. parti kompleks araçlar kurulurdu.
2026 itibariyle **Native HA Cluster** zorunluluktur.

### Nasıl Çalışır?
- Minimum 2 adet Zabbix Sunucu kurulur. İkisi de aynı veritabanına bağlanır.
- `zabbix_server.conf` dosyasında `HANodeName=Node1` ve `NodeAddress` tanımlanır.
- Sunuculardan sadece biri **"Active"** modda çalışır (Tüm polling ve db işlemlerini yapar).
- Diğeri tamamen uyku modundadır (**"Standby"**). 
- Active sunucudan ping/heartbeat (genellikle 5 saniyede bir) alınamazsa, Standby sunucu otomatik olarak Active pozisyonuna geçer ve IP (veya DNS) üzerinden işlemleri devralır.

## 3. Zabbix Veritabanı (Zabbix DB)

Zabbix, veriyi Relational (İlişkisel) DB'lerde tutar. Ancak 2026 standardı PostgreSQL + **TimescaleDB** eklentisidir.

### 🗄️ History ve Trends Ayrımı (Kritik Kavram)
- **History (Tarihçe):** Sensörlerden saniyelik/dakikalık gelen en saf (raw) veridir. Milyarlarca satır yer kaplar. Zabbix'te best-practice olarak bu veriden sadece 7 gün veya maksimum 30 gün saklanması önerilir.
- **Trends (Eğilim):** Saf verilerin saatlik ortalamasıdır (Min, Max, Avg olarak sıkıştırılır). History silinse bile 1 yıllık, 5 yıllık verileri çok az yer kaplayarak bu sayede görebilirsiniz.
  
*Zabbix performansının altın kuralı: Tarihçe(History) verilerini kısa tut, Trends verilerini uzun tut.*

## 4. Zabbix Proxy (Dağıtık Mimari ve Load Balancing)

Binlerce sunuculu ortamlarda veya uzak veri merkezlerinde Zabbix Server'ın her makineye ayrı port açması güvenlik felaketidir ve ping gecikmesi yaratır. Bu işi Proxy çözer.

### Proxy Türleri
*   **Active Proxy (Önerilen):** Proxy kendi Network'ündeki (Örn: Frankfurt Veri Merkezi) binlerce cihazdan veriyi toplar, sıkıştırır (Zlib compression) ve tek bir güvenli hat üzerinden Server'a gönderir (Push). Server hiçbir şekilde Proxy'e port açmak zorunda kalmaz.
*   **Passive Proxy:** Server'ın Proxy'ye bağlanıp "Ne topladın anlat" dediği, genellikle tercih edilmeyen yöntemdir.

### Offline Buffer (Veri Saklama)
Eğer Proxy ile Server arasındaki WAN (internet) hattı koparsa, Proxy topladığı verileri kendi SQLite3 veritabanında saklamaya başlar (`ProxyOfflineBuffer=72` - 72 saat). İnternet geldiğinde milyonlarca satırı saniyeler içinde Server'a topluca iletir. Zabbix'te kör uç yoktur.

## 5. Zabbix Agent (Active vs Passive)

Ajanların çalışma mantığı Mimarinin performansını temelden sarsabilir:
- **Passive Agent:** Sunucu, cihaza bağlanır (Port 10050) ve bekler. CPU yükü Sunucu (Server) üzerindedir. *Ölçeklenemez.*
- **Active Agent:** Cihaz konfigürasyonunu Server'dan alır ve kendi CPU'sunu kullanarak metrikleri düzenli aralıklarla Server'a "postalar" (Port 10051). *Milyonlarca hosta ölçeklenebilen tek yöntemdir.*

---
[Önceki Modül](./01_Giris.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Kurulum ve DB](./03_Kurulum.md)

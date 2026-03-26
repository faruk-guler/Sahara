# Modül 13: Dinamik Dashboardlar ve Görselleştirme (Data Visualization) Şöleni

İzleme yapıyorsunuz, veriler kusursuz akıyor ama C-Level yöneticiler Excel gibi metin (Text) satırlarına veya ham grafiklere (Classic Graph) bakmazlar. Gözlemlenebilirlik satır aralarında yatar, başarılı Sistem Yönetimi ise onu en süslü ve anlaşılır şekilde ekrana boyamak (Görselleştirme) ile ölçülür.

## 1. Zabbix 7.0/8.0 Yeni Nesil Master Widgetlar

Eski tip "Sıcaklık 45, CPU %90" veren Text barlarına ek olarak, büyük veri setlerini dar alanlara (NOC ekranı - Network Operations Center) sığdıran Widgetlar şunlardır:

### 1.1 Honeycomb (Petek) Widget'ı Devrimi
10.000 adet sunucunun CPU durumunu ekrana çizemezsiniz ancak `Honeycomb` kullanabilirsiniz. Elinizde 10.000 adet minik altıgen hücre bulunur.
- En çok CPU yiyen sunucu devasa parlak Kırmızı altıgen (Hexagon) olur.
- Kırmızı altıgene Zoom (yakınlaştırma) yaptığınızda petek büyür ve o sunucunun IP'si ekranda görünür. Zabbix arayüzündeki en iddialı ve Grafana'yı gereksiz kılan göstergedir.

### 1.2 Top Hosts (En Sorunlu Cihazlar Ligi)
`Top Hosts` widgetı sizin IT deponuzun "Lig Tablosu"dur.
Geleneksel Item değeri (CPU değerleri vb) yerine tablolar oluşturup: 
- CPU'su en yüksek ilk 10 Sunucuyu,
- Disk doluluğu en kritik ilk 5 Sunucuyu,
- RAM kullanımı son 1 saatte en hızlı değişen (Trend gradientleri ile) sunucuları canlı bar bar sıralar.

### 1.3 Geomap (Coğrafi/Map Dağılımı)
Eğer banka ATM'lerini, IoT Cihazlarını veya gemileri izliyorsanız, her cihaza Latitude ve Longitude (Enlem: `41.00` Boylam: `28.97`) koordinatları "Host Inventory" bölümünden işlenir. 
Zabbix, OpenStreetMap veya Google Maps altyapısında tüm şubelerinizi canlı Türkiye/Dünya Haritası üzerine nokta nokta serper. Herhangi bir ATM çevrimdışı (Offline) kalırsa, İstanbul'daki o kırmızı noktayı dev map ekranında görebilirsiniz.

## 2. Dynamic (Dinamik) Dashboardlar: Tıklayıp Değişen Formlar

Klasik bir yapıda MySQL sunucunuz için 1, Web Sunucunuz için 1, Ağ cihazlarınız için 1 ayrı dashboard açarsanız elinizde 50 adet dashboard olur, yönetemezsiniz. 

**Çözüm: Template Dashboard ve Dynamic Item Geçişidir.**

- **Template Dashboard (Şablon Kartı):** Siz sadece 1 defaya mahsus `Template OS Linux`'in içine girip bir Dashboard yaratırsınız (Diskler, Çekirdek Kullanımı, CPU I/O wait). Otomatik olarak, sistemdeki 500 adet Linux sunucunun herbiri kendi Zabbix ana sayfasında (Host Dashboard sekmesinde) bu ekrana sahip olur!
- **Dynamic Widgets:** Dashboard oluştururken grafik özelliklerinden **`Dynamic Item`** işaretini tiklerseniz, ekranın üstüne çıkan Host Bar'ından "DB_Sunucu_1" yazınca tüm ekran saniyede DB-1'i çizer. O kutudan "Web_Sunucu_2" derseniz Dashboard sayfa değiştirmeden saniyede veri değiştirir.

## 3. Zabbix vs Grafana Birlikteliği

Mevcut Zabbix Widgetları (Honeycomb vb) ne kadar iyi olursa olsun, Gözlemlenebilirlik dünyasında Grafana rüzgarını es geçmek imkansızdır.

### Zabbix'teki Milyonlarca Veriyi Grafana'ya Aktarmak
Zabbix arkaplanından veri çalmak için SQL'e girmek hatadır. Alexander Zobnin tarafından geliştirilen resmi "Grafana Zabbix Plugin" kurulur.
Bu eklenti Zabbix Sunucusunun `API`'sine bağlanır.
Siz Grafana'da bir grafik çizerken: "Zabbix DB'den git Group: Linux, Host: WEB-01, Item: CPU" dersiniz. Grafana SQL yerine Zabbix API (Modül 15'te anlatılan Tokenlar ile) sorgusu atarak grafiği Grafana stili (Karanlık tema ve modern bar chartlar) ile renderlar (işler).

---
[Önceki Modül](./12_Zabbix_Proxy.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Rol ve Yetki Yönetimi](./14_Kullanici_Yonetimi.md)

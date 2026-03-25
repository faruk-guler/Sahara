# Modül 12: Geniş Ölçekli Ortamlarda Zabbix Proxy ve Proxy Grupları

On binlerce cihazı izlerken her birinin tüm performans (Network, CPU) yükünü tek bir Zabbix Server'a yıkmak felakettir. Ayrıca yurt dışındaki veri merkezlerinden Türkiye'deki ana Zabbix sunucusuna (Outbound) 10 bin farklı Firewall (Port) kuralı açmak imkansızdır.

Çözüm: **Zabbix Proxy**. Proxy veriyi sahadan toplar, kompresler (sıkıştırır), ufak bir yığın haline getirip güvenli zırhlı bir tünelden (TLS/PSK üzerinden) Server'a fırlatır.

## 1. Zabbix Proxy Neden Kurulur? (4 Ana Neden)

1. **Performans (Load Offloading):** 50.000 adet sensörü Server kontrol ederse CPU %100 olur. 10 adet farklı Proxy'ye bu işi paylaştırırsanız Server sadece bir "Veritabanına Yazıcı (Writer)" haline gelir ve asıl yük Proxylere aktarılır.
2. **Güvenlik Çözümü (DMZ / NAT):** Cloud (AWS, Azure) lokasyonlarında veya banka DMZ'sinde olan cihazlar dışarıya (Server'a) konuşamaz. İçeriye bir Proxy atılır. Proxy içerideki binlerce bilgisayarın portunu tarayıp içeri veri alır, ardından tek kanal üzerinden TCP 10051 portundan dışarı çıkar.
3. **Data Buffer (Ağ Kesintilerini Tolere Etme):** İzmir şubesinden Merkez Merter'deki Zabbix'e gelen metro internet fibere takıldı ve koptu. Proxy kullanmasaydık 8 saatlik veri tamamen "Kayıp" olacaktı ve grafikte Boşluk (Gap) görecektik. Oysa Proxy varsa, SQLite (Bellek tabanlı DB) sayesinde: `ProxyOfflineBuffer=24` saatlik veriyi harddiskinde tutar. İnternet gelince saatler öncesinin verisini tıpkı Flashback yapar gibi Server DB'sine geriye dönük işler.
4. **Keşif (Discovery):** Ağa atılan SNMP taramaları ve port taramaları ağır Network Broadcast yaratır. Bunu Server yapacağına sahaya dikilen en uçtaki Proxy cihazlar yapar, bant genişliği tıkanmaz.

## 2. Proxy Yönetim Modeli (Active vs Passive)

Agent'taki mantık burada da geçerlidir.
*   **Active Proxy (ÖNERİLEN):** Proxy, veriyi kendi SQLite DB'sinde sıkar. Kendi inisiyatifi ile Zabbix Server'a "Ben İstanbul Proxy'si, al bu devasa JSON paketi" diyerek PUSH yapar.
*   **Passive Proxy:** Zabbix Server kendi eliyle Proxy'lerin kapısını çalar (POLL). Çok fazla Poller harcadığı için tavsiye edilmez.

## 3. Zabbix 7.0 ile Gelen Oyun Değiştirici: Proxy Groups (Yük Dengeleme)

Eskiden "Ankara-Proxy-1" çöktüğünde (fiziksel donanım arızalandığında), o proxy'nin üzerine zimmetli 3.000 sunucu **gri** ye döner ve sistem kör kalırdı. Elle müdahale edip hostları Sunucuya taşımak gerekirdi.

2026/7.0 sürümü itibarıyla **Zabbix Load Balancing (Proxy Groups)** devrede:

1. Ana Sayfadan (Administration -> Proxy Groups) `Ankara-Proxyleri` adında bir grup oluşturursunuz.
2. Bu gruba 3 adet Proxy atanır: (Ankara-Prx1, Ankara-Prx2, Ankara-Prx3)
3. İzlenen 3000 host, tek bir proxy'ye değil, "Ankara-Proxyleri" grubuna zimmetlenir. Zabbix bunların tamamını **Round-Robin** mantığıyla paylaştırır (1000, 1000, 1000).
4. Eğer `Ankara-Prx1`'in fişi çekilirse... Geri kalan 1.000 cihazlık veri anında ve otomatik olarak diğer 2 proxy'ye kaydırılır (Failover).

*Bu teknoloji Zabbix'in sadece yazılımsal boyuttan çıkıp Datacenter standartlarında yatay büyüyebilir (Horizontally Scalable) olmasını resmileştirmiştir.*

---
[Önceki Modül](./11_SLA_Servisler.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Dashboardlar](./13_Dashboardlar.md)

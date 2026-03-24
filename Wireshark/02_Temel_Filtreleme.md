# 2. Temel Filtreleme Teknikleri

Yakalanan paket yığınları içinde gezinmek imkânsıza yakındır. Karmaşık sorgular oluşturmak için mutlaka `&&` (AND/Ve), `||` (OR/Veya) ve `!` (NOT/Değil) mantıksal operatörleri kullanılarak filtreler birleştirilmelidir.

---

## 2.1 Ağ ve IP Adresi Filtreleri

* **Sadece Belirli Kaynak IP'yi Göster:**
  `ip.src == 10.10.10.11`
* **Sadece Hedef IP'yi Göster:**
  `ip.dst == 10.10.10.11`
* **Tüm İletişim (İster Kaynak, İster Hedef Olsun):**
  `ip.addr == 10.10.10.11`
* **Çoklu IP Taraması:**
  `ip.addr == 10.10.10.11 || ip.addr == 10.10.10.12`
* **Özel IP'yi Göz Ardı Etme:**
  `!(ip.addr == 10.10.10.12)`
* **MAC Adresi Taraması:**
  `eth.addr == 00:70:f4:23:18:c4`
* **Broadcast (Yayın) Trafiği:**
  `eth.dst == ff:ff:ff:ff:ff:ff`

---

## 2.2 Port ve Katman Filtreleri

* **Sadece 80 Portu (HTTP) Trafiği (TCP):**
  `tcp.port == 80`
* **Hedefe Özel TCP Port Taraması:**
  `tcp.dstport == 80`
* **Kaynağa Özel UDP Port Taraması:**
  `udp.srcport == 53`
* **HTTP Metotları ile Taramak:**
  `http.request.method == "GET"` veya `http.request.method == "POST"`
* **Kelime / URL Hedefli Arama:**
  `http.host == "www.google.com"` veya `http.host contains "google"`
* **Sadece Yapılan DNS Sorguları (Cevaplar Hariç):**
  `dns.flags.response == 0`
* **Ekstra Büyük Paket Tespiti (Anomali veya DoS):**
  `frame.len > 2000`

---

## 2.3 TCP Bayrak (Flags) Analizi

Siber saldırıları veya bağlantı kopmalarını analiz ederken, `TCP Header` (Başlık) içerisindeki `Flags` kısmını süzeriz.

* **Sadece Bağlantı Başlangıçları (SYN - Tarama/Tarama Girişimi):**
  `tcp.flags.syn == 1 && tcp.flags.ack == 0`
* **Bağlantı Sonlandırmaları (FIN):**
  `tcp.flags.fin == 1`
* **Zorla Sıfırlanan, Kopan veya Reddedilen Bağlantılar (RST):**
  `tcp.flags.reset == 1`
* **Veriyi Beklemeden İtme/Zorlama Komutları (PUSH - Veri Sızıntısı Analizinde Taranır):**
  `tcp.flags.push == 1`
* **Acil Paket Bildirimleri (URG):**
  `tcp.flags.urg == 1`

---

## 2.4 Temel Teşhis Protokolleri (ARP & DHCP)

Ağdaki cihazların nasıl IP aldığını ve birbirlerinin fiziksel (MAC) adreslerini nasıl bulduğunu anlamak için bu filtreler kullanılır.

### ARP (Adres Çözümleme)

* **Tüm ARP trafiği:** `arp`
* **Belirli bir MAC adresi için yapılan ARP sorgusu:** `arp.dst.hw_mac == 00:11:22:33:44:55`

### DHCP / BOOTP (IP Tahsisi)

* **Tüm DHCP trafiği:** `dhcp` veya `bootp`
* **Sadece DHCP Discover (Cihazın IP arama isteği):** `dhcp.option.dhcp == 1`
* **Sadece DHCP Ack (Sunucunun IP onay cevabı):** `dhcp.option.dhcp == 5`

[< Gelişmiş Tehdit Analizi >](03_Gelismis_Tehdit_Analizi.md)

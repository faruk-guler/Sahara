# Wireshark Kapsamlı Analiz ve Kullanım Kılavuzu

Wireshark (eski adıyla Ethereal), ağ trafiğini yakalamak ve derinlemesine analiz etmek için kullanılan, Windows, Linux ve MacOS gibi platformlarda çalışan dünyanın en popüler ağ protokol analizi aracıdır. Bu doküman, temel seviyeden ileri seviyeye kadar Wireshark kullanımını özetlemektedir.

---

## 1. Arayüz ve Görünüm Özelleştirme
Daha verimli analizler yapmak için Wireshark arayüzünü aşağıdaki kolonları ekleyerek özelleştirebilirsiniz.

**Ayarlar:** `Edit → Preferences → Columns` yolunu izleyerek yeni kolonlar ekleyin:

| Kolon Adı | Format / Alan (Custom) | Açıklama |
| :--- | :--- | :--- |
| **No.** | `%m` | Paket sıra numarası |
| **Time** | `%t` | Zaman damgası |
| **Source** | `%s` | Kaynak IP adresi |
| **Destination** | `%d` | Hedef IP adresi |
| **Protocol** | `%p` | Protokol tipi |
| **Length** | `%L` | Paket boyutu |
| **DNS Name** | `dns.qry.name` | DNS sorgu adı |
| **TLS SNI** | `tls.handshake.extensions_server_name` | TLS Sunucu adı (Server Name Indication) |
| **HTTP Host** | `http.host` | HTTP Host bilgisi |
| **TCP Stream** | `tcp.stream` | TCP akış numarası |
| **Dst Port** | `tcp.dstport` | Hedef port numarası |
| **Info** | `%i` | Genel bilgilendirme |

---

## 2. Temel Filtreleme Operatörleri
Karmaşık sorgular oluşturmak için aşağıdaki mantıksal operatörleri kullanabilirsiniz:

*   **AND (`&&`):** Her iki koşul da doğru olmalı. (Örn: `http && ip.src == 192.168.0.1`)
*   **OR (`||`):** Koşullardan biri doğru olmalı.
*   **NOT (`!`):** Belirtilen koşulu hariç tutar. (Örn: `!(ip.addr == 10.10.10.12)`)

---

## 3. Ağ Adresi ve Port Filtreleri

### IP Adresi Filtreleri
*   **Tüm Trafik:** `ip.addr == 10.10.10.11`
*   **Sadece Kaynak:** `ip.src == 10.10.10.11`
*   **Sadece Hedef:** `ip.dst == 10.10.10.11`
*   **Belirli Bir Bilgisayar (Hostname):** `ip.host == sunucu05`

### Port ve Protokol Filtreleri
*   **TCP Port:** `tcp.port == 80`
*   **UDP Port:** `udp.port == 53`
*   **Belirli Bir IP ve Port:** `ip.addr == 10.10.10.11 and tcp.port == 80`
*   **MAC Adresi:** `eth.addr == 00:70:f4:23:18:c4`
*   **Yayın (Broadcast) Trafiği:** `eth.dst == ff:ff:ff:ff:ff:ff`

---

## 4. Uygulama Katmanı Filtreleri (HTTP, DNS, TLS)

### HTTP Filtreleri
*   **Tüm HTTP İstekleri:** `http.request`
*   **Sadece GET Metodu:** `http.request.method == GET`
*   **Sadece POST Metodu:** `http.request.method == POST`
*   **Belirli Bir URL/Host:** `http.host == "www.google.com"` veya `http.host contains "google"`

### Gelişmiş Filtreler (Hızlı Analiz Şablonları)
Aşağıdaki filtreler, bir ağdaki şüpheli veya önemli trafiği hızlıca izole etmek için kullanılabilir:

*   **DNS & TLS İzleme:** `dns || tls.handshake.extensions_server_name`
*   **Genel Analiz Seti:** `dns || http || ssl.handshake || tcp.flags.syn == 1`
*   **Detaylı Analiz (IP Bazlı):** 
    `(ip.addr == 192.168.1.43) && (tcp.flags.syn == 1 || dns || tls.handshake.extensions_server_name || http.request || quic || frame.len > 1500)`

---

## 5. Kablosuz Ağ (WLAN) Analizi
WLAN trafiğini analiz ederken Frame tiplerine göre filtreleme yapabilirsiniz:

### Frame Tipleri
*   **Management Frame (Yönetim):** `wlan.fc.type == 0` (Cihaz bağlantı süreçleri)
*   **Control Frame (Kontrol):** `wlan.fc.type == 1` (Veri bütünlüğü kontrolü)
*   **Data Frame (Veri):** `wlan.fc.type == 2` (Asıl verinin taşındığı frame)

### Özel WLAN Filtreleri
*   **Association Request:** `wlan.fc.type_subtype == 0`
*   **Association Response:** `wlan.fc.type_subtype == 1`
*   **Probe Request:** `wlan.fc.type_subtype == 4`
*   **Probe Response:** `wlan.fc.type_subtype == 5`
*   **Beacon Signals:** `wlan.fc.type_subtype == 8`
*   **Authentication Request:** `wlan.fc.type_subtype == 11`
*   **Deauthentication:** `wlan.fc.type_subtype == 12`

---

## 6. Pratik İpuçları ve Kısayollar

*   **Arama Yapmak:** `Ctrl + F` tuşlarına basarak "String" seçeneği ile paket içeriklerinde arama yapabilirsiniz.
*   **Akışı Takip Et:** Bir pakete sağ tıklayıp `Follow -> TCP Stream` diyerek tüm iletişimi okunabilir bir metin olarak görebilirsiniz.
*   **Büyük Paketler:** Boyutu 2000 byte'tan büyük olan paketleri bulmak için: `frame.len > 2000`

---
> [!TIP]
> Wireshark'ta filtreleri kaydetmek için filtre çubuğunun yanındaki "+" butonunu kullanabilirsiniz. Bu sayede sık kullandığınız karmaşık filtreleri tekrar yazmak zorunda kalmazsınız.

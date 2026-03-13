# Wireshark Master Analiz ve Kullanım Kılavuzu

Wireshark, ağ trafiğini yakalamak ve derinlemesine analiz etmek için kullanılan, dünyanın en popüler açık kaynaklı ağ protokol analizi aracıdır. Bu rehber, kurulumdan ileri düzey konfigürasyona ve analize kadar her şeyi kapsamaktadır.

---

## 1. Kurulum ve Başlangıç

### Windows Kurulumu
1.  [Wireshark Resmi Sayfası](https://www.wireshark.org/download.html) üzerinden yükleyiciyi indirin.
2.  Kurulum sırasında **Npcap** yüklemesini mutlaka onaylayın. Npcap, ağ kartınızdan ham (raw) veri paketlerini yakalamak için gereklidir.
3.  **USBPcap** (Opsiyonel): USB trafiğini analiz etmek istiyorsanız bu seçeneği de işaretleyin.

### Linux Kurulumu (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install wireshark
```
*Not: Kurulum sırasında "Should non-superusers be able to capture packets?" sorusuna **Yes** diyerek, Wireshark'ı her seferinde `sudo` ile çalıştırmaktan kurtulabilirsiniz.*

### macOS Kurulumu
Resmi web sitesinden `.dmg` dosyasını indirin veya Homebrew kullanın:
```bash
brew install --cask wireshark
```

---

## 2. İlk Konfigürasyon ve Arayüz

### Karma ve Monitör Modu
*   **Promiscuous Mod (Karma):** Ağ kartınızın sadece size gelen değil, ağ segmentindeki tüm paketleri yakalamasını sağlar. (`Capture -> Options`)
*   **Monitor Mod:** Kablosuz ağlarda (WLAN) trafiği havadan (air) yakalamak için kullanılır. Ek donanım desteği gerektirebilir.

### Kolon Özelleştirme (Analiz Verimliliği)
Analiz hızını artırmak için `Edit -> Preferences -> Columns` yolundan aşağıdaki kolonları ekleyin:

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

## 3. Filtreleme Teknikleri

### 3.1. Yakalama Filtreleri (Capture Filters)
Paketler henüz yakalanmadan önce uygulanır. `Capture -> Options` kısmındaki alana yazılır.
*   `host 192.168.1.1` : Sadece bu IP trafiğini yakala.
*   `port 80` : Sadece 80 portunu yakala.
*   `not icmp` ve `not dns`: Gereksiz trafiği yakalamadan CPU tasarrufu yap.

### 3.2. Görüntüleme Filtreleri (Display Filters)
Yakalanan veriler içinde arama yapmak için kullanılır.

| Operatör | Anlamı | Örnek |
| :--- | :--- | :--- |
| `==` | Eşittir | `ip.addr == 192.168.1.1` |
| `&&` | AND (Ve) | `http && ip.src == 1.1.1.1` |
| `\|\|` | OR (Veya) | `tcp.port == 80 \|\| tcp.port == 443` |
| `!` | NOT (Değil) | `!dns` |
| `contains` | İçerir | `http.host contains "google"` |

---

## 4. Uzman Analiz Şablonları (AI Destekli Filtreler)
Ağ trafiğini hızlıca analiz etmek için aşağıdaki karmaşık filtre setlerini kullanabilirsiniz:

### Genel Gözetleme (Claude/GPT/Gemini Şablonları)
Bu filtreler ağdaki kritik olayları (bağlantı başlangıçları, DNS, TLS, POST istekleri vb.) tek bir ekranda toplar:

*   **Standart Analiz:**
    `(ip.addr == 192.168.1.43) && (tcp.flags.syn == 1 || dns || tls.handshake.type == 1 || http.request || quic || frame.len > 1500)`
*   **Tam Güvenlik Taraması:**
    `(tcp.flags.syn == 1 && tcp.flags.ack == 0) || (dns.flags.response == 0 && dns.qry.name) || tls.handshake.extensions_server_name || http.request || quic || (udp && not (udp.port == 53 || udp.port == 123 || udp.port == 443))`

---

## 5. Protokol Bazlı Derin Analiz

### 5.1. Kablosuz Ağ (WLAN) Detayları
WLAN frame yapılarını analiz etmek için aşağıdaki alt tipleri kullanın:

| Frame Tipi / Alt Tipi | Filtre Komutu | Açıklama |
| :--- | :--- | :--- |
| **Management** | `wlan.fc.type == 0` | Yönetim paketleri (Genel) |
| **Association Request**| `wlan.fc.type_subtype == 0` | Bağlantı isteği |
| **Association Response**| `wlan.fc.type_subtype == 1` | Bağlantı cevabı |
| **Probe Request** | `wlan.fc.type_subtype == 4` | Ağ arama isteği |
| **Probe Response** | `wlan.fc.type_subtype == 5` | Ağ arama cevabı |
| **Beacon** | `wlan.fc.type_subtype == 8` | Sinyal yayını |
| **Authentication** | `wlan.fc.type_subtype == 11` | Kimlik doğrulama |
| **Deauthentication** | `wlan.fc.type_subtype == 12` | Bağlantı kesme |
| **Control** | `wlan.fc.type == 1` | Akış kontrol paketleri |
| **Data** | `wlan.fc.type == 2` | Veri taşıyan paketler |

### 5.2. TLS ve Şifreli Trafik
*   **Client Hello (SNI):** `tls.handshake.type == 1`. Hangi siteye gidildiğini (şifreli olsa bile) görmek için.
*   **Server Name Indication:** `tls.handshake.extensions_server_name`.

---

## 6. İstatistik ve Teşhis Araçları

### İstatistik Menüsü
*   **Endpoints:** En çok konuşan cihazları tespit eder.
*   **Protocol Hierarchy:** Ağdaki trafik dağılımını (TCP vs UDP vb.) gösterir.
*   **TCP Stream Graphs:** Paket kaybı veya gecikme (latency) sorunlarını görselleştirir.

### Expert Info (Uzman Bilgisi)
Ekranın sol altındaki renkli butona basarak Wireshark'ın yakaladığı anormallikleri görün:
- **Chat:** Standart iş akışları.
- **Note:** HTTP Error (404 vb.) gibi durumlar.
- **Warn:** TCP Retransmission (Paket tekrarı) gibi ağ sorunları.
- **Error:** Ciddi protokol hataları.

---

## 7. Pratik Kısayollar ve İpuçları

*   **Ctrl + F:** Paketlerin içinde metin (String) aratın.
*   **Follow TCP/HTTP Stream:** Sağ tık -> `Follow`. Tüm yazışmayı bir metin belgesi gibi okuyun.
*   **Export Objects:** `File -> Export Objects -> HTTP`. Trafikten resim/dosya ayıklayın.
*   **Profile Management:** Sağ alt köşeden farklı analiz işleri için farklı profil ayarları oluşturun.

---
> [!TIP]
> Şifreli (HTTPS) trafiği çözmek için: Tarayıcınızı `SSLKEYLOGFILE` ortam değişkeniyle başlatın ve bu dosyayı `Edit -> Preferences -> Protocols -> TLS -> (Pre)-Master-Secret log filename` kısmına gösterin.

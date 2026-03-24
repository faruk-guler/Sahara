# 1. Kurulum ve Arayüz Düzeni

Wireshark (eski adıyla Ethereal), ağ trafiğini yakalamak ve derinlemesine analiz etmek için kullanılan, Windows, Linux, macOS başta olmak üzere birçok platformda çalışan dünyanın en popüler ağ protokol analizi aracıdır.

---


## 1.1 Kurulum

### Windows Kurulumu

1. [Wireshark Resmi Sayfası](https://www.wireshark.org/download.html) üzerinden yükleyiciyi indirin.
2. Kurulum sırasında **Npcap** yüklemesini onaylamayı unutmayın. Npcap, ağ kartınızdan ham (raw) veri paketlerini yakalamak için mecburi olan güçlü bir sürücüdür.
3. Opsiyonel olarak, USB trafiğini de analiz etmek istiyorsanız kurulum ekranında **USBPcap** seçeneğini de işaretleyebilirsiniz.

### Linux Kurulumu (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install wireshark
```

> [!NOTE]
> Kurulum sırasında terminalde sorulan *"Should non-superusers be able to capture packets?"* sorusuna **Yes** derseniz, Wireshark'ı sonraki her seferinde `sudo` yetkisiyle başlatmak zorunda kalmadan, standart kullanıcınız ile çalıştırabilirsiniz.

### macOS Kurulumu

Resmi web sitesinden `.dmg` dosyasını indirip yükleyebilir veya pratik bir şekilde Homebrew ile kurabilirsiniz:

```bash
brew install --cask wireshark
```

---


## 1.2 İlk Konfigürasyon

### Karma ve Monitör Modu

* **Promiscuous Mode (Karma - Standart Mod):** Ağ kartınızın yönlendirildiği segmentteki yalnızca size ait verileri değil, ağdaki tüm paketleri yakalamasını sağlar. Paket yakalama seçeneklerinden (`Capture -> Options`) varsayılan olarak açık gelir.
* **Monitor Mode (İzleme Modu):** Yalnızca kablosuz ağ (WLAN) arayüzlerine has bir özelliktir. Havadaki (air) paketleri şifrelenmiş de olsa yakalamak için kullanılır.

---


## 1.3 Kolon Özelleştirmesi

Wireshark'ın varsayılan kolon düzeni detaylı analizler için her zaman yeterli olmayabilir. İnceleme esnasında hangi pakette ne tür veriler olduğunu hızlıca anlayabilmeniz için, aşağıdaki gibi kendi özel kolonlarınızı tasarlayabilirsiniz.

Menüden `Edit -> Preferences -> Columns` yolunu izleyin veya doğrudan kolon başlıklarına sağ tıklayıp "Column Preferences" kısmından yeni satırlar ekleyin:

| Kolon Adı İstediğiniz | Field (Alan) veya Format Türü | Ne İşe Yarar / Açıklama |
| :--- | :--- | :--- |
| **No.** | `%m` | Yakalanan paket sırası |
| **Time** | `%t` | Paketin zamansal damgası |
| **Source** | `%s` | Paketi gönderen (kaynak) IP adresi |
| **Destination** | `%d` | Paketin alıcısı (hedef) IP adresi |
| **Protocol** | `%p` | Protokol (TCP, UDP, TLS vb.) bilgisi |
| **Length** | `%L` | Bayt cinsinden paketin boyutu |
| **DNS Name** | `dns.qry.name` | Çözümlenmek istenen DNS Domaini |
| **TLS SNI** | `tls.handshake.extensions_server_name` | TLS el sıkışmasındaki sunucu (SSL Gidilecek Hedef) adı |
| **HTTP Host** | `http.host` | Şifresiz HTTP Host (URL) bilgisi |
| **TCP Stream** | `tcp.stream` | İlgili TCP paketinin birbiriyle bütün TCP akış indeksi |
| **Src Port** | `tcp.srcport` / `udp.srcport` | Paketin çıkmış olduğu kaynak port numarası |
| **Dst Port** | `tcp.dstport` / `udp.dstport` | Paketin ulaşmak istediği hedef port numarası |
| **Info** | `%i` | Wireshark'ın paket hakkındaki genel bilgilendirmesi |

---

## 1.4 Konfigürasyon Profilleri (Configuration Profiles)

Farklı analiz senaryoları (Siber Güvenlik, VoIP, Troubleshooting) için farklı arayüz, kolon ve filtre ayarları gerekebilir. Wireshark'ta her senaryo için ayrı bir profil oluşturup tek tıkla geçiş yapabilirsiniz.

1. Sağ alt köşedeki **Profile:** yazısına sağ tıklayın veya `Edit -> Configuration Profiles...` yolunu izleyin.
2. Mevcut profilinizi kopyalayarak yeni bir isim verin (Örn: `Siber_Guvenlik`).
3. Bu profil seçiliyken yaptığınız tüm kolon, renk ve filtre ayarları sadece bu profile kaydedilir.

[< REKLAM / BİR SONRAKİ SAYFAYA GEÇ (Temel Filtreleme) >](02_Temel_Filtreleme.md)

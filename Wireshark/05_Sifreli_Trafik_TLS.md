# 5. Şifreli Trafik (TLS) ve Uygulama Katmanı

Modern web trafiğinin neredeyse tamamı şifrelidir. Wireshark içerisinde `http` trafiği kadar `tls` (eski adıyla SSL) analizleri de kritik önem taşır.

> [!WARNING]
> Modern ve güncel Wireshark sürümlerinde güvenlikle alakalı TLS protokolünde artık sınıf adı `ssl` olarak geçmez. Filtrelerde `ssl.handshake` yerine daima **`tls.handshake`** kullanılmalıdır.

---

## 5.1 TLS Handshake ve SNI (Server Name Indication) Yakalama

* **Client Hello Yakalama:**
  `tls.handshake.type == 1`
  Bu paket, iletişimin sadece başlangıcında ortaya çıkar. HTTPS dahi kullanılsa; şifreleme anahtarları oluşmadan önce istemci, **gideceği sunucunun alan adını dışarı sızdırır.** (SNI aracılığıyla).

* **Gidilen Hedefi (SNI) Görme Filtresi:**
  İletişim HTTPS dahi olsa aradığınız kişinin hangi web sayfasına veya hostuna gittiğini yakalayın:
  `tls.handshake.extensions_server_name`

---

## 5.2 HTTPS Trafiğini Gerçekten Çözmek (Decrypt)

Sadece SNI başlıklarını değil, HTTP paketinin içeriğini (şifreleri, gönderilen mesajları) de okumak istiyorsanız tarayıcınızın oluşturduğu geçici şifreleme anahtarlarını Wireshark'a vermeniz gerekir:

> [!TIP]
> Kendinize ait bilgisayarın Şifreli (HTTPS) olan TLS trafiğini tamamıyla okumak için: Tarayıcınızı masaüstü çevre değişkenleriyle başlatarak (`SSLKEYLOGFILE=C:\ssl\log.txt`) loglar ürettirin. Sonra o log dosyasını `Edit -> Preferences -> Protocols -> TLS -> (Pre)-Master-Secret log filename` yönergesi ile Wireshark'a tanıtarak tüm kilitleri (padlock) aşın!

[< İstatistikler ve Kısayollar >](06_Istatistik_ve_Tezhis.md)

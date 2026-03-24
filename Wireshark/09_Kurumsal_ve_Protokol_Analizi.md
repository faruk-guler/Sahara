# 9. Kurumsal Ağ ve Protokol Analizi

Kurumsal ağlarda (Enterprise Networks) standart web trafiğinin ötesinde; isim çözme mekanizmaları, dosya paylaşımları ve kimlik doğrulama süreçleri yoğunluktadır. Bir Siber Güvenlik Analisti veya Ağ Uzmanının bu protokollerde uzmanlaşması şarttır.

---

## 9.1 DNS (Domain Name System) Analizi

Ağdaki şüpheli hareketlerin birçoğu (C2 sunucu iletişimleri, Malware veri sızdırma) DNS üzerinden başlar.

* **Sadece DNS Trafiği:** `dns` (Sadece 53. port üzerinden giden istek ve cevaplar).
* **Başarısız DNS Sorguları (NXDOMAIN):** Alan adı bulunamadı hataları, genellikle bir zararlının rastgele alan adları (DGA - Domain Generation Algorithm) türettiğine veya bir Domain Controller sorunu olduğuna işaret eder.
  `dns.flags.rcode == 3` (3 = No such name)
* **Spesifik Alan Adı Sorguları:**
  `dns.qry.name contains "facebook.com"` veya `dns.qry.name == "malicious.com"`
* **Uzun DNS Sorguları (DNS Tunneling Tespiti):** Saldırganlar dışarıya veri kaçırmak (Exfiltration) için çok uzun TXT veya A kayıtları aratabilirler.
  `dns.qry.name.len > 50`

---

## 9.2 SMB (Server Message Block) ve Windows Ağları

Şirket içi dosya paylaşımı ve Active Directory etkileşimleri SMB protokolü üzerinden yürür. Fidye yazılımı (Ransomware) taranması veya yetkisiz dosya kopyalama tespiti için hayati önem taşır.

* **Tüm SMB/SMB2 Trafiği:** `smb || smb2`
* **Dosya Açma/Okuma İşlemleri:** Hangi dizinlere erişiliyor?
  `smb2.filename` (Ağdaki bir PC'de açılan veya kopyalanan dosyaların adlarını listeler).
* **Klasör Erişim İstekleri:**
  `smb2.cmd == 5` (Create/Open komutu)
* **Giriş Başarısızlıkları (Logon Failures):** Brute-Force veya yanal hareket (Lateral Movement) tespiti.
  `smb2.nt_status == 0xc000006d` (STATUS_LOGON_FAILURE)

---

## 9.3 HTTP/2 ve HTTP/3 (QUIC) Trafiği

Günümüzde çoğu popüler platform, HTTP/1.1 yerine performans ve güvenlik nedeniyle HTTP/2 veya UDP tabanlı QUIC (HTTP/3) protokollerini kullanmaktadır.

* **HTTP/2 Trafiği:** `http2`
* **QUIC Protokolü:** `quic`
  *Not: QUIC trafiği UDP tabanlı ve varsayılan olarak şifrelidir. Tıpkı TLS modülünde (Modül 5) bahsettiğimiz gibi, `SSLKEYLOGFILE` yöntemiyle QUIC trafiğini de çözebilirsiniz.*

---

## 9.4 Açık Metin (Plaintext) Parolaların Yakalanması

Eski veya yanlış yapılandırılmış ağlarda kimlik doğrulama verileri şifrelenmeden gidebilir (Telnet, FTP, HTTP Basic Auth). Bu verileri Wireshark'ta anında tespit edebilirsiniz.

* **FTP Şifreleri:**
  `ftp.request.command == "USER" || ftp.request.command == "PASS"`
* **HTTP Temel Kimlik Doğrulaması (Base64 kodlu şifreler):**
  `http.authbasic`
* **Telnet Parola Denemeleri:**
  `telnet contains "password" || telnet contains "login"`

[< Pcap İşlemleri (Önceki) >](08_Renklendirme_ve_Pcap_Islemleri.md) | [< Performans ve Optimizasyon (Sonraki) >](10_Performans_ve_Ozel_Yapi.md)

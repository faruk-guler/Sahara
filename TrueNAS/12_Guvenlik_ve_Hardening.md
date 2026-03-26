# Modül 12: TrueNAS Güvenlik (Hardening) ve Kapanış

Sistemi tasarlayıp, veri paylaşımlarını hızlandırdık. Ancak bir Veri Depolama sisteminin dış tehlikelerden (Örn: Ransomware sızması veya İnsan hatası/Kötü niyet) korunması sistemin devamlılığı için son savunma hattıdır.

## 1. Web Yüzeyinin Sıkılaştırılması (Hardening)

TrueNAS Web arayüzü fabrika çıkışıyla (Out-of-box) standart HTTP (80) portundan güvensiz açılır ve parolalar "Clear text" olarak ağınıza saçılır.

### Aksiyon 1: HTTPS (TLS) Sertifikası Atamak

Self-Signed (Kendi imzaladığı ve tarayıcının kırımızı "Güvenli Değil" ibaresi bastığı) sertifikadan kaçınılmalıdır.

- *Kurumsal Çözüm:* Active Directory'nin Certificate Base (CA Root) yapısından alınan X.509 formatlı sertifika ve Public Key içe aktarılarak HTTPS'ye gömülür.
- *Modern Çözüm:* `Credentials` -> `Certificates` -> `ACME DNS-Authenticator` kullanılarak (Cloudflare üzerinden vs.) ZFS kendiliğinden *Let's Encrypt* sertifikasını her 90 günde bir otomatik çeker ve yeniler (Yeşil Asma Kilit garantisi). Ardından `System Settings` -> `General` üzerinden sistem kesinlikle (Force HTTPS) seçeneği ile kilitlenmelidir.

### Aksiyon 2: İki Aşamalı Doğrulama (2FA/MFA)

Bütün yöneticilerin paneli Google Authenticator desteğine sahiptir. System Settings üzerinden 2FA (Two-Factor Authentication) açılır ve cihaz (QR barkodu) cep telefonu ile eşleştirilir. IT yöneticisi şifresini Excel dosyasına kaydetse ve bu dosya çalınsa bile hacker panele login olamaz.

## 2. Veri İstirahat Şifrelemesi: Encryption at Rest

Birisi Datacenter'a (Kabinete) sızıp 4 adet diski söküp kaçarsa? Arayüz şifreleri diskleri korumaz, disk başka bilgisayara takıldığında (Zpool Import denildiğinde) saniyeler içinde bütün Dataya ulaşılır. Verinin diskteyken Kriptolu kalması gereklidir. İki farklı yöntem vardır:

### 2.1 ZFS Native Encryption

Modül 05'te Dataset'i yaratırken `Encryption` seçeneği (AES-256-GCM algoritmasıyla) açılır.

- **Gücü:** İşletim sisteminin ve işlemcinin devasa (AES-NI destekli) hızını donanımdan soyutlanmış şekilde uygular.
- **Mimarisi:** Sistem ilk açıldığında (Boot) o Havuz Kilitlidir. Dataya ulaşılamaz. Siz Key'i (Passphrase/JSON File) girdiğiniz an Havuz Un-Lock olur ve anında okunur hale gelir. Bu dosyayı (Anahtarı) kaybetmeniz, veriyi sonsuza dek imha etmeniz (Hacker ile aynı kefeye düşmeniz) demektir.

### 2.2 SED (Self-Encrypting Drives)

Kurumsal olarak SED markalı SATA/NVMe diskler alındığında, şifreleme işini ZFS yazılımı DEĞİL, Diskin kendi devre kartındaki (Çip) donanımsal donanım yapar. Hız kaybı tam olarak %0'dır. Ancak ZFS bu işlemin farkında bile değildir (Güven yönetimi donanım seviyesindedir).

## 3. SSH Hizmetinin (Kapısının) Duvarlanması

ZFS bakımı ve otomasyon araçları `root` (veya Master Admin) ayrıcalığıyla arka kapıdan (Port 22 SSH) girerek komut gönderir.

- Hizmetler (Services) sekmesinden SSH servisi yapılandırmasında `Log in as Root with Password` (Root parolasını yazarak direkt girmek) KESİNLİKLE devredışı bırakılmalıdır (Brute-Force Attack kurbanı olursunuz!).
- Admin sisteme sadece `SSH Key Pairs` (Public/Private Key) yöntemi ve `.pem / .pub` anahtarları aracılığıyla şifre sorulmadan, Kriptografik anahtar uyuşmasıyla sızabilmelidir.
- Ağ kartında (Modül 06) eğer VLAN/Bridge ayarları doğru yapıldıysa, SSH servisi `0.0.0.0` (Herhangi bir ağ kartı) üzerinden dinlenmez, yalnızca ve yalnızca **Admin Management VLAN Subnet'ine (Örn: 10.0.1.X)** (Bind Interface) bağlanarak izole edilir.

### Sonuç: Master Mimarisi Hazır

Bir "TrueNAS Master"ı sadece bir menüden paylaşım tıklayan kişi değil, altında yatan Linux/ZFS Kernel'ini, Copy-on-Write mantığının kırılma dinamiklerini ve Veri İzolasyon (VLAN/MFA/Crypto) yöntemlerini senkronize eden kişidir. Okuduğunuz 12 modül sizi standart donanım yöneticisinden ayırıp, Data Mimarı (Storage Architect) pozisyonuna taşımıştır.

---
[Önceki Modül: Modül 11](./11_Performans_Tuning.md)

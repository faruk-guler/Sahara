# Modül 20: Zabbix Platform Hardening, TLS Şifreleme ve Kasa Entegrasyonu (Secret Management)

Zabbix, doğası gereği sunucuların (Linux/Windows) tüm disklerini, loglarını, bellek verilerini okuma (hatta Modül 10'da gösterildiği üzere komut çalıştırma) gücüne sahiptir. Bu durum, Zabbix'i bilgisayar korsanları (Hackerlar) için "Geleceğin Casus Yazılımı" (En kolay hedefler listesindeki sıçrama tahtası) konumuna getirir.

Sistemi sıkılaştırmak (Hardening) Master (Kurumsal) düzeyin tartışılmaz şartıdır.

## 1. Zabbix Bileşenleri Arası (Internal) TLS Şifrelemesi

Eğer Zabbix Agent (İzlenen Makine) ile Zabbix Server/Proxy arasındaki iletişim açık metin (Cleartext / TCP 10050/10051) olarak kalırsa, ağ içerisindeki herhangi bir sızdırıcı (Sniffer/Man-in-the-Middle) giden gelen konfigürasyon (veritabanı logları, API Keyler) paketlerini okuyabilir. Mimarinin tamamı şifrelenmelidir (Encryption).

### 1.1 PSK (Pre-Shared Key) - Küçük/Orta Ölçekli Çözüm

- İki taraf (İzlenen Ajan ve İzleyen Sunucu) statik (Örn: 32 Karakter Rastgele Üretilmiş) ortak bir şifre kullanır.
- `zabbix_agent2.conf` içinde `TLSConnect=psk` yapılarak bu rastgele (Örn: `1f8e...`) PSK Key Ajanın diskinde (`/etc/zabbix/psk.file`) tutulur.
- Kurulum otomasyonu için Ansible kullanılır.

### 1.2 x509 Certificates (Sertifika Mimarisi) - Enterprise Çözüm

Elinizde 15.000 cihaz varsa tek tek PSK dağıtmak/güncellemek imkânsızdır.
Şirketin Kök Sertifikasyonu (Root CA), Active Directory veya Vault üzerinden her sunucu bir imzalı SSL Sertifikası (`subject=WEB-Server-01`) alır. Zabbix İletişimi, cihaz isimlerini SSL Headerlarından denetleyerek kurar.

## 2. Parola Yönetimi: HashiCorp Vault ve CyberArk Entegrasyonu

Modül 07'de anlatılan "Secret Macro" Zabbix'in kendi DB'si içerisinde çalışır. Ancak devasa Datacenter güvenlik mimarilerinde **Hiçbir ürün** şifrelerini (Active Directory, MySQL şifresi, API anahtarı, SNMP cihaz şifreleri) kendi veritabanında saklayamaz. Bütün sır perdesi Merkezi bir Kasa'da (Vault) tutulur.

**Nasıl Çalışır?**

1. Zabbix Server'a Vault'a bağlanabilmesi için özel bir `Root Vault Token` verilir (`zabbix_server.conf` içine: `VaultToken=xxxx`).
2. Vault'un makine adresi verilir (`VaultURL=https://10.0.0.9:8200`).
3. Veritabanı (Zabbix DB PostgreSQL) şifresi dahi konfigürasyonda elle yazılmaz! `DBPassword=secret:database/zabbix_db_creds` makrosu kullanılır.
4. Zabbix servisi ilk başlarken (Sıfırıncı Saniye) Vault'un kapısını çalar, anlık olarak PostgreSQL şifresini sorar. Şifreyi saniyesinde alır, doğrudan Login olur ve RAM'den tamamen siler (Ekrana asla basmaz, diske işlemez).
5. Vault Admin, şifreyi ayda 1 kez değiştirse bile, Zabbix bunu 1 saniye sonra farkeder ve sistemi yeniden başlatmadan taze parolayı alarak çalışmaya devam eder.

## 3. Web Arayüzü Hardening (Nginx / Apache Güvenliği)

Zabbix Web UI açıkta ise:

- **X-Frame-Options Koruması:** `DENY` veya `SAMEORIGIN`. Başka bir "Kötü" sitenin Zabbix arayüzünü (Iframe) olarak çalmasını ve yöneticinin yanlışlıkla tıklamasını (ClickJacking) engellemek. Sunucunun `nginx.conf` veya `httpd.conf` dosyasına yazılır.
- **SSL / TLS 1.3 Zorunluluğu:** HTTP kesinlikle kapalı olmalıdır, port 80 direkt 443'e (HTTPS) dönürülür ve HSTS (`Strict-Transport-Security: max-age=31536000;`) kuralları eklenir.
- **Ajan (Ping/Komut) Sınırlandırması:** Linux Agent makinesinde, RemoteCommand servisini açarken Zabbix kullanıcısını `Root` yetkisiyle ASLA donatmayın, sadece `/etc/sudoers.d/zabbix` içinden 3 spesifik servisi (`/usr/bin/systemctl restart nginx` vb.) resetlemeye sınırlandırın. Aksi taktirde Zabbix sunucusu ele geçerse Zabbix ajanlarının kurulu olduğu bütün sunucu çiftliğiniz anında çöker ("Root" olan Hacker yüzünden).

---
[Önceki Modül](./19_Bulut_ve_K8s.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Kurumsal Raporlama](./21_Raporlama.md)

# Modül 04: Zabbix Agent 2, Mimarisi ve Plugin Yönetimi

Zabbix dünyasında 2026 yılı itibarıyla eski nesil "C" tabanlı Ajan (Zabbix Agent) yalnızca Gömülü Sistemler (Embedded, IoT) veya çok eski OS'ler (CentOS 6 vb.) için desteklenir. Kurumsal ortamlarda **Golang (Go)** tabanlı **Zabbix Agent 2** zorunluluktur.

## 1. Neden Zabbix Agent 2?

Eski Ajan ile Ajan 2 arasındaki en belirgin fark **Eşzamanlılık (Concurrency)** yeteneğidir:

- **Persistent Connections (Kalıcı Bağlantılar):** Veritabanı (Örn: PostgreSQL, MySQL) izlemelerinde her sorguda yeni bir TCP/IP bağlantısı açıp kapatmak yerine bağlantıyı açık tutar (Connection Pooling). Bu veritabanındaki yükü %90 azaltır.
- **Third-Party Plugin Desteği:** Go diliyle yazıldığı için yeni servisleri izlemek (Örn: Redis, Docker, Systemd) için external script, UserParameter yazmak zorunda kalmazsınız. Native pluginler ile gelir.
- **Data Buffering:** Ağ bağlantısı kopsa bile topladığı verileri SQLite veya Memory'de tutarak bağlantı gelince pushlar.

## 2. Ajan Türleri: Aktif (Active) vs Pasif (Passive)

Mimarinin performansını temelden etkileyen konudur.

### Pasif Çekim (Passive Check) - Port 10050

1. Zabbix Server/Proxy, ajan'a sorar: "192.168.1.5'in CPU'su nedir?"
2. Ajan hesaplar ve cevaplar.
3. *Sorun:* Server binlerce TCP bağlantısı açıp kapatmak zorunda kalır. Çok fazla Poller süreci tüketilir. Performans düşüşü yaşanır.

### Aktif İletim (Active Check) - Port 10051 (ÖNERİLEN)

1. Ajan, Zabbix Server/Proxy'e sorar: "Benim adım WEB-01, izlemem gereken metrikler (Item) neler?"
2. Server/Proxy listeyi gönderir: "CPU(1dk), Disk(5dk), Ping(1dk) izleyip bana at."
3. Ajan artık kendi kendine hesaplamaları yapar ve verileri paketleyip Zabbix'e fırlatır.
4. *Avantajı:* Binlerce sunucuda ölçeklenir, Firewall ve NAT arkasından dışarı çıkan (Outbound) bağlantı olduğu için kural yazmak çok daha basittir.

## 3. Profesyonel `zabbix_agent2.conf` Konfigürasyonu

Kurumsal bir `zabbix_agent2.conf` dosyasında şu parametreler bulunmalıdır:

```ini
# --- PASİF KONTROLLER İÇİN (Sadece Test/Ping için kullanılması önerilir) ---
# Server/Proxy IP adresi
Server=192.168.10.50

# --- AKTİF KONTROLLER İÇİN (Esas Veri Akışı) ---
ServerActive=192.168.10.50

# --- HOSTNAME AYARLARI ---
# Sunucunun adını elle girmemek (otomatize etmek) için sistem adını çeker:
HostnameItem=system.hostname

# --- OTOMATİK KEŞİF (AUTO-REGISTRATION) METADATASI ---
# Bu parametre Modül 05'te detaylı işlenecek. Hangi template'lerin ekleneceğini belirler.
HostMetadata=Linux-Web-Nginx

# --- PERFORMANS VE LİMİTLER ---
BufferSend=5
BufferSize=1000

# --- GÜVENLİK (TLS PSK) ---
# Şifrelenmemiş veri gönderimini engellemek için.
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=SecretPSK_WEB_01
TLSPSKFile=/etc/zabbix/zabbix_agent2.psk
```

## 4. Zabbix Agent 2 Eklentileri (Plugins)

Agent 2, pluginleri loadable (yüklenebilir) DLL veya .so dosyaları olarak destekler.
Bir Docker ortamını izlemek isterseniz:

1. `usermod -aG docker zabbix` komutu ile Zabbix kullanıcısına izin verin.
2. Ajanı yeniden başlatın. Zabbix Agent 2, Docker socket API'sini doğrudan okuyarak *Container CPU, Memory ve I/O* metriklerini anında (External script olmadan) Zabbix'e iletir.

### Sorun Giderme (Troubleshooting) Komutları

Metriklerin düzgün okunup okunmadığını Server üzerinden sorgulamak için (Pasif check denemesi için):

```bash
zabbix_get -s 192.168.1.10 -k "system.cpu.load[all,avg1]"
```

Ajan tarafından kendi bilgisini test etmek için:

```bash
zabbix_agent2 -t "docker.containers.discovery"
# Eğer "Permission Denied" alırsanız Zabbix kullanıcısını doğru gruba (docker) almamışsınız demektir.
```


---
[Önceki Modül](./03_Kurulum.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Otomatik Keşif](./05_Otomatik_Kesif.md)

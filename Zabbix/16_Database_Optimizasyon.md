# Modül 16: Üst Seviye (Master) Database Optimizasyon ve Performans Tuning

Bir Zabbix Mimarisi çöktüğünde sebep %95 Frontend, %4 network, **%91 Veritabanı Darboğazıdır.** (I/O Wait). Veritabanını optimize etmeyen hiçbir Zabbix uzmanı kendisine "Master" diyemez.

## 1. Zabbix Cache (Önbellek) Mekanizması

Zabbix Server, veritabanına yazmadan önce verileri RAM (Bellek) üzerinde depolar (Buffer). Veritabanı yavaşladığında Zabbix kendi RAM'ini o kadar doldurur ki, Zabbix Server Out of Memory (OOM) yiyip çöker.
Bunu engellemek için `/etc/zabbix/zabbix_server.conf` dosyasında şu cache (önbellek) değerlerinin Ram miktarınıza göre artırılması gerekir:

- **CacheSize=8G** (Binlerce cihazınızın item, trigger ve obje tanımlamalarının (Configuration) durduğu hafıza alanıdır. Küçük kalırsa sistem kitlenir).
- **HistoryCacheSize=2G** (Her bir cihazdan akan Milyarlarca verinin veritabanına gideceği saniyeye kadar depolandığı, arabellektir).
- **TrendCacheSize=1G** (Ortalama değerlerin saatlik uydurulduğu alandır).

**Kritik Gözlem:** Ekranda `Zabbix server history syncer processes more than 75% busy` hatası görürseniz Veritabanı diski o kadar zayıf ki (veya Postgresql tune edilmemiş ki) saniyede 15.000 MB'i (NVPS) diske yazamıyor, yukarıdaki RAM limitleri hıncahınç dolmuş ve Zabbix can çekişiyor demektir.

## 2. PostgreSQL + TimescaleDB Housekeeping

Standard bir ilişkisel veritabanında (Örn: MySQL MyISAM veya basit InnoDB) 2 yıllık, 5 milyar satırlık verinin içinden 7 gün öncesini `DELETE` komutuyla silmek "Lock" (Kilit) yaratır. Silme işlemi arka planda diskte saatlerce sürer, Zabbix felç olur (Bu işleme Housekeeping denir).

**Mimarinin Çözümü: TimescaleDB Chunks (LSM-Tree Mimarisi)**
Zabbix tabloları TimescaleDB üzerinde dev dalgalar yerine, günlük "Chunk" (Parçacık) olarak bölünür.
Pazartesi günü (Chunk-A), Salı günü (Chunk-B).
1 Ay sonra, 30 gün öncesine ait History verisini silmek istediğinizde Zabbix veritabanına büyük bir `DELETE` atarak diski tıkamaz; direkt dosyayı havaya uçurur (Drop Chunk). Silme işlemi mili saniyeler sürer, CPU kullanımını (I/O) %90 oranda azaltır.

Bunun çalışması için Zabbix arayüzünden `Administration` -> `Housekeeping` sekmesine girip, **"Enable override for history period"** değerinin tikinin açılıp (Örn: 7d veya 30d) Timescale tarafına yetki devredilmesi zorunludur.

## 3. PostgreSQL Ölçekleme: PgBouncer + WAL Optimizasyonu

Büyük veri merkezlerinde bir PostgreSQL 16 makinesinin RAM/CPU kullanımı rastgele bırakılamaz.

### Write-Ahead Logging (WAL) Tuning

Zabbix her saniye Milyonlarca satır veri okuyan değil **sürekli veri yazan (Write-Heavy)** bir mimaridir.
PostgreSQL'in yazma hızı (Checkpoint performansıyla) direkt olarak Zabbix Queue'sunu belirler. `postgresql.conf` içinde:

```ini
# SSD disklere sahip bir NVMe ortamında performansı katlamak için:
max_wal_size = 8GB              # Küçük WAL boyutu checkpoint fırtınası yaratır ve I/O kilitler. 8G iyidir.
checkpoint_timeout = 15min      # Her 5 dakikada değil, 15 dakikada bir diske bas, diski yorma.
checkpoint_completion_target = 0.9
```

### PgBouncer (Connection Pooling)

Her bir Zabbix Poller, Trapper veya Frontend UI servisi PostgreSQL'e anlık doğrudan Socket (Bağlantı) açar. Binlerce process olduğunda (max_connections=3000 yapmak yanlıştır), PostgreSQL "Connection Saturated" (Bağlantı darboğazı) olur ve çöker.
PgBouncer, PostgreSQL önüne bir duvar örerek "Gelen 3.000 isteği havuzlar, arkadaki 100 kalıcı bağlantı hortumundan (Pool) akıtır" Zabbix'e hız katar.

---
[Önceki Modül](./15_API_Otomasyon.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Bakım Modu](./17_Bakim_Modu.md)

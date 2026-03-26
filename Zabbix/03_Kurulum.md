# Modül 03: Master Seviye Kurulum ve Database Deployment

Zabbix'in standart bir LAMP/LEMP kurulumundan farkı, performans odaklı yapılması gereken kernel, veritabanı ve güvenlik (SELinux) tuning işlemleridir. Bu modül, Rocky Linux 9 / RHEL 9 üzerinde 2026 kurumsal mimari kurulumunu temel alır.

## 1. Paket Yönetimi ve Repository
Zabbix'in resmi deposu kurulmalı ve ön bellekleme temizliği yapılmalıdır:

```bash
# Zabbix v7.0/v8.0 LTS Repository
rpm -Uvh https://repo.zabbix.com/zabbix/7.0/rocky/9/x86_64/zabbix-release-7.0-1.el9.noarch.rpm
dnf clean all
```

Tüm modüller (HA destekli server, Agent 2, Web Service, SNMP destekleri vb.):
```bash
dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent2 zabbix-web-service google-chrome-stable
```

## 2. PostgreSQL ve TimescaleDB Optimizasyonlu Kurulum
PostgreSQL standart ayarlarla Zabbix (OLTP tarzı yazma-yoğun) için yetersiz kalır. Ek olarak `TimescaleDB` Zabbix'in can damarıdır.

```bash
# PostgreSQL 16 ve TimescaleDB kurulumu
dnf install postgresql16-server timescaledb-2-postgresql-16
/usr/pgsql-16/bin/postgresql-16-setup initdb

# Timescale kütüphanesini PostgreSQL'e bağlamak
echo "shared_preload_libraries = 'timescaledb'" >> /var/lib/pgsql/16/data/postgresql.conf
systemctl enable --now postgresql-16
```

### 2.1. Veritabanı Tuning (`timescaledb-tune`)
Ram ve CPU kaynaklarınızı TimescaleDB'ye tanıtarak otomatik PostgreSQL `postgresql.conf` ayarlarını yazdırmak hayati önem taşır:

```bash
timescaledb-tune --pg-config=/usr/pgsql-16/bin/pg_config
# [Onaylayarak devam edin (y)]
systemctl restart postgresql-16
```

### 2.2. Zabbix Veritabanını Hazırlama
```bash
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```

## 3. Veri Aktarımı (Schema Import) ve TimescaleDB Aktivasyonu
İlk olarak Zabbix tabloları oluşturulur. Ancak buradaki EN KRİTİK ADIM, `-U zabbix` (yetkisiz kullanıcı) yerine tablo yapısını PostgreSQL extension kurulumu yapabilecek şekilde yüklemektir:

```bash
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

Daha sonra TimescaleDB uzantısını Zabbix'e bildirmek için şu fonksiyon çalıştırılır:
```bash
echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix
cat /usr/share/zabbix-sql-scripts/postgresql/timescaledb/schema.sql | sudo -u postgres psql zabbix
```
*Bu komutla birlikte Zabbix altındaki tüm ağır tablolar (history, trends) otomatik Chunk (Parça) yapısına dönüştürülür.*

## 4. `zabbix_server.conf` Konfigürasyonu
`/etc/zabbix/zabbix_server.conf` düzenleyerek DB bağlantısını sağlayın:

```ini
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=ZabbixComplexPassword2026!

# HA (High Availability) Aktivasyonu (Opsiyonel)
HANodeName=Ankara-Node-1
NodeAddress=192.168.10.101:10051

# Raporlama için Web Service
StartReportWriters=3
WebServiceURL=http://localhost:8053/report
```

## 5. Güvenlik, SELinux ve Firewall (Master İpuçları)
Güvenlik duvarlarını kapatmak `systemctl stop firewalld` bir acemi hareketidir. Doğru kurallar:

```bash
# Zabbix Portları
firewall-cmd --permanent --add-port=10051/tcp # Server
firewall-cmd --permanent --add-port=10050/tcp # Agent
firewall-cmd --permanent --add-service={http,https}
firewall-cmd --reload
```

### SELinux'i Kapatmadan Zabbix'i Çalıştırmak
Kurumsal firmalarda SELinux kapatılamaz! Zabbix'in yerel policy'sini yüklediğimiz için (`zabbix-selinux-policy`) sadece boolean izin vermemiz gerekir:
```bash
setsebool -P zabbix_can_network 1
setsebool -P httpd_can_connect_zabbix 1
```

## 6. Servisleri Başlatma ve Hata Analizi
```bash
systemctl enable --now zabbix-server zabbix-agent2 httpd php-fpm
```

***Kritik Sorun Giderme:*** Servis çalışmıyorsa ilk bakmanız gereken yer logdur:
```bash
tail -100f /var/log/zabbix/zabbix_server.log
```
Eğer *"database is down"* veya *"connection refused"* görüyorsanız MySQL/PostgreSQL soketini veya şifrenizi kontrol etmelisiniz.

---
[Önceki Modül](./02_Mimari.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Zabbix Agent 2](./04_Zabbix_Agent2.md)

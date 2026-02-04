# 🔧 Ceph Troubleshooting Rehberi

Bu doküman, Ceph cluster'ında karşılaşılan yaygın sorunların teşhisi ve çözümünü kapsar.

---

## 1. Cluster Health Sorunları

### HEALTH_WARN Durumları

```bash
# Detaylı sağlık bilgisi
ceph health detail
```

#### "1 osds down"

```bash
# Hangi OSD?
ceph osd tree | grep down

# OSD loglarını kontrol et
cephadm logs --name osd.5 -- --tail 100

# Disk durumunu kontrol et
smartctl -a /dev/sdb

# OSD'yi yeniden başlat
ceph orch daemon restart osd.5
```

#### "clock skew detected"

```bash
# NTP durumunu kontrol et
chronyc sources

# Saat farkını gör
chronyc tracking

# Chrony'yi yeniden başlat
systemctl restart chrony
```

#### "X pgs inactive"

```bash
# Hangi PG'ler?
ceph pg ls | grep -v active

# PG detayı
ceph pg <pg-id> query
```

#### "X pgs degraded"

```bash
# Replikasyon eksik - veri hala okunabilir
# OSD durumlarını kontrol et
ceph osd tree

# Recovery durumunu izle
ceph -w
```

### HEALTH_ERR Durumları

#### "X pgs inconsistent"

```bash
# Hangi PG?
ceph health detail

# Repair başlat (dikkatli!)
ceph pg repair <pg-id>
```

#### "full ratio(s) reached"

```bash
# Disk doluluk kontrolü
ceph osd df tree

# Acil: Yeni disk ekle veya veri sil
# Geçici: Oranları ayarla (dikkat!)
ceph osd set-full-ratio 0.97
ceph osd set-nearfull-ratio 0.90
```

---

## 2. OSD Sorunları

### OSD Başlamıyor

```bash
# Log kontrolü
cephadm logs --name osd.5 -- --tail 200

# Yaygın nedenler:
# 1. Disk bozuk
smartctl -H /dev/sdb

# 2. Mount sorunu
lsblk

# 3. Container imaj sorunu
podman images | grep ceph

# 4. Keyring sorunu
ceph auth get osd.5
```

### OSD Sürekli Crash Oluyor

```bash
# Crash dump'ları kontrol et
ls /var/lib/ceph/crash/

# Son crash'i görüntüle
ceph crash ls-new
ceph crash info <crash-id>

# Crash'i arşivle (tekrar gösterilmez)
ceph crash archive <crash-id>
# veya hepsini
ceph crash archive-all
```

### Slow OSD Tespiti

```bash
# Yavaş OSD'leri bul
ceph osd perf | sort -k2 -n | tail -10

# Blocked operations
ceph daemon osd.5 dump_blocked_ops

# Slow ops uyarısı
ceph health detail | grep slow
```

---

## 3. PG (Placement Group) Sorunları

### PG Durumları

| Durum | Anlam | Aksiyon |
| :--- | :--- | :--- |
| `active+clean` | Normal | - |
| `active+degraded` | Eksik replika | OSD'leri kontrol et |
| `peering` | OSD'ler senkronize oluyor | Bekle |
| `recovering` | Veri kurtarılıyor | Bekle |
| `backfilling` | Veri taşınıyor | Bekle |
| `incomplete` | Yeterli OSD yok | OSD ekle |
| `stale` | PG güncel değil | MON/OSD kontrol |
| `inconsistent` | Veri tutarsız | Repair |

### Stuck PG'leri Çözme

```bash
# Stuck PG'leri listele
ceph pg dump_stuck

# Stuck türleri
ceph pg dump_stuck inactive
ceph pg dump_stuck unclean
ceph pg dump_stuck stale

# PG detaylı durumu
ceph pg <pg-id> query
```

### Inconsistent PG Repair

```bash
# DİKKAT: Scrub önce, repair sonra
ceph pg deep-scrub <pg-id>

# Sonuç kontrolü
ceph pg <pg-id> query | grep state

# Repair (sadece inconsistent ise)
ceph pg repair <pg-id>
```

### PG Undersized/Degraded

```bash
# Acting set'i kontrol et
ceph pg <pg-id> query | grep acting

# Yeterli OSD var mı?
ceph osd tree

# Pool replica sayısını kontrol et
ceph osd pool get <pool> size
```

---

## 4. MON Sorunları

### Quorum Kaybı

```bash
# MON durumu
ceph mon stat
ceph quorum_status

# Hangi MON'lar quorum'da?
ceph mon dump

# MON loglarını kontrol et
cephadm logs --name mon.node1 -- --tail 100
```

### MON Election Sorunları

```bash
# Election sürekli tekrarlanıyorsa:
# 1. Clock skew kontrolü
chronyc sources -v

# 2. Network latency kontrolü
ping -c 10 node2

# 3. MON'ları sırayla yeniden başlat
ceph orch daemon restart mon.node1
```

### MON Database Büyümesi

```bash
# MON DB boyutunu kontrol et
ceph tell mon.* compact

# DB durumu
ceph daemon mon.node1 mon_status
```

---

## 5. Network Sorunları

### Public vs Cluster Network Sorunları

```bash
# Network yapılandırmasını kontrol et
ceph config get mon public_network
ceph config get osd cluster_network

# OSD'den ping testi
cephadm shell -- ping -c 5 192.168.1.11
```

### Slow Network Tespiti

```bash
# OSD heartbeat sorunları
ceph daemon osd.0 dump_historic_ops | grep -i timeout

# Network istatistikleri
ip -s link
```

### Firewall Sorunları

```bash
# Gerekli portlar açık mı?
# MON: 3300, 6789
# OSD: 6800-7300
# MGR: 8443

# Firewall kurallarını kontrol et
iptables -L -n
# veya
firewall-cmd --list-all
```

---

## 6. CephFS Sorunları

### MDS Sorunları

```bash
# MDS durumu
ceph fs status

# MDS logları
cephadm logs --name mds.myfs.node1 -- --tail 100
```

### Client Mount Sorunu

```bash
# Kernel modülü yüklü mü?
lsmod | grep ceph

# MON'lara erişim var mı?
ping 192.168.1.10

# Secret key doğru mu?
cat /etc/ceph/ceph.client.user1.keyring
```

### Stale Client Sessions

```bash
# Aktif oturumları görüntüle
ceph daemon mds.myfs.node1 session ls

# Takılı oturumu zorla kapat
ceph daemon mds.myfs.node1 session evict client.12345
```

---

## 7. RGW Sorunları

### RGW Başlamıyor

```bash
# Log kontrolü
cephadm logs --name rgw.myrgw.node1 -- --tail 100

# Port çakışması kontrolü
ss -tlnp | grep 8000
```

### S3 Erişim Sorunu

```bash
# Kullanıcı mevcut mu?
radosgw-admin user info --uid=testuser

# Access key doğru mu?
radosgw-admin user info --uid=testuser | grep access_key

# Bucket yetkileri
radosgw-admin bucket stats --bucket=mybucket
```

### RGW Performance Issues

```bash
# Frontend iş parçacıkları
ceph config get client.rgw rgw_thread_pool_size

# Artır
ceph config set client.rgw rgw_thread_pool_size 512
```

---

## 8. Recovery ve Backfill Yönetimi

### Recovery Hızını Ayarlama

```bash
# Mevcut ayarlar
ceph config get osd osd_recovery_max_active
ceph config get osd osd_max_backfills

# Hızlandır (dikkat: client performansı düşer)
ceph config set osd osd_recovery_max_active 5
ceph config set osd osd_max_backfills 3
ceph config set osd osd_recovery_sleep 0

# Yavaşlat (client önceliği)
ceph config set osd osd_recovery_max_active 1
ceph config set osd osd_max_backfills 1
ceph config set osd osd_recovery_sleep 0.5
```

### Recovery'yi Durdurma

```bash
# Geçici durdurma
ceph osd set norecover
ceph osd set nobackfill

# Tekrar başlat
ceph osd unset norecover
ceph osd unset nobackfill
```

### Recovery İlerlemesini İzleme

```bash
# Genel durum
ceph -s

# Detaylı ilerleme
ceph pg stat

# Kalan veri miktarı
ceph pg <pg-id> query | grep bytes_recovered
```

---

## 9. Scrubbing Sorunları

### Deep Scrub Takılması

```bash
# Scrub durumunu kontrol et
ceph pg dump | grep -i scrub

# Scrub zamanlamasını görüntüle
ceph pg <pg-id> query | grep scrub

# Manuel scrub tetikle
ceph pg deep-scrub <pg-id>
```

### Scrub'ı Geçici Devre Dışı Bırakma

```bash
# Performans için geçici durdurma
ceph osd set noscrub
ceph osd set nodeep-scrub

# Tekrar etkinleştir
ceph osd unset noscrub
ceph osd unset nodeep-scrub
```

---

## 10. Acil Durum Komutları

### Cluster'ı Koruma Moduna Alma

```bash
# Tüm değişiklikleri durdur
ceph osd set noout
ceph osd set norebalance
ceph osd set norecover
ceph osd set nobackfill
ceph osd set noscrub
ceph osd set nodeep-scrub
```

### Koruma Modundan Çıkma

```bash
ceph osd unset noout
ceph osd unset norebalance
ceph osd unset norecover
ceph osd unset nobackfill
ceph osd unset noscrub
ceph osd unset nodeep-scrub
```

### Force Recovery (Son Çare)

```bash
# PG'yi zorla aktif yap (VERİ KAYBI RİSKİ!)
ceph pg force-recovery <pg-id>

# Daha agresif (ÇOK RİSKLİ!)
ceph pg force-backfill <pg-id>
```

---

## 11. Diagnostic Komutları

### Daemon Admin Socket

```bash
# OSD config'ini görüntüle
ceph daemon osd.0 config show

# Performans sayaçları
ceph daemon osd.0 perf dump

# İşlemdeki operasyonlar
ceph daemon osd.0 dump_ops_in_flight

# Geçmiş operasyonlar
ceph daemon osd.0 dump_historic_ops
```

### Object Düzeyinde Debug

```bash
# Object yerini bul
ceph osd map <pool> <object-name>

# Object metadata
rados -p <pool> stat <object-name>

# Object içeriğini al
rados -p <pool> get <object-name> /tmp/object-data
```

---

## 12. Troubleshooting Checklist

### Hızlı Kontrol

```bash
# 1. Cluster sağlığı
ceph -s

# 2. OSD durumu
ceph osd tree

# 3. PG durumu
ceph pg stat

# 4. Disk doluluk
ceph osd df tree

# 5. Son olaylar
ceph log last 20
```

### Sorun Tespiti Sırası

```text
1. ceph health detail → Hatayı anla
2. ceph -w → Gerçek zamanlı izle
3. cephadm logs → İlgili daemon logları
4. ceph osd tree → OSD durumları
5. ceph pg dump → PG detayları
6. smartctl → Disk sağlığı
7. dmesg → Kernel mesajları
```

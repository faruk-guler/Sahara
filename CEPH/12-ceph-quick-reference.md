# 12. Ceph Hızlı Referans ve Yönetim Rehberi (Cheat Sheet)

Bu belge, Ceph kurulumu sonrası sık kullanılan komutlar, yapılandırma kontrolleri, kullanıcı yönetimi ve temel izleme işlemleri için bir **"Hızlı Başvuru"** (Cheat Sheet) niteliğindedir.

> **İpucu:** Belirli bir işi (örn: VMware Datastore, S3 Bucket) nasıl yapacağınızı arıyorsanız, **[14-ceph-scenario-cookbook.md](14-ceph-scenario-cookbook.md)** dosyasına bakın.

---

## 0. CLI Erişimi (Ceph Shell)

Tüm komutları çalıştırmadan önce Ceph yönetim kabuğuna (shell) girmelisiniz.

```bash
# Ceph yönetim kabuğuna giriş
cephadm shell

# Veya tek seferlik komut çalıştırma (Shell'e girmeden)
cephadm shell -- ceph status
```

---

## 1. Versiyon ve Sağlık Kontrolü

Kümenizin genel durumunu ve çalışan versiyonları kontrol etmek için kullanılan temel komutlar.

### Küme Sağlığı

```bash
# Anlık durum ve sağlık özeti
ceph status
# veya kısaca
ceph -s

# Sağlık detayları (Warning/Error nedenlerini görmek için)
ceph health detail
```

### Versiyon Kontrolü

```bash
# Hangi Ceph sürümünün çalıştığını görmek için
ceph versions

# Cephadm sürümünü görmek için (Orchestrator node üzerinde)
cephadm version

# Tüm daemon'ların versiyon dağılımı
ceph orch ps --format json | jq -r '.[].version' | sort | uniq -c
```

---

## 2. Kullanıcı ve Erişim Yönetimi (Auth)

Ceph istemcileri (RBD, RGW, CephFS vb.) için kullanıcı oluşturma ve yetkilendirme işlemleri.

### Kullanıcıları Listeleme

```bash
# Tüm kullanıcıları ve yetkilerini listele
ceph auth list
```

### Yeni Kullanıcı Oluşturma

Örnek: `client.uygulama1` adında, `rbd` havuzuna tam yetkili bir kullanıcı.

```bash
ceph auth get-or-create client.uygulama1 \
  mon 'allow r' \
  osd 'allow class-read object_prefix rbd_children, allow rwx pool=rbd' \
  -o /etc/ceph/ceph.client.uygulama1.keyring
```

### Anahtar (Parola) Güncelleme / Değiştirme

Mevcut bir kullanıcının anahtarını değiştirmek veya yetkilerini güncellemek için `caps` (capabilities) komutları kullanılır.

```bash
# Sadece yetkileri güncellemek (Key değişmez)
ceph auth caps client.uygulama1 mon 'allow r' osd 'allow * pool=yeni_havuz'

# Anahtarı ve yetkileri tamamen yeniden basmak (Key Rotation)
# Dikkat: `get-or-create` mevcut kullanıcı varsa key'i değiştirmez.
# Key'i yenilemek için önce kullanıcı silinmeli, sonra tekrar oluşturulmalıdır:
ceph auth del client.uygulama1
ceph auth get-or-create client.uygulama1 mon 'allow r' osd 'allow *' -o /etc/ceph/ceph.client.uygulama1.keyring
```

### Kullanıcı Silme

```bash
ceph auth del client.uygulama1
```

### Dashboard Kullanıcı Parolası Değiştirme

Dashboard (Web UI) kullanıcılarının parolasını değiştirmek için:

```bash
# Dosyadan okuyarak değiştirme (Güvenli)
echo "YeniGucluParola123!" > pass.txt
ceph dashboard ac-user-set-password admin -i pass.txt

# Doğrudan komut satırında (History'de görünebilir!)
ceph dashboard ac-user-set-password admin 'YeniGucluParola123!'
```

---

## 3. Servis ve Daemon Yönetimi (Cephadm/Orchestrator)

Konteyner tabanlı servislerin (OSD, MON, MGR, RGW) yönetimi.

### Servisleri Listeleme

```bash
# Tanımlı servis özellikleri (Spec)
ceph orch ls

# Çalışan daemon süreçleri (Container durumu, Image ID vb.)
ceph orch ps
```

### Servis Başlatma / Durdurma / Restart

Belirli bir daemon'ı yeniden başlatmak için:

```bash
# Tüm OSD'leri sırayla restart etmek (Rolling restart)
ceph orch restart osd

# Sadece belirli bir OSD'yi restart etmek (Örn: osd.12)
ceph orch daemon restart osd.12

# Bir servisi durdurmak (Stop)
ceph orch daemon stop osd.12

# Bir servisi başlatmak (Start)
ceph orch daemon start osd.12
```

---

## 4. Depolama ve Havuz (Pool) İşlemleri

### Kapasite ve Kullanım

```bash
# Küme geneli disk kullanım durumu (Linux 'df' komutuna benzer)
ceph df

# OSD bazlı detaylı kullanım (Hangi disk ne kadar dolu?)
ceph osd df tree
```

### Hızlı Havuz Oluşturma

Test veya basit kullanım için (Production için "03-ceph-pool-guide.md" referans alınmalıdır).

```bash
# Replicated havuz oluşturma (varsayılan 3 kopya)
ceph osd pool create test_pool 32 32

# Havuzu RGW veya RBD için initialize etme
rbd pool init test_pool
```

### Havuz Silme (Güvenlik Korumalı)

Varsayılan olarak havuz silmek korumalıdır. Önce mon üzerine izin verilmeli, sonra silinmelidir.

```bash
# 1. Silme korumasını kaldır (Safety switch)
ceph tell mon.* injectargs --mon_allow_pool_delete=true

# 2. Havuzu sil (İsim iki kez girilir)
ceph osd pool delete test_pool test_pool --yes-i-really-really-mean-it

# 3. Güvenliği tekrar aç (Opsiyonel ama önerilir)
ceph tell mon.* injectargs --mon_allow_pool_delete=false
```

---

## 5. Maintenance ve Koruma Bayrakları (Flags)

Kümede bakım yaparken veya bir sorun oluştuğunda Ceph'in hareketlerini kısıtlamak için kullanılır.

```bash
# Otomatik iyileştirmeyi durdur (Bakım sırasında önerilir)
ceph osd set noout
ceph osd set norebalance
ceph osd set norecover

# Bayrakları kaldır (Normale dön)
ceph osd unset noout
ceph osd unset norebalance
ceph osd unset norecover

# Scrubbing'i durdur (Performans darboğazı varsa)
ceph osd set noscrub
ceph osd set nodeep-scrub

# Tüm bayrakları listele
ceph osd dump | grep flags
```

---

## 6. OSD ve Disk Yönetimi (İleri Seviye)

### Ağırlık (Weight) ve Dengeleme

```bash
# Kapasite ağırlığını değiştir (TB cinsinden)
ceph osd crush reweight osd.5 2.0

# Kullanım bazlı reweight (Dengesizliği gidermek için)
ceph osd reweight osd.5 0.85

# Tüm küme için dengesizliği otomatik düzelt
ceph osd reweight-by-utilization
```

### OSD Hata Ayıklama

```bash
# OSD'nin servis detaylarını gör
ceph orch device ls

# Bir OSD'nin konteynerine gir (Hata ayıklama için)
cephadm unit --name osd.5 --enter

# OSD performans istatistikleri
ceph osd perf
```

---

## 7. PG (Placement Group) İşlemleri

```bash
# Tüm PG'lerin durum özeti
ceph pg stat

# Sorunlu (Stuck) PG'leri bul
ceph pg dump_stuck stale|inactive|unclean

# PG Scrubbing (Manuel tetikleme)
ceph pg scrub <pg-id>
ceph pg deep-scrub <pg-id>

# Tutarsız PG'yi onar (Scrub/Deep-Scrub sonrası)
ceph pg repair <pg-id>
```

---

## 8. S3 ve Object Storage (RGW) Özet

```bash
# S3 kullanıcısı oluştur
radosgw-admin user create --uid=user1 --display-name="User 1"

# Kullanıcı listesi
radosgw-admin user list

# Bucket listele
radosgw-admin bucket list

# Bucket onarımı (Index sorunları için)
radosgw-admin bucket check --bucket=mybucket --fix

# Access/Secret Key Yönetimi (Key Rotation)
# Yeni bir anahtar çifti oluştur
radosgw-admin key create --uid=user1 --key-type=s3 --gen-access-key --gen-secret

# Belirli bir anahtarı sil
radosgw-admin key rm --uid=user1 --access-key=ESKIACCESSKEY
```

---

## 9. Yapılandırma ve Loglar

### Config Yönetimi

```bash
# Mevcut tüm özel yapılandırmaları gör
ceph config dump

# Belirli bir ayarı getir
ceph config get osd osd_max_backfills

# Bir ayarı değiştir (Kalıcı)
ceph config set osd osd_max_backfills 2

# Bir ayarı runtime'da değiştir (Geçici - Daemon restart olana kadar)
ceph tell osd.* injectargs '--debug-osd 10'
```

### Log Kontrolü

```bash
# Canlı olay akışını izle
ceph -w

# Son 50 olay
ceph log last 50

# Cephadm ile belirli daemon logları
cephadm logs --name mgr.node1
```

---

## 10. RBD (Block Storage) Hızlı Komutlar

### Image Yönetimi

```bash
# Image oluştur
rbd create mypool/disk1 --size 10G

# Image listele
rbd ls mypool

# Image bilgisi
rbd info mypool/disk1

# Image sil
rbd rm mypool/disk1

# Image boyutlandır (büyüt)
rbd resize mypool/disk1 --size 20G
```

### Snapshot ve Clone

```bash
# Snapshot oluştur
rbd snap create mypool/disk1@snap1

# Snapshot listele
rbd snap ls mypool/disk1

# Snapshot'tan geri dön (rollback)
rbd snap rollback mypool/disk1@snap1

# Clone oluştur (COW)
rbd snap protect mypool/disk1@snap1
rbd clone mypool/disk1@snap1 mypool/disk1-clone
```

### Map/Unmap (Linux Host)

```bash
# RBD image'ı host'a bağla
rbd map mypool/disk1

# Hangi device oldu?
rbd showmapped

# Host'tan ayır
rbd unmap /dev/rbd0
```

---

## 11. CephFS Hızlı Komutlar

### Volume ve Mount

```bash
# CephFS volume oluştur
ceph fs volume create myfs

# Volume durumu
ceph fs status

# Client mount (Kernel driver)
sudo mount -t ceph 192.168.1.10:6789:/ /mnt/cephfs -o name=admin,secret=<key>

# Unmount
sudo umount /mnt/cephfs
```

### Quota ve Snapshot

```bash
# Dizine quota koy (1GB)
setfattr -n ceph.quota.max_bytes -v 1073741824 /mnt/cephfs/project

# Snapshot oluştur
mkdir /mnt/cephfs/.snap/backup-$(date +%F)

# Snapshot'tan geri yükle
cp -a /mnt/cephfs/.snap/backup-2024-01-01/file.txt /mnt/cephfs/
```

---

## 12. Cephadm Orchestrator Hızlı Komutlar

```bash
# Host ekle
ceph orch host add node4 192.168.1.13

# Host çıkar
ceph orch host rm node4

# Servis uygula (örn: 3 MGR)
ceph orch apply mgr --placement="3 node1 node2 node3"

# Tüm daemon'ları listele
ceph orch ps

# Bir daemon'ı yeniden dağıt
ceph orch daemon redeploy mon.node1
```

---

## 13. Acil Durum (Emergency) Komutları

### Quick Freeze (Cluster'ı Dondur)

```bash
# Tüm iyileştirme mekanizmalarını kapat
ceph osd set noout && ceph osd set norebalance && ceph osd set norecover && ceph osd set nobackfill
```

### Quick Resume (Devam Ettir)

```bash
# Tüm kısıtlamaları kaldır
ceph osd unset noout && ceph osd unset norebalance && ceph osd unset norecover && ceph osd unset nobackfill
```

### OSD Acil Tasfiye

```bash
# OSD'yi zorla kaldır (veri kaybı riski!)
ceph osd out osd.5
ceph osd down osd.5
ceph osd crush remove osd.5
ceph auth del osd.5
ceph osd rm osd.5
```

---

## 14. CRUSH Map Yönetimi (Advanced)

### Map Manipülasyonu

```bash
# Map'i dışa aktar (binary)
ceph osd getcrushmap -o cm.bin

# Decompile et (okunabilir yap)
crushtool -d cm.bin -o cm.txt

# Compile et (binary yap)
crushtool -c cm.txt -o cm-new.bin

# Map'i inject et (uygula)
ceph osd setcrushmap -i cm-new.bin
```

### Kural ve Sınıf Değişikliği

```bash
# Device Class değiştir (örn: SSD görünümlü HDD)
ceph osd crush rm-device-class osd.0
ceph osd crush set-device-class ssd osd.0

# OSD'yi CRUSH ağacında başa bir yere taşı
ceph osd crush move osd.0 host=node2
```

---

## 15. MGR Modülleri ve Balancer

### Modül Yönetimi

```bash
# Aktif modülleri listele
ceph mgr module ls

# Modül etkinleştir/devre dışı bırak
ceph mgr module enable prometheus
ceph mgr module disable dashboard
```

### Balancer (Otomatik Dengeleyici)

```bash
# Dengeleyici durumu
ceph balancer status

# Modu "upmap" yap (Luminous+ için en iyisi)
ceph balancer mode upmap

# Aktifleştir
ceph balancer on

# Planı uygula (otomatik değilse)
ceph balancer eval
ceph balancer execute
```

---

## 16. Benchmark ve Performans Testi

### RADOS Bench (Cluster Hızı)

```bash
# Yazma testi (10 saniye)
rados bench -p mypool 10 write --no-cleanup

# Sıralı okuma testi
rados bench -p mypool 10 seq

# Rastgele okuma testi
rados bench -p mypool 10 rand

# Temizlik
rados -p mypool cleanup
```

### RBD Bechmark

```bash
# RBD map perf testi
rbd bench --io-type write --io-pattern rand --io-size 4K --io-total 1G mypool/image
```

---

## 17. Network Analizi

### Public vs Cluster Network

```bash
# Ping testi (Monitörler arası)
ceph ping mon.node2

# OSD heartbeat kontrolü (Latency sorunları için)
ceph daemon osd.0 dump_historic_ops | grep -i timeout
```

---
**Not:** Bu belge özet niteliğindedir. Detaylar için ilgili rehberlere bakınız:

* [01-ceph-concepts-guide.md](01-ceph-concepts-guide.md)
* [02-ceph-installation-guide.md](02-ceph-installation-guide.md)
* [11-ceph-troubleshooting-guide.md](11-ceph-troubleshooting-guide.md)
* [13-ceph-disaster-recovery.md](13-ceph-disaster-recovery.md)

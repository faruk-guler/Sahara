# 🎱 Ceph Pool Yönetimi Rehberi

Bu doküman, Ceph havuzlarının (pool) oluşturulması, yapılandırılması ve yönetimini kapsar. Pool'lar, Ceph'te verilerin organize edildiği mantıksal birimlerdir.

---

## 1. Pool Temelleri

### Pool Nedir?

Pool, Ceph'te nesnelerin (objects) gruplandığı mantıksal bir kapsayıcıdır. Her pool'un kendine özgü:

- Replikasyon veya Erasure Coding ayarı
- Placement Group (PG) sayısı
- CRUSH kuralı (hangi OSD'lere yazılacağı)
- Uygulama etiketi (rbd, cephfs, rgw)

### Pool Listesini Görme

```bash
# Basit liste
ceph osd lspools

# Detaylı bilgi
ceph osd pool ls detail

# İstatistiklerle birlikte
ceph df
```

---

## 2. Replicated Pool Oluşturma

### Temel Oluşturma

```bash
# Sözdizimi: ceph osd pool create <pool-adı> <pg-sayısı>
ceph osd pool create mypool 128

# Uygulama etiketi ekle (zorunlu)
ceph osd pool application enable mypool rbd
```

### Parametreli Oluşturma

```bash
# Replikasyon sayısını belirterek
ceph osd pool create mypool 128 128 replicated
ceph osd pool set mypool size 3        # 3 kopya
ceph osd pool set mypool min_size 2    # Minimum 2 kopya yazılmalı
```

### PG Sayısı Hesaplama

```text
PG Sayısı = (OSD Sayısı × 100) ÷ Replikasyon Sayısı

Örnek: 9 OSD, size=3 → (9 × 100) ÷ 3 = 300 → En yakın 2'nin kuvveti = 256
```

> **Pro Tip:** Ceph Reef'te `pg_autoscaler` varsayılan olarak açıktır. Manuel hesaplamaya gerek kalmaz.

```bash
# Autoscaler durumunu kontrol et
ceph osd pool autoscale-status
```

---

## 3. Erasure Coded (EC) Pool Oluşturma

EC, replikasyondan daha verimli disk kullanımı sağlar ancak yazma performansı düşer.

### EC Profili Oluşturma

```bash
# k=4, m=2 profili (4 veri + 2 parite = 6 parça)
# 2 OSD bozulsa bile veri kurtarılabilir
ceph osd erasure-code-profile set my-ec-profile k=4 m=2 crush-failure-domain=host

# Profili görüntüle
ceph osd erasure-code-profile get my-ec-profile
```

### EC Pool Oluşturma

```bash
ceph osd pool create ec-pool 128 128 erasure my-ec-profile
ceph osd pool application enable ec-pool rgw
```

### EC vs Replicated Karşılaştırma

| Özellik | Replicated (size=3) | EC (4+2) |
| :--- | :--- | :--- |
| Disk Verimliliği | %33 | %66 |
| Yazma Hızı | Hızlı | Yavaş (CPU yoğun) |
| Okuma Hızı | Hızlı | Orta |
| Recovery Süresi | Kısa | Uzun |
| Kullanım Alanı | RBD, CephFS | Arşiv, Yedekleme, RGW |

---

## 4. Pool Yapılandırması

### Replikasyon Ayarları

```bash
# Kopya sayısını değiştir
ceph osd pool set mypool size 3

# Minimum yazılabilir kopya (size'dan küçük olmalı)
ceph osd pool set mypool min_size 2
```

### PG Sayısını Değiştirme

```bash
# PG artırma (azaltma Reef'ten itibaren desteklenir)
ceph osd pool set mypool pg_num 256

# Autoscaler modunu ayarla
ceph osd pool set mypool pg_autoscale_mode on    # Otomatik
ceph osd pool set mypool pg_autoscale_mode warn  # Sadece uyarı
ceph osd pool set mypool pg_autoscale_mode off   # Kapalı
```

### Pool Quotası

```bash
# Maksimum boyut limiti (100 GB)
ceph osd pool set-quota mypool max_bytes 107374182400

# Maksimum nesne sayısı limiti
ceph osd pool set-quota mypool max_objects 1000000

# Quota'yı kaldır
ceph osd pool set-quota mypool max_bytes 0
```

---

## 5. Pool Silme

> ⚠️ **DİKKAT:** Pool silme geri alınamaz ve tüm verileri kalıcı olarak siler!

### Güvenlik Kilidi Açma

```bash
# Önce config'de pool silmeye izin ver
ceph config set mon mon_allow_pool_delete true
```

### Pool Silme

```bash
# İki kez pool adını yazarak sil
ceph osd pool delete mypool mypool --yes-i-really-really-mean-it
```

### Güvenlik Kilidini Tekrar Kapat

```bash
ceph config set mon mon_allow_pool_delete false
```

---

## 6. Pool Yeniden Adlandırma

```bash
ceph osd pool rename old-pool-name new-pool-name
```

---

## 7. Pool Snapshot

### Snapshot Oluşturma

```bash
ceph osd pool mksnap mypool snap1
```

### Snapshot Listeleme

```bash
rados -p mypool lssnap
```

### Snapshot Silme

```bash
ceph osd pool rmsnap mypool snap1
```

---

## 8. Pool Compression

BlueStore ile pool seviyesinde sıkıştırma aktif edilebilir.

```bash
# Sıkıştırma algoritması ayarla
ceph osd pool set mypool compression_algorithm snappy  # veya lz4, zstd, zlib

# Sıkıştırma modunu ayarla
ceph osd pool set mypool compression_mode aggressive  # none, passive, aggressive, force

# Minimum sıkıştırma boyutu (varsayılan 0)
ceph osd pool set mypool compression_min_blob_size 4096
```

### Sıkıştırma İstatistikleri

```bash
ceph osd pool stats mypool
```

---

## 9. Cache Tiering (Önbellek Katmanı)

> **Not:** Cache tiering artık önerilmiyor. Yerine SSD-based pool + CRUSH rules kullanın.

Yine de legacy bilgi olarak:

```bash
# Cache pool oluştur (SSD'lerde)
ceph osd pool create cache-pool 128

# Cache tier olarak bağla
ceph osd tier add data-pool cache-pool
ceph osd tier cache-mode cache-pool writeback
ceph osd tier set-overlay data-pool cache-pool
```

---

## 10. Pool İstatistikleri ve İzleme

```bash
# Pool kullanım istatistikleri
ceph df detail

# Pool başına IOPS ve throughput
ceph osd pool stats

# Belirli pool için detay
ceph osd pool get mypool all
```

---

## 11. CRUSH Kuralını Değiştirme

```bash
# Pool'un kullandığı CRUSH kuralını değiştir
ceph osd pool set mypool crush_rule hdd-rule
```

---

## 12. Pool Best Practices

### ✅ Yapılması Gerekenler

- Her pool'a uygulama etiketi ekleyin (`rbd`, `cephfs`, `rgw`)
- Production'da `pg_autoscaler` kullanın
- Pool silmeden önce snapshot alın
- Farklı iş yükleri için ayrı pool'lar oluşturun

### ❌ Yapılmaması Gerekenler

- Tek bir pool'da tüm verileri depolamayın
- PG sayısını çok düşük tutmayın (performans düşer)
- EC pool'u yoğun yazma gerektiren iş yükleri için kullanmayın
- `min_size=1` ayarlamayın (veri kaybı riski)

# ⬆️ Ceph Upgrade ve Migration Rehberi

Bu doküman, Ceph cluster'ının güvenli bir şekilde yükseltilmesi, sürüm geçişleri ve migration işlemlerini kapsar.

---

## 1. Ceph Sürüm Yapısı

### Sürüm İsimlendirmesi

Ceph sürümleri alfabetik sırayla hayvan/deniz yaratığı isimleri alır:

| Sürüm | İsim | Yayın Tarihi | Durum |
|-------|------|--------------|-------|
| 14.x | Nautilus | 2019 | EOL |
| 15.x | Octopus | 2020 | EOL |
| 16.x | Pacific | 2021 | Maintenance |
| 17.x | Quincy | 2022 | LTS |
| 18.x | Reef | 2023 | Current |
| 19.x | Squid | 2024 | Development |

### Sürüm Numarası Anlamı

```
18.2.1
│  │ └── Patch sürümü (hata düzeltmeleri)
│  └──── Minor sürümü (özellik eklemeleri)
└─────── Major sürümü (önemli değişiklikler)
```

---

## 2. Upgrade Öncesi Hazırlık

### Pre-Upgrade Checklist

```bash
# 1. Cluster sağlığını kontrol et
ceph -s
# HEALTH_OK olmalı!

# 2. Mevcut sürümü kontrol et
ceph version

# 3. Tüm daemon'ların çalıştığını doğrula
ceph orch ps

# 4. OSD durumlarını kontrol et
ceph osd tree
# Tüm OSD'ler UP ve IN olmalı

# 5. PG durumunu kontrol et
ceph pg stat
# Tüm PG'ler active+clean olmalı

# 6. Cluster map'i yedekle
ceph osd getmap -o /backup/osdmap-$(date +%Y%m%d).bin
ceph osd getcrushmap -o /backup/crushmap-$(date +%Y%m%d).bin
ceph mon dump > /backup/monmap-$(date +%Y%m%d).txt
```

### Upgrade Sırası

**Kritik:** Her zaman şu sırayı takip edin:

```
1. MON (Monitor)
2. MGR (Manager)
3. OSD (Object Storage Daemon)
4. MDS (Metadata Server) - CephFS kullanıyorsanız
5. RGW (Rados Gateway) - Object storage kullanıyorsanız
```

---

## 3. Cephadm ile Upgrade (Önerilen)

### Mevcut Sürümü Kontrol Etme

```bash
ceph version
ceph orch upgrade status
```

### Hedef Sürümü Belirleme

```bash
# Mevcut image'ı görüntüle
ceph config get mgr container_image

# Kullanılabilir sürümleri kontrol et (registry'den)
# Resmi image: quay.io/ceph/ceph:<version>
```

### Upgrade Başlatma

```bash
# Reef sürümüne yükselt
ceph orch upgrade start --ceph-version 18.2.1

# Alternatif: Image ile belirt
ceph orch upgrade start --image quay.io/ceph/ceph:v18.2.1
```

### Upgrade Durumunu İzleme

```bash
# Upgrade durumu
ceph orch upgrade status

# Gerçek zamanlı izleme
watch -n 5 'ceph orch upgrade status'

# Daemon sürümlerini kontrol et
ceph versions
```

### Upgrade Duraklatma/İptal

```bash
# Duraklat
ceph orch upgrade pause

# Devam et
ceph orch upgrade resume

# İptal et (dikkatli kullan!)
ceph orch upgrade stop
```

---

## 4. Rolling Upgrade Detayları

### Her Daemon Türü İçin Süreç

**MON Upgrade:**

```bash
# Cephadm otomatik yapar:
# 1. Bir MON'u yeniden başlat (yeni sürümle)
# 2. Quorum'un yeniden oluşmasını bekle
# 3. Sonraki MON'a geç
```

**OSD Upgrade:**

```bash
# Her OSD için:
# 1. OSD'yi noout moduna al (gereksiz rebalance'ı önler)
ceph osd set noout

# 2. OSD'yi durdur, güncelle, başlat
# 3. PG'lerin active+clean olmasını bekle
# 4. Sonraki OSD'ye geç
```

### Manuel OSD Upgrade (Gerekirse)

```bash
# Belirli bir host'taki OSD'leri güncelle
ceph orch daemon redeploy osd.5

# Tüm OSD'leri yeniden dağıt
ceph orch redeploy osd
```

---

## 5. Major Version Upgrade (Örn: Quincy → Reef)

### Önemli Değişiklikleri Kontrol Etme

Her major sürümün "Release Notes" ve "Upgrade Procedure" belgelerini okuyun:

- <https://docs.ceph.com/en/latest/releases/>

### Quincy → Reef Upgrade

```bash
# 1. Quincy'de en son minor sürüme yükselt
ceph orch upgrade start --ceph-version 17.2.7

# 2. Stabilize ol ve test et
ceph -s

# 3. Reef'e yükselt
ceph orch upgrade start --ceph-version 18.2.1
```

### Feature Flag'leri

Major upgrade sonrası yeni özellikler için flag'ler etkinleştirilmelidir:

```bash
# Yeni özellikleri etkinleştir (geri dönüşü zorlaştırır!)
ceph osd require-osd-release reef
```

---

## 6. Upgrade Sonrası İşlemler

### Doğrulama

```bash
# Tüm daemon'ların yeni sürümde olduğunu kontrol et
ceph versions

# Cluster sağlığı
ceph -s

# OSD durumu
ceph osd stat

# Upgrade raporunu gör
ceph orch upgrade status
```

### Post-Upgrade Optimization

```bash
# Yeni sürüm özelliklerini etkinleştir
ceph osd require-osd-release reef

# Balancer'ı kontrol et
ceph balancer status

# pg_autoscaler durumu
ceph osd pool autoscale-status
```

---

## 7. Downgrade (Geri Alma)

> ⚠️ **UYARI:** Downgrade genellikle desteklenmez ve veri kaybına yol açabilir!

### Acil Durumda Downgrade

```bash
# Sadece bir daemon için eski sürüme geç
ceph orch upgrade stop
ceph config set mgr container_image quay.io/ceph/ceph:v17.2.7

# Dikkat: Bu çok risklidir ve test edilmiş olmalıdır!
```

### Downgrade Önleme

`require-osd-release` flag'i ayarlandıktan sonra downgrade **MÜMKÜN DEĞİLDİR**.

---

## 8. Offline/Air-Gapped Upgrade

### Image'ları Önceden İndirme

```bash
# İnternet olan bir makinede
podman pull quay.io/ceph/ceph:v18.2.1
podman save quay.io/ceph/ceph:v18.2.1 -o ceph-18.2.1.tar
```

### Offline Makinelere Yükleme

```bash
# Her node'da
podman load -i ceph-18.2.1.tar
```

### Upgrade Başlatma

```bash
# Local image ile upgrade
ceph orch upgrade start --image ceph:v18.2.1
```

---

## 9. Data Migration

### Pool Migration (Replicated → EC)

```bash
# 1. Yeni EC pool oluştur
ceph osd pool create new-ec-pool 128 erasure

# 2. Verileri kopyala (rados copy-pool desteklenmez EC için)
# Uygulama seviyesinde migration gerekir

# 3. Eski pool'u sil
ceph osd pool delete old-pool old-pool --yes-i-really-really-mean-it
```

### RBD Image Migration

```bash
# Farklı pool'a taşı
rbd migration prepare old-pool/image new-pool/image
rbd migration execute old-pool/image
rbd migration commit old-pool/image
```

### CephFS Migration

```bash
# Veriyi rsync ile taşı
rsync -avz /mnt/old-cephfs/ /mnt/new-cephfs/
```

---

## 10. Disaster Recovery Senaryoları

### MON Database Recovery

Tüm MON'lar bozulduysa:

```bash
# Hayatta kalan bir MON'dan
ceph-monstore-tool /var/lib/ceph/mon/ceph-a get monmap > /tmp/monmap
ceph-monstore-tool /var/lib/ceph/mon/ceph-a rebuild
```

### OSD Recovery

```bash
# OSD veritabanını yeniden oluştur
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-5 \
    --op rebuild-pg-info
```

---

## 11. Upgrade Best Practices

### ✅ Yapılması Gerekenler

- Her zaman HEALTH_OK durumunda upgrade başlat
- Upgrade öncesi full backup al
- Önce test/staging ortamında dene
- Release notes'u mutlaka oku
- Yeterli zaman ayır (acele etme)
- Monitoring'i açık tut

### ❌ Yapılmaması Gerekenler

- HEALTH_WARN/ERR durumunda upgrade başlatma
- Birden fazla major sürüm atlama (Quincy → Reef OK, Pacific → Reef HAYIR)
- require-osd-release'i düşünmeden uygulama
- Upgrade sırasında cluster'da değişiklik yapma
- Yedek almadan upgrade başlatma

---

## 12. Upgrade Timeline Örneği

**3 Node Cluster için:**

| Aşama | Süre | Notlar |
|-------|------|--------|
| Hazırlık | 1 saat | Backup, kontrol |
| MON Upgrade | 15 dk | 3 MON x 5 dk |
| MGR Upgrade | 10 dk | 2 MGR x 5 dk |
| OSD Upgrade | 1-2 saat | OSD sayısına bağlı |
| MDS/RGW | 30 dk | Varsa |
| Doğrulama | 30 dk | Test |
| **TOPLAM** | **3-4 saat** | Küçük cluster |

---

## 13. Upgrade Checklist

### Upgrade Öncesi

```
[ ] HEALTH_OK durumu doğrulandı
[ ] Tüm OSD'ler UP ve IN
[ ] Tüm PG'ler active+clean
[ ] CRUSH map yedeklendi
[ ] OSD map yedeklendi
[ ] Release notes okundu
[ ] Test ortamında denendi
[ ] Bakım penceresi planlandı
[ ] Ekip bilgilendirildi
```

### Upgrade Sonrası

```
[ ] Tüm daemon'lar yeni sürümde
[ ] HEALTH_OK durumu
[ ] Client erişimi test edildi
[ ] Performans metrikleri normal
[ ] require-osd-release ayarlandı (opsiyonel)
[ ] Dokumentasyon güncellendi
```

# 🍳 Ceph Senaryo Kitabı (Scenario Cookbook)

Bu doküman, teorik bilgilerden arındırılmış, doğrudan **"Nasıl Yapılır?"** sorusunu cevaplayan, kopyala-yapıştır yapmaya hazır komut reçetelerini içerir.

> **Ön Koşul:** Tüm komutlar `cephadm shell` içerisinde veya `client.admin` yetkisine sahip bir terminalde çalıştırılmalıdır.

---

## 🏗️ Senaryo 1: Sanallaştırma için Disk Alanı (Proxmox / VMware)

**Hedef:** Sanal makineler (VM) için yüksek performanslı, replikasyonlu (3 kopya) bir depolama alanı oluşturmak ve sunucuya tanıtmak.

### Reçete

```bash
# 1. 'vms' adında bir havuz oluştur (128 PG, SSD odaklı olması için önerilir)
ceph osd pool create vms 128 128

# 2. Havuzu RBD (Blok Cihazı) olarak etiketle (ZORUNLU)
ceph osd pool application enable vms rbd

# 3. 100 GB'lık bir sanal disk (Image) oluştur
rbd create vm-100-disk-1 --size 100G --pool vms

# 4. Diski listele
rbd ls -p vms -l

# 5. (Opsiyonel) Diski bu sunucuya bağla (Map)
rbd map vms/vm-100-disk-1
# Çıktı: /dev/rbd0
```

---

## 🚀 Senaryo 2: Veritabanı için Yüksek Performans (SSD/NVMe Only)

**Hedef:** PostgreSQL/MySQL gibi veritabanları için sadece SSD veya NVMe diskleri kullanan, gecikmesi düşük özel bir alan.

### Reçete

```bash
# 1. Özel bir CRUSH kuralı oluştur (Sadece SSD'leri kullansın)
ceph osd crush rule create-replicated rule-ssd default host ssd

# 2. Bu kuralı kullanan 'db_wal' havuzunu oluştur
ceph osd pool create db_wal 32 32
ceph osd pool set db_wal crush_rule rule-ssd

# 3. Havuzu RBD olarak işaretle
ceph osd pool application enable db_wal rbd

# 4. Performanslı disk imajı oluştur
rbd create postgres-wal --size 50G --pool db_wal

# 5. Linux'a bağla ve performans ayarlarıyla mount et
rbd map db_wal/postgres-wal
mkfs.xfs /dev/rbd1
mount -o noatime,nodiratime,discard /dev/rbd1 /var/lib/postgresql/wal
```

---

## 📂 Senaryo 3: Ortak Dosya Paylaşımı (Team Share / NAS)

**Hedef:** 50 kişilik bir yazılım ekibi için 10 TB kotalı, herkesin aynı anda yazabildiği ortak klasör.

### Reçete

```bash
# 1. 'team_fs' adında bir dosya sistemi oluştur
ceph fs volume create team_fs

# 2. (Opsiyonel) Kotayı ayarla (Kök dizine 10 TB limit)
# Önce mount edilmeli veya MDS üzerinden ayarlanmalı. Burada mount ediyoruz:
mkdir -p /mnt/team_share
mount -t ceph 192.168.1.10:6789:/ /mnt/team_share -o name=admin,secret=$(ceph auth get-key client.admin),fs=team_fs

# 3. Kotayı koy (Byte cinsinden 10 TB)
setfattr -n ceph.quota.max_bytes -v 10995116277760 /mnt/team_share

# 4. Kullanıcılar için erişim anahtarı oluştur (Sadece /mnt/team_share dizinine yetkili)
ceph fs authorize team_fs client.developers / rw
```

---

## 💾 Senaryo 4: Ucuz Yedekleme Alanı (Erasure Coding)

**Hedef:** Veeam veya arşiv verileri için diskten tasarruf eden (RAID 6 benzeri) ama biraz daha yavaş bir alan oluşturmak. Veri 3 kopya yerine %66 (4+2) yer kaplayacak.

### Reçete

```bash
# 1. Erasure Code profili oluştur (4 veri + 2 parite = 1 sunucu ölse de çalışır)
ceph osd erasure-code-profile set ec-backup-profile k=4 m=2 crush-failure-domain=host

# 2. Bu profili kullanan 'backups' havuzunu oluştur
ceph osd pool create backups 128 128 erasure ec-backup-profile

# 3. Havuzu RGW (Object Storage) için etkinleştir (EC genelde RGW ile kullanılır)
ceph osd pool application enable backups rgw

# 4. RGW'ye bu havuzu kullanmasını söyle (Zone config gerektirir, basitçe manuel bucket açarken):
# Not: Modern Ceph'te bu işlem Zone Placement ayarlarıyla yapılır.
# Basit kullanım için bu havuzu bir RBD pool gibi de kullanabilirsiniz:
ceph osd pool application enable backups rbd
```

---

## 🔒 Senaryo 5: Ransomware Koruması (WORM / Immutable)

**Hedef:** Yedeklenen verilerin 30 gün boyunca "root" dahil kimse tarafından silinememesi (Fidye yazılımı koruması).

### Reçete

> **Dikkat:** RGW (Object Gateway) kurulu olmalıdır.

```bash
# 1. Object Lock özellikli bir S3 Bucket oluştur (AWS CLI ile)
aws --endpoint-url http://ceph-ip:8000 s3api create-bucket \
    --bucket guvenli-yedek \
    --object-lock-enabled-for-bucket

# 2. Koruma kuralını koy (Compliance Modu = Kimse silemez)
aws --endpoint-url http://ceph-ip:8000 s3api put-object-lock-configuration \
    --bucket guvenli-yedek \
    --object-lock-configuration '{"ObjectLockEnabled": "Enabled", "Rule": {"DefaultRetention": {"Mode": "COMPLIANCE", "Days": 30}}}'

# 3. Test et (Dosyayı silmeyi dene - Hata almalısın)
aws --endpoint-url http://ceph-ip:8000 s3 rm s3://guvenli-yedek/test-dosyasi.txt
# Hata: AccessDenied
```

---

## 📹 Senaryo 6: Güvenlik Kamerası Kayıtları (CCTV)

**Hedef:** Sürekli yazma (Sequential Write) yapılan ve eski kayıtların otomatik silindiği bir alan.

### Reçete

```bash
# 1. Kamera kayıtları için bucket oluştur
aws --endpoint-url http://ceph-ip:8000 s3 mb s3://cctv-kayitlari

# 2. Lifecycle Policy (Yaşam Döngüsü) oluştur - 90 günden eskileri sil
# policy.json dosyası:
# {
#     "Rules": [{
#         "ID": "EskiKayitlariSil",
#         "Status": "Enabled",
#         "Filter": { "Prefix": "" },
#         "Expiration": { "Days": 90 }
#     }]
# }

# 3. Politikayı uygula
aws --endpoint-url http://ceph-ip:8000 s3api put-bucket-lifecycle-configuration \
    --bucket cctv-kayitlari \
    --lifecycle-configuration file://policy.json
```

---

## 💎 Senaryo 7: Enterprise "All-Flash" Deneyimi (High-End Özellikler)

Pure Storage veya NetApp All-Flash dizilerinde alışık olduğunuz o "Premium" özellikleri Ceph'te nasıl yaparsınız?

### A. "Always-On" Sıkıştırma (Compression)

Veriyi diske yazmadan önce havada sıkıştırarak yer tasarrufu sağlar. CPU kullanımı artar ama disk alanı kazanılır (All-Flash için ideal).

```bash
# 1. Havuzda sıkıştırmayı "aggressive" modda aç (Her şeyi sıkıştır)
ceph osd pool set vms compression_mode aggressive

# 2. Algoritmayı seç (lz4 = Hızlı, zstd = Güçlü sıkıştırma)
ceph osd pool set vms compression_algorithm zstd
```

### B. Quality of Service (QoS) - Hız Limitleme

"Gürültülü Komşu" (Noisy Neighbor) sorununu çözmek için. Test sunucusu production'ı yavaşlatmasın.

```bash
# 1. Test diskine saniyede max 500 IOPS limiti koy
rbd config image set vms/test-vm-disk conf_rbd_qos_iops_limit 500

# 2. Veya bant genişliğini (Bandwidth) sınırla (Örn: 50 MB/s)
rbd config image set vms/test-vm-disk conf_rbd_qos_bps_limit 52428800
```

### C. Anında Klonlama (Instant Cloning) - DevOps

10 TB'lık canlı veritabanının kopyasını, geliştiriciler için **saniyeler içinde** ve **sıfır yer kaplayarak** oluşturun.

```bash
# 1. Canlı diskin snapshot'ını al (Saniyeler sürer)
rbd snap create db_wal/postgres-prod@v1

# 2. Snapshot'ı "koru" (Silinmesini engelle)
rbd snap protect db_wal/postgres-prod@v1

# 3. Geliştirici-1 için klon oluştur (Copy-On-Write, anında biter)
rbd clone db_wal/postgres-prod@v1 db_wal/dev-db-1

# 4. Geliştirici-2 için klon oluştur
rbd clone db_wal/postgres-prod@v1 db_wal/dev-db-2
```

*Sonuç: Elinizde birbirinden bağımsız, üzerinde çalışılabilir 2 yeni veritabanı var ve diskte kapladıkları yer başlagıçta 0 KB.*

### D. Otomatik Katmanlama (Tiering) - Hot/Cold Data

Sıcak veriyi pahalı NVMe'de, soğuk veriyi ucuz HDD'de tut. (RGW Bucket Lifecycle ile).

1. **Storage Class Yapılandırması:** RGW üzerinde `STANDARD` (NVMe) ve `COLD` (HDD) depolama sınıfları tanımlanır.
2. **Örnek Politika:**

```json
{
  "Rules": [{
    "ID": "MoveToColdStorage",
    "Status": "Enabled",
    "Filter": { "Prefix": "logs/" },
    "Transitions": [{
      "Days": 30,
      "StorageClass": "COLD"
    }]
  }]
}
```

---

## 🏢 Senaryo 8: VMware vSphere Cluster'a Datastore Sağlama

**Hedef:** VMware ESXi host'larına Ceph üzerinden paylaşılan bir Datastore (veri deposu) vermek. Birden fazla ESXi host'un aynı anda erişebildiği, HA yapılabilir bir disk alanı oluşturmak.

### A. iSCSI Gateway ile (Önerilen Yöntem)

```bash
# 1. iSCSI Gateway için özel havuz oluştur
ceph osd pool create vmware_iscsi 64 64
ceph osd pool application enable vmware_iscsi rbd

# 2. iSCSI Gateway servisini deploy et (3 node minimum önerilir)
ceph orch apply iscsi vmware_iscsi api_user=admin api_password=Passw0rd \
    trusted_ip_list=192.168.1.0/24 \
    placement="node1,node2,node3"

# 3. iSCSI Target ve LUN oluştur (Admin UI'dan veya CLI ile)
# Dashboard'a giriş yap: https://ceph-mgr-ip:8443
# Block > iSCSI > Targets > Create
# - Target IQN: iqn.2024-01.com.ceph:vmware-datastore
# - LUN: vmware_ds1 (2TB boyutunda)
# - Initiator: ESXi host'ların IQN'lerini ekle

# 4. ESXi tarafında (Her host'ta)
# Storage > Adapters > Software iSCSI > Enable
# Dynamic Discovery > Add Server: <ceph-iscsi-gateway-ip>:3260
# Rescan Storage
```

### B. RBD Direkt Mount (Alternatif - Proxmox tarzı)

> **Not:** VMware resmi olarak RBD'yi desteklemiyor, ama Proxmox gibi KVM tabanlı sistemlerde çalışır.

```bash
# Proxmox için:
pvesm add rbd vmware-storage --pool vmware_iscsi --content images
```

---

## 🪟 Senaryo 9: Hyper-V Cluster'a CSV Storage Sağlama

**Hedef:** Windows Hyper-V Failover Cluster için Cluster Shared Volume (CSV) olarak kullanılabilir bir Ceph depolama alanı vermek.

### A. iSCSI Gateway ile (Önerilen)

```bash
# 1. Hyper-V için havuz oluştur
ceph osd pool create hyperv_iscsi 64 64
ceph osd pool application enable hyperv_iscsi rbd

# 2. iSCSI Gateway deploy et (Daha önce yapıldıysa atla)
ceph orch apply iscsi hyperv_iscsi api_user=admin api_password=Passw0rd \
    trusted_ip_list=192.168.2.0/24

# 3. Target ve disk oluştur
# Dashboard > Block > iSCSI > Targets > Create
# - Target IQN: iqn.2024-01.com.ceph:hyperv-csv
# - Disk: hyperv_csv1 (5TB)
# - Initiator: Hyper-V node'larının IQN'lerini ekle

# 4. Windows Hyper-V Node'larında (Her node'da)
# iSCSI Initiator açılır
# Discovery Tab > Discover Portal: <ceph-iscsi-ip>:3260
# Targets Tab > Connect > Enable MPIO
# Disk Management > Online yap
# Failover Cluster Manager > Storage > Add Disk
# "Add to Cluster Shared Volumes" seç
```

### B. SMB 3.0 ile (Alternatif - CephFS üzerinden)

> **Dikkat:** SMB üzerinden Hyper-V performansı, iSCSI'den düşük olabilir. Kritik production için iSCSI tercih edin.

```bash
# 1. CephFS volume oluştur
ceph fs volume create hyperv_smb

# 2. Samba Gateway deploy et (Üçüncü parti araç gerekir)
# Örnek: ceph-deploy veya manuel Samba kurulumu
# /etc/samba/smb.conf:
[hyperv_vms]
   path = /mnt/cephfs/hyperv
   read only = no
   vfs objects = ceph
   ceph:config_file = /etc/ceph/ceph.conf

# 3. Windows'ta
# Hyper-V Manager > Settings > Default Stores
# File Share Path: \\ceph-smb-gateway\hyperv_vms
```

---

## 🎯 Senaryo Özeti: Hangi Yöntemi Seçmeliyim?

| Platform | Önerilen Yöntem | Performans | Kurulum Kolaylığı |
| :--- | :--- | :--- | :--- |
| **VMware vSphere** | iSCSI Gateway | ⭐⭐⭐⭐ | Orta |
| **Hyper-V Cluster** | iSCSI Gateway | ⭐⭐⭐⭐ | Orta |
| **Proxmox VE** | RBD (Native) | ⭐⭐⭐⭐⭐ | Kolay |
| **OpenStack** | RBD (Cinder) | ⭐⭐⭐⭐⭐ | Kolay |

---

## 🌐 Modern İş Yükleri ve Gelişmiş Senaryolar

### 🌍 Senaryo 10: Global Veri Senkronizasyonu (Multi-site RGW)

**Problem:** İstanbul ve Londra'daki iki farklı ekip aynı verilere hızlı erişmeli ve bir lokasyon çökerse diğeri üzerinden devam edilmeli.

**Çözüm:** RGW Multi-site (Active-Active) Replikasyon.

```bash
# 1. Zonegroup ve Zone yapılandırması (Ana Cluster'da)
# realm, zonegroup ve zone oluşturulur
radosgw-admin realm create --rgw-realm=global_realm --default
radosgw-admin zonegroup create --rgw-zonegroup=turkiye --master --default
radosgw-admin zone create --rgw-zonegroup=turkiye --rgw-zone=istanbul --master --default

# 2. Senkronizasyon kullanıcısı oluştur
radosgw-admin user create --uid="syncuser" --display-name="Synchronization User" --system

# 3. İkinci lokasyonda (Londra) bu realm bilgilerini çek ve zone oluştur
radosgw-admin realm pull --url=http://istanbul-ip:8000 --access-key=<ak> --secret-key=<sk>
radosgw-admin zone create --rgw-zonegroup=turkiye --rgw-zone=londra --default

# 4. Değişiklikleri uygula ve restart et
radosgw-admin period update --commit
```

### 🧠 Senaryo 11: AI / Derin Öğrenme Veri Besleme (High-Throughput)

**Problem:** GPU cluster'ları, binlerce küçük dosyadan oluşan devasa dataset'leri (ImageNet vb.) aynı anda çok hızlı okumalı.

**Çözüm:** CephFS Client Tuning ve MDS Ön Bellek Optimizasyonu.

```bash
# 1. MDS Cache boyutunu artır (RAM'iniz bolsa 8GB veya üstü)
ceph config set mds mds_cache_memory_limit 8589934592

# 2. Client tarafında okuma hızını artırmak için readahead ayarı
mount -t ceph <mon_ip>:/ /mnt/ai_data -o name=admin,secret=<key>,rasize=67108864

# 3. Küçük dosyalar için 'lazyio' özelliğini kullanarak eşzamanlılığı artırın (Uygulama desteği gerektirir)
```

### 📊 Senaryo 12: Büyük Veri Log/Analitik Havuzu (Elasticsearch Backend)

**Problem:** Saniyede 50.000 log satırı geliyor. Veri hem güvenli saklanmalı hem de hızlı indekslenmeli.

**Çözüm:** Replikasyonlu Index Havuzu + Erasure Coded Data Havuzu.

```bash
# 1. Metadata/Index için SSD tabanlı Replicated havuz (Hızlı arama)
ceph osd pool create log_index 32 32
ceph osd pool set log_index crush_rule rule-ssd

# 2. Asıl veri (Data) için HDD tabanlı Erasure Coded havuz (Ucuz depolama)
ceph osd pool create log_data 128 128 erasure ec-profile-4-2

# 3. S3 katmanında bu iki havuzu birleştirerek logları saklayın.
```

### 🛠️ Senaryo 13: CI/CD Ephemeral Storage (Hızlı Test Ortamları)

**Problem:** Her yeni 'Build' için 200GB'lık temiz bir OS diski lazım ama saniyeler içinde hazır olmalı.

**Çözüm:** RBD Snapshot ve Hızlı Klonlama.

```bash
# 1. Altın imajı (Gold Image) hazırla ve snap al
rbd snap create vms/ubuntu-22-template@gold
rbd snap protect vms/ubuntu-22-template@gold

# 2. Her yeni Job için anlık klon oluştur (0 saniye sürer)
rbd clone vms/ubuntu-22-template@gold vms/job-id-455-disk

# 3. Test bitince klonu anında sil
rbd rm vms/job-id-455-disk
```

### 📺 Senaryo 14: Media Sunucusu (Plex / Jellyfin / Emby)

**Problem:** Film, dizi ve müzik koleksiyonunuzu tüm evdeki cihazlara (TV, tablet, telefon) akışla (stream) vermek istiyorsunuz. Binlerce video dosyası olacak ve birden fazla kişi aynı anda izleyebilmeli.

**Çözüm:** CephFS ile Yüksek Throughput Media Storage.

```bash
# 1. Media için yüksek performanslı CephFS volume oluştur
ceph fs volume create media_storage

# 2. Media sunucunuza mount edin
mkdir -p /mnt/media
mount -t ceph <mon_ip>:/ /mnt/media -o name=admin,secret=<key>,fs=media_storage

# 3. Klasör yapısı oluşturun
mkdir -p /mnt/media/{movies,series,music,photos}

# 4. (Opsiyonel) Quota koyun (Örn: 20TB)
setfattr -n ceph.quota.max_bytes -v 21990232555520 /mnt/media

# 5. Plex/Jellyfin'i kurun ve media klasörünü gösterin
# Plex Library Settings > Add Library > Browse > /mnt/media/movies
```

**Performans İpuçları:**

```bash
# Yüksek okuma performansı için client tarafında cache artırın
mount -t ceph <mon_ip>:/ /mnt/media -o rasize=67108864,name=admin,secret=<key>

# MDS'lerde cache'i artırın (4K transcoding için)
ceph config set mds mds_cache_memory_limit 17179869184
```

**Neden CephFS?**

- **Çoklu Erişim:** Hem Plex hem de download araçları (Sonarr/Radarr) aynı anda yazabilir.
- **Sınırsız Büyüme:** Koleksiyon büyüdükçe sadece disk ekleyin.
- **Yedekleme Kolay:** CephFS snapshot ile anlık yedek alın.

---

## 🏛️ Mimari Vizyon: Ceph Bir "Veri İşletim Sistemi"dir

Bu reçete kitabındaki 13 senaryoyu tek tek uyguladığınızda, elinizde sadece bir "depolama cihazı" değil, komple bir **Veri Altyapısı (Data Infrastructure)** oluşur.

İşte "Büyük Resim" (The Big Picture):

| Katman | Bileşenler | Ceph Karşılığı |
| :--- | :--- | :--- |
| **1. Uygulama Katmanı** | Oracle, SAP, Kubernetes, AI Model Training, Splunk, Video Editörleri | *Tüketiciler* |
| **2. Protokol Katmanı** | Block (SAN), File (NAS), Object (S3), iSCSI, NFS | **RBD, CephFS, RGW** |
| **3. Veri Hizmetleri** | Sıkıştırma, Şifreleme, Snapshot, Klonlama, Tiering, WORM | **BlueStore Features** |
| **4. Fiziksel Katman** | NVMe, SSD, HDD, Network | **OSD, CRUSH Map** |

**Özetle:**
Artık firmanızdaki herhangi bir proje için "Disk lazım" dendiğinde, "Hangi marka storage alalım?" diye düşünmenize gerek yok. Cevap her zaman aynıdır:
> *"Ceph cluster'ında yeni bir Pool açalım."*

---
**🎉 Tebrikler!** Artık Ceph'i en basit disk ihtiyacından, en karmaşık yapay zeka ve global replikasyon senaryolarına kadar yönetebilecek bir başucu rehberine (Cookbook) sahipsiniz.

# 🚀 Ceph İstemci Kullanımı ve Performans Testi (Client & Benchmark)

Bu doküman, kurduğumuz Ceph kümesini "nasıl kullanacağımızı" ve "hızını nasıl test edeceğimizi" anlatır. Blok (RBD), Dosya (CephFS) ve Nesne (S3) erişim yöntemleri ile hız testi senaryolarını içerir.

---

## 📂 1. CephFS (Dosya Sistemi) Kullanımı

CephFS, tıpkı bir NFS sunucusu gibi çalışır. Birden fazla sunucu aynı klasöre aynı anda yazabilir (Shared File System).

### A. CephFS Volume Oluşturma (Admin Tarafı)

Daha önce kurulumda MDS servisini açmıştık. Şimdi bir dosya sistemi yaratalım:

```bash
# Volume adı: myfs
ceph fs volume create myfs
```

### B. Linux İstemciye Bağlama (Mount)

İstemci makinede (Client) `ceph-common` yüklü olmalıdır.

**1. Secret Key'i Al:**
Admin sunucusunda şu komutla bağlanma yetkisi (key) oluştur ve al:

```bash
ceph fs authorize myfs client.user1 / rw
# Çıktıdaki anahtarı kopyala (Örn: AQAJz................==)
```

**2. Mount Et (Kernel Driver ile):**
> **Not:** En iyi performans ve uyumluluk için Linux Kernel 5.x veya üzerini öneririm.
İstemci makinede:

```bash
mkdir /mnt/mycephfs
# mon_ip: Node1 IP adresi
# secret: Kopyaladığın anahtar
mount -t ceph 192.168.1.10:6789:/ /mnt/mycephfs -o name=user1,secret=AQAJz...==
```

Artık `/mnt/mycephfs` klasörüne yazdığın her şey, arka planda Ceph cluster'ına dağıtılır. `df -h` ile baktığında Petabyte boyutunda bir alan görebilirsin.

---

## ☁️ 2. S3 Object Storage (Rados Gateway) Kullanımı

Ceph'i kendi AWS S3'ünüz gibi kullanabilirsiniz.

### A. Kullanıcı Oluşturma (Admin Tarafı)

Önce bir S3 kullanıcısı yaratalım ve Access/Secret key üretelim.

```bash
radosgw-admin user create --uid=testuser --display-name="Test User"
```

**Çıktıdaki şu değerleri kaydet:**

* `access_key`: (Örn: J8X...)
* `secret_key`: (Örn: 9bN...)

### B. İstemci ile Bağlanma (AWS CLI veya S3 Browser)

AWS CLI yüklü bir makineden test edelim.

**1. AWS CLI Yapılandırması:**

```bash
aws configure --profile myceph
# Access Key ve Secret Key'i gir.
# Region: default (veya us-east-1)
# Output: json
```

**2. Bucket Oluşturma ve Dosya Atma:**
Ceph RGW varsayılan olarak rastgele bir porttan (örn: 8000, 8080) çalışabilir.

```bash
# RGW Portunu öğren:
ceph orch ps --daemon_type rgw
# PORTS sütununa bak (Örn: *:8000)
```

Aşağıdaki komutlarda portu **8000** olarak varsayıyoruz (kendi portunla değiştir):

```bash
# Bucket oluştur
aws --endpoint-url http://192.168.1.10:8000 s3 mb s3://testbucket --profile myceph

# Dosya yükle
aws --endpoint-url http://192.168.1.10:8000 s3 cp deneme.txt s3://testbucket/ --profile myceph

# Listele
aws --endpoint-url http://192.168.1.10:8000 s3 ls s3://testbucket --profile myceph
```

---

## 🏎️ 3. Performans Testi (Benchmark)

"Sistemim kaç MB/s basıyor?" sorusunun cevabı için Ceph'in kendi içinde gelen `rados bench` aracını kullanırız.

### Yazma Testi (Write Benchmark)

Cluster'ın **yazma** kapasitesini ölçer.

```bash
# testpool havuzuna, 10 saniye boyunca, durmaksızın veri yazar.
ceph osd pool create testpool 32 32
rados bench -p testpool 10 write --no-cleanup
```

**Çıktıda şuna bak:** `Bandwidth (MB/sec): [Değer]`

### Okuma Testi (Read Benchmark)

Az önce yazılan verileri ne kadar hızlı **okuyabiliyor**?

```bash
rados bench -p testpool 10 seq
```

### Temizlik

Test bittikten sonra çöp verileri silmeyi unutma:

```bash
rados -p testpool cleanup
```

---

## 🛠️ 4. Gelişmiş Test: FIO (Sanallaştırma Benzeri Yük)

`rados bench` ham disk performansını ölçer. Ancak sanal makineler (VM) rastgele (random) okuma/yazma yapar. Bunu simüle etmek için `fio` kullanılır.

Bir RBD diski (blok cihazı) mount ettikten sonra şu komutu çalıştır:

```bash
# 4k rastgele yazma testi (IOPS ölçer)
fio --name=randwrite --ioengine=libaio --iodepth=1 --rw=randwrite --bs=4k --direct=1 --size=1G --numjobs=1 --runtime=60 --group_reporting --filename=/mnt/myrbd/testfile
```

* **IOPS:** Saniye başına işlem sayısı (Veritabanları için en önemli değer).
* **BW:** Bant genişliği (Büyük dosya transferleri için önemli değer).

---

## 📸 5. RBD İleri Seviye Özellikler

### A. RBD Snapshot

```bash
# Snapshot oluştur
rbd snap create mypool/disk1@snap1

# Snapshot listele
rbd snap ls mypool/disk1

# Snapshot'tan geri yükle (image'ı durdur önce!)
rbd snap rollback mypool/disk1@snap1

# Snapshot sil
rbd snap rm mypool/disk1@snap1

# Tüm snapshot'ları sil
rbd snap purge mypool/disk1
```

### B. RBD Clone (Copy-on-Write)

```bash
# Snapshot'ı korumaya al (clone için zorunlu)
rbd snap protect mypool/disk1@snap1

# Clone oluştur
rbd clone mypool/disk1@snap1 mypool/disk1-clone

# Clone'u bağımsız hale getir (flatten)
rbd flatten mypool/disk1-clone

# Korumayı kaldır (clone silinmeli önce)
rbd snap unprotect mypool/disk1@snap1
```

### C. RBD Resize

```bash
# Büyütme (online yapılabilir)
rbd resize mypool/disk1 --size 20G

# Küçültme (DİKKAT: veri kaybı!)
rbd resize mypool/disk1 --size 5G --allow-shrink
```

### D. RBD Export/Import

```bash
# Dışa aktar
rbd export mypool/disk1 /backup/disk1.raw

# İçe aktar
rbd import /backup/disk1.raw mypool/restored-disk
```

---

## 📁 6. CephFS İleri Seviye Özellikler

### A. CephFS Quota (Dizin Limiti)

```bash
# Dizin için maksimum boyut (1 GB)
setfattr -n ceph.quota.max_bytes -v 1073741824 /mnt/mycephfs/project1

# Maksimum dosya sayısı
setfattr -n ceph.quota.max_files -v 10000 /mnt/mycephfs/project1

# Quota'yı görüntüle
getfattr -n ceph.quota.max_bytes /mnt/mycephfs/project1

# Quota'yı kaldır
setfattr -n ceph.quota.max_bytes -v 0 /mnt/mycephfs/project1
```

### B. CephFS Snapshot

```bash
# Snapshot dizini
mkdir /mnt/mycephfs/.snap/daily-backup

# Snapshot'ları listele
ls /mnt/mycephfs/.snap/

# Snapshot'tan dosya geri yükle
cp /mnt/mycephfs/.snap/daily-backup/myfile.txt /mnt/mycephfs/

# Snapshot sil
rmdir /mnt/mycephfs/.snap/daily-backup
```

### C. NFS-Ganesha (CephFS üzerinden NFS)

NFS-Ganesha, CephFS'i NFS protokolü ile sunmanızı sağlar.

```bash
# NFS-Ganesha servisini dağıt
ceph orch apply nfs myfs-nfs --placement="node1"

# NFS export oluştur
ceph nfs export create cephfs myfs-nfs /cephfs myfs --path=/

# Export'u listele
ceph nfs export ls myfs-nfs

# Client'tan bağlan (Linux)
mount -t nfs4 192.168.1.10:/cephfs /mnt/nfs-ceph
```

### D. FUSE vs Kernel Mount

| Özellik | Kernel Mount | FUSE (ceph-fuse) |
| :--- | :--- | :--- |
| Performans | Yüksek | Orta |
| Stabilite | Kernel bağımlı | Daha esnek |
| Kurulum | Kernel modülü | Userspace |
| Kullanım | Production | Test/Debug |

```bash
# FUSE ile mount
ceph-fuse -m 192.168.1.10:6789 /mnt/mycephfs --client_fs=myfs
```

---

## 🔌 7. iSCSI Gateway (Windows/VMware için)

Ceph RBD disk'lerini iSCSI protokolü ile sunabilirsiniz.

### iSCSI Gateway Kurulumu

```bash
# iSCSI gateway servisini dağıt
ceph orch apply iscsi myiscsi mypool admin admin --placement="node1 node2"

# Dashboard üzerinden yapılandırma:
# https://192.168.1.10:8443 → Block → iSCSI
```

### Windows Client Bağlantısı

1. Windows Server → iSCSI Initiator
2. Target Portal: Ceph iSCSI Gateway IP
3. Target'a bağlan
4. Disk Management'ta diski görünür yap

---

## 🌐 8. RBD Mirroring (Site-to-Site Replikasyon)

İki farklı veri merkezi arasında RBD image'larını senkronize eder.

### Mirroring Modları

| Mod | Açıklama | Kullanım |
| :--- | :--- | :--- |
| `pool` | Tüm pool senkronize | Basit DR |
| `image` | Seçili image'lar | Granüler kontrol |

### Mirroring Kurulumu

**Site A (Primary):**

```bash
# Pool mirroring'i etkinleştir
rbd mirror pool enable mypool pool

# Peer ekle (Site B'nin bağlantı bilgisi)
rbd mirror pool peer add mypool client.admin@site-b --remote-mon-host 10.0.0.10
```

**Site B (Secondary):**

```bash
# rbd-mirror daemon'ı başlat
ceph orch apply rbd-mirror --placement="node1"

# Senkronizasyonu kontrol et
rbd mirror pool status mypool
```

### Failover

```bash
# Primary çöktüğünde, Secondary'de:
rbd mirror image promote mypool/disk1

# Primary düzeldiğinde:
rbd mirror image demote mypool/disk1
rbd mirror image resync mypool/disk1
```

---

## 📊 9. Client Best Practices

### ✅ Yapılması Gerekenler

* Her uygulama için ayrı RBD image kullanın

* Düzenli snapshot alın
* CephFS'te quota kullanın
* Kernel mount tercih edin (production için)
* Mirroring ile DR planı yapın

### ❌ Yapılmaması Gerekenler

* Tek bir büyük image'a tüm veriyi koymayın

* Snapshot'ları çok uzun tutmayın (alan dolar)
* Windows için native kernel RBD client beklemeyin (iSCSI kullanın)
* NFS-Ganesha'yı yüksek performans gereken yerlerde kullanmayın

# 🌪️ Ceph Disaster Recovery (DR) & Backup Stratejisi

Bu doküman, "her şeyin ters gittiği" senaryolar içindir. Standart operasyonların dışında, **veri kaybı riski içeren** ancak hayati kurtarma işlemlerini (Last Resort) kapsar.

> [!WARNING]
> **YASAL UYARI:** Buradaki komutlar (özellikle `ceph-objectstore-tool` ve `monmap` araçları) yanlış kullanıldığında verilerinizi kalıcı olarak yok edebilir. Production ortamında uygulamadan önce mutlaka yedeklerinizi doğrulayın.
>
> **ÖNEMLİ NOT (Cephadm Kullanıcıları):** Bu dokümandaki `ceph-mon`, `ceph-objectstore-tool` gibi düşük seviyeli araçlar host işletim sisteminde yüklü gelmez. Bu komutları çalıştırmak için ilgili daemon'ın konteynerine girmeniz veya `cephadm shell` kullanmanız gerekir.
>
> **Container içine girmek için:**
> `cephadm unit --name mon.node1 --enter`
> veya veri dizinlerini mount ederek shell açmak için:
> `cephadm shell --mount /var/lib/ceph:/var/lib/ceph`

---

## 💾 1. Backup Stratejisi (Yedekleme)

Ceph kendi kendine yedek almaz. Cluster çöktüğünde geri dönebilmek için şu 3 şeyi elinizde tutmalısınız:

1. **Ceph Konfigürasyonları:** `/etc/ceph/` dizini.
2. **Monitor Database (RocksDB):** Cluster haritasının kalbi.
3. **OSD Keyring'leri:** Diskleri tanımak için.

### A. MON Veritabanı Yedeği

Monitor'ler cluster'ın beynidir. Hepsi ölürse cluster biter. Düzenli olarak **en az bir MON'un** yedeğini alın.

```bash
# Servisi durdurmadan yedek almak mümkün değildir!
# Ancak RocksDB snapshot özelliği ile canlı yedek alınabilir (Riskli olabilir).
# En güvenlisi: Bir MON'u durdurup kopyalamaktır.

# 1. Klasik Yöntem (Cold Backup)
systemctl stop ceph-mon@node1
tar -czvf mon-backup-$(date +%F).tar.gz /var/lib/ceph/mon/ceph-node1
systemctl start ceph-mon@node1
```

### B. Konfigürasyon ve Keyring Yedeklemesi

Her node'da değil, admin node'da bu betiği cron'a ekleyin:

```bash
#!/bin/bash
BACKUP_DIR="/backup/ceph-config"
mkdir -p $BACKUP_DIR
tar -czvf $BACKUP_DIR/ceph-etc-$(date +%F).tar.gz /etc/ceph
```

---

## 🧠 2. Monitor Recovery (Quorum Kaybı)

**Senaryo:** Elektrik kesintisi oldu, diskler yandı ve 3 MON'dan 3'ü de açılmıyor veya veritabanları bozuldu (Corrupted). Cluster cevap vermiyor.

**Çözüm:** Elimizde kalan en son çalışan MON verisini bulup tek bir MON ile cluster'ı zorla ayağa kaldırmak.

### Adım 1: Monmap'i Çıkar

Bozuk olan ama diski sağlam MON sunucusuna girin:

```bash
# MON servisini durdur
systemctl stop ceph-mon@node1

# O anki monmap'i dışarı aktar (Konteyner içinde veya cephadm shell ile)
# Yöntem 1: Container içine girerek (Önerilen)
cephadm unit --name mon.node1 --enter

# İçeride komutu çalıştır:
ceph-mon -i node1 --extract-monmap /tmp/monmap
exit

# Yöntem 2: Dışarıdan çalıştırma (Host üzerinden)
cephadm shell --name mon.node1 -- fsid=$(ceph fsid) -- ceph-mon -i node1 --extract-monmap /tmp/monmap
```

### Adım 2: Monmap'i Düzenle

Cluster'ı tek MON ile (mesela node1 ile) kandırarak açacağız.

```bash
# Monmap'i görüntüle
monmaptool --print /tmp/monmap

# Diğer bozuk MON'ları listeden sil (node2 ve node3'ü siliyoruz)
monmaptool --rm node2 --rm node3 /tmp/monmap

# Tek kalan MON'u "inject" ediyoruz
# (Monmap dosyasını container'ın görebileceği bir yere taşıdığınızdan emin olun)
cephadm unit --name mon.node1 --enter
ceph-mon -i node1 --inject-monmap /tmp/monmap
```

### Adım 3: Cluster'ı Başlat

```bash
systemctl start ceph-mon@node1
ceph -s
# Cluster "HEALTH_WARN" ile gelmeli. Sonra diğer MON'ları sıfırdan ekleyebilirsiniz.
```

---

## 🛠️ 3. OSD Recovery (ceph-objectstore-tool)

**Senaryo:** Bir OSD `down` oldu ve tekrar `up` olmuyor. İçindeki verileri kurtarıp başka yere taşımamız lazım ama disk mount edilemiyor çünkü BlueStore kullanıyor.

**Çözüm:** `ceph-objectstore-tool` kullanarak ham veriyi dışarı çekmek.

### Adım 1: Bozuk OSD'yi Hazırla

```bash
systemctl stop ceph-osd@5
```

### Adım 2: Veri Tutarlılığını Kontrol Et (FSCK)

```bash
# Deep fsck (Container içinde)
# Önce OSD container'ına girin veya shell açın
cephadm unit --name osd.5 --enter

# Container içinde:
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-5 \
  --op fsck \
  --type bluestore
```

### Adım 3: PG (Placement Group) Dışa Aktarma

Diyelim ki `2.5f` numaralı PG'de kritik verin var.

```bash
# PG'yi dosyaya yedekle
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-5 \
  --pgid 2.5f \
  --op export \
  --file /tmp/pg-2.5f.export

# Başka bir (sağlam) OSD'ye import et (Örn: OSD 6)
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-6 \
  --op import \
  --file /tmp/pg-2.5f.export
```

### Adım 4: Tek Bir Objeyi (Dosyayı) Kurtarmak

```bash
# Objeyi listele
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-5 --op list

# Objeyi dışarı al (extract)
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-5 \
  --pgid 2.5f \
  '{"oid":"myobject","key":"","snapid":-2,"hash":...}' \
  get-bytes > /tmp/kurtarilan_dosya
```

---

## 🗄️ 4. Metadata Hasarı (CephFS & RGW)

### CephFS Journal Recovery

MDS sunucuları çöktü ve dosya sistemi mount edilemiyorsa:

```bash
# Journal'ı sıfırla (Veri kaybı olabilir, sadece metadata düzelir)
cephfs-journal-tool --rank=myfs:0 journal reset

# Metadata tablosunu tarayıp onar
cephfs-table-tool myfs:0 reset session
cephfs-table-tool myfs:0 reset snap
cephfs-table-tool myfs:0 reset inode
```

### RGW Index Senkronizasyonu

S3 bucket listelemede dosya var ama indirince yok diyor veya tam tersi (Index mismatch).

```bash
# Bucket index'ini kontrol et
radosgw-admin bucket check --bucket=mybucket

# Index'i onar
radosgw-admin bucket check --fix --bucket=mybucket

# Ciddi durumlarda index'i silip yeniden oluştur
radosgw-admin bucket rm --bucket=mybucket --bypass-gc
# (Dikkat: Bucket boş değilse objects kalır ama orphan olur)
```

---

## ☠️ 5. "Data Lost" İlan Etme (Nuclear Option)

Eğer diskler fiziksel olarak yandıysa ve replikalar yetersizse (min_size altına düşüldüyse), cluster G/Ç işlemlerini durdurur. İşlemleri devam ettirmek için "Veriyi kaybettim, devam et" demeniz gerekir.

**Bu işlem veriyi geri getirmez, sadece cluster'ın takılmasını engeller.**

```bash
# 1. Hatalı PG'leri bul
ceph pg dump_stuck

# 2. PG'yi "lost" olarak işaretle
# Eğer PG komple kayıpsa:
ceph pg 2.5f mark_unfound_lost delete

# Veya eski versiyona dön (Revert)
ceph pg 2.5f mark_unfound_lost revert
```

---

## 📜 6. DR Checklist (Afet Anında Yapılacaklar)

1. **Sakin Ol:** Panikle komut girmek daha büyük felakete yol açar.
2. **Durumu Anla:** `ceph -s`, `ceph health detail`, `dmesg` çıktılarına bak.
3. **Donanımı Kontrol Et:** Diskler fiziksel olarak dönüyor mu? Ağ kablosu takılı mı?
4. **Noout Koy:** `ceph osd set noout` ile cluster'ın gereksiz rebalance yapmasını engelle.
5. **Logları Oku:** `/var/log/ceph/` altındaki loglarda "panic", "segfault" ara.
6. **IRC/Mailing List:** Çözemiyorsan Ceph topluluğuna (ceph-users) loglarla sor.

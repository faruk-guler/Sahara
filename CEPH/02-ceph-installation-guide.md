# 🐙 Ceph Storage Cluster: Uçtan Uca Kurulum Rehberi

Bu dokümanı, Ceph Reef (veya Quincy) sürümünü baz alarak, modern standart olan **cephadm (Containerized Deployment)** yöntemiyle hazırladım. Senin Docker ve Linux ağ bilgine güvenerek, sadece "kur geç" değil, "neden böyle yapıyoruz" detaylarına da gireceğim.

> **Not:** Bu kurulum Production-Ready (Canlı Ortam) standartlarına en yakın ev laboratuvarı/küçük işletme senaryosudur.

## 🏗️ 1. Mimari ve Gereksinimler (Planlama)

Ceph kurmadan önce masaya koyman gereken mimari şu olmalı. En az **3 Node (Sunucu)** kullanacağız.

### Donanım Önerisi (Node Başına)

* **CPU:** En az 4 Çekirdek (Replikasyon CPU yer).
* **RAM:** En kritik nokta. OSD (Disk) başına yaklaşık 2-4 GB RAM hesapla. 3 Diskli bir sunucu için 16GB RAM idealdir.
  * **Production Formülü:** `Total RAM = [OSD sayısı] x (4 GB + 8 GB) + 16 GB`
  * Örnek: 3 OSD → 3 x 12 GB + 16 GB = **52 GB** önerilir.
* **Diskler:**
  * 1x SSD/NVMe -> İşletim Sistemi (OS) için.
  * 2x veya daha fazla HDD/SSD -> Ceph Verisi (OSD) için. (**ÖNEMLİ:** Bu diskler ham/RAW olacak, formatlanmayacak, partition olmayacak!)

### Network (İncelik 1)

Ceph'te iki trafik vardır:

1. **Public Network:** İstemcilerin (VM'ler, PC'ler) bağlandığı ağ.
2. **Cluster Network:** OSD'lerin arka planda veri kopyaladığı (Replication/Recovery) ağ.

> **Pro Tip:** Mümkünse bunları fiziksel olarak ayırın veya VLAN ile ayırın. Replikasyon trafiği, public hattı tıkamamalı.

### Örnek Lab Ortamı

* **Node 1 (Admin/Monitor):** 192.168.1.10
* **Node 2 (OSD Node):** 192.168.1.11
* **Node 3 (OSD Node):** 192.168.1.12
* **OS:** Ubuntu 22.04 LTS veya Debian 12 (Rocky Linux da olur ama Debian tabanına aşinasın).

---

## 🛠️ 2. Ön Hazırlık (Tüm Node'larda Yapılacak)

Bu komutları **TÜM** sunucularda çalıştır.

### A. Hostname ve DNS Ayarı

Ceph, node'ları isimle tanır. `/etc/hosts` dosyasına hepsini ekle:

```bash
# /etc/hosts dosyasına ekle:
192.168.1.10 node1.local node1
192.168.1.11 node2.local node2
192.168.1.12 node3.local node3
```

### B. Zaman Senkronizasyonu (Kritik İncelik ⚠️)

Ceph'in toleransı milisaniyelerdir. Saatler kayarsa cluster çöker (Clock Skew hatası).

```bash
sudo apt update && sudo apt install -y chrony lvm2 curl podman
# Podman veya Docker şarttır. Ceph artık konteyner içinde çalışır.
```

### C. Swap'ı Kapatmak (Performans İnceliği)

Ceph RAM'i sever. RAM bitince diske (Swap) yazarsa performans yerle bir olur.

```bash
sudo swapoff -a
# /etc/fstab içinden swap satırını kalıcı olarak sil veya yorum satırı yap (#).
```

---

## 🚀 3. Cluster'ı Başlatma (Sadece İlk Node - Node1)

Artık sadece Node 1 üzerindeyiz. Burası bizim yönetim merkezimiz olacak.

### A. Cephadm ve CLI Araçlarını Yükle

```bash
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/reef/src/cephadm/cephadm
chmod +x cephadm
sudo ./cephadm add-repo --release reef
sudo ./cephadm install ceph-common
```

### B. Bootstrap (Büyük Patlama) 💥

Bu komut; ilk Monitor (MON), ilk Manager (MGR) daemonlarını ayağa kaldırır ve bir Dashboard oluşturur.

```bash
# Public network IP adresini belirtiyoruz.
# Opsiyonel: Eğer cluster trafiğini ayırmak istersen --cluster-network 10.10.10.0/24 ekle.
sudo cephadm bootstrap --mon-ip 192.168.1.10
```

> Bu işlem bittiğinde sana bir **Dashboard URL, Kullanıcı Adı (admin) ve Şifre** verecek. Bunu bir yere not et!
> **Pro Tip - Komut Kısayolu:** Her seferinde `cephadm shell -- ceph` yazmak yerine alias oluştur:
>
> ```bash
> alias ceph='cephadm shell -- ceph'
> # Kalıcı yapmak için ~/.bashrc dosyasına ekle
> echo "alias ceph='cephadm shell -- ceph'" >> ~/.bashrc
> ```

---

## 🔗 4. Diğer Sunucuları Cluster'a Ekleme

Cephadm, diğer sunuculara SSH ile bağlanıp onları yönetir.

### A. SSH Anahtarını Dağıt

Bootstrap işlemi sırasında Node1'de özel bir SSH key oluştu (`/etc/ceph/ceph.pub`). Bunu diğer sunuculara kopyalamalıyız.

```bash
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node2
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node3
```

### B. Node'ları Ekleyin (Orchestrator)

Artık Ceph CLI üzerinden diğer sunucuları havuza dahil ediyoruz.

```bash
# Önce Ceph shell'e gir (Container içine girmek gibidir)
sudo cephadm shell

# Node'ları ekle
ceph orch host add node2 192.168.1.11  # Buraya Management/Public IP adresini yaz.
ceph orch host add node3 192.168.1.12

# Durumu kontrol et
ceph orch host ls
```

> Burada 3 sunucuyu da "Status" sütununda aktif görmelisin.

---

## 💾 5. OSD (Disk) Ekleme ve Depolama Alanı

Şimdi sunucuların üzerindeki boş diskleri Ceph'e tanıtacağız.

### A. Diskleri Kontrol Et

Sistemdeki boş diskleri Ceph görüyor mu?

```bash
ceph orch device ls
```

> Çıktıda `/dev/sdb` gibi diskleri ve "Available: Yes" ibaresini görmelisin. Eğer "Available: No" diyorsa diskte partition veya LVM kalıntısı vardır. `wipefs -a /dev/sdb` ile veya inatçı durumlarda `ceph-volume lvm zap /dev/sdb --destroy` ile silmen gerekir.

### B. Tüm Boş Diskleri OSD Yap (Kolay Yol)

Aşağıdaki komut, tüm node'lardaki tüm boş ve uygun diskleri otomatik olarak OSD yapar:

```bash
ceph orch apply osd --all-available-devices
```

### C. Gelişmiş Disk Yapılandırması (İncelik 🧠)

Eğer "Benim SSD'lerim var, bunları HDD'lerin cache'i (WAL/DB) olarak kullanmak istiyorum" dersen (Hybrid OSD), basit komut yerine Drive Group (YAML) kullanmalısın.

Örnek (Sadece bilgi amaçlı, yukarıdaki komut genelde yeterlidir):

```yaml
service_type: osd
service_id: osd_spec_default
placement:
  host_pattern: '*'
data_devices:
  paths:
    - /dev/sdb
    - /dev/sdc
db_devices:
  paths:
    - /dev/nvme0n1  # HDD'lerin metadata'sı bu hızlı diske yazılır.
```

---

## 🧠 6. Servisleri Dağıtma (MDS, RGW)

Ceph sadece blok (RBD) depolama değildir.

### A. CephFS (Dosya Sistemi - NAS gibi kullanım) için

Metadata Server (MDS) gereklidir.

```bash
# Önce MDS servislerini dağıt
ceph orch apply mds myfs --placement="3 node1 node2 node3"

# Ardından CephFS volume'ü oluştur
ceph fs volume create myfs
```

### B. Object Storage (S3 uyumlu) için

Rados Gateway (RGW) gereklidir.

```bash
ceph orch apply rgw myrgw --placement="3 node1 node2 node3"
```

---

## 🕵️ 7. Ceph'in "İncelikleri" ve Pro İpuçları

Bir sistemci olarak bilmen gereken ve kurulum rehberlerinde yazmayan sırlar:

### 1. PG Calc (Placement Groups)

Ceph, veriyi disklere direkt yazmaz. `Veri -> Obje -> PG (Placement Group) -> OSD` zincirini izler.

* **Sorun:** PG sayısını az verirsen veri dağılmaz, performans düşer. Çok verirsen CPU patlar.
* **Kural:** OSD başına yaklaşık 100 PG hedeflenir.
* Ceph Reef sürümünde `pg_autoscaler on` olarak gelir, yani Ceph bunu senin yerine hesaplar. Bunu sakın kapatma!

### 2. Failure Domain (Hata Alanı)

Varsayılan olarak Ceph "Host" bazlı koruma yapar. Yani 3 sunucun varsa, bir dosyanın 1. kopyası Node1'de, 2. kopyası Node2'de, 3. kopyası Node3'tedir.

* **İncelik:** Eğer tek sunucuya Ceph kurarsan (önermemiştim), Ceph verileri replike edemez ve `HEALTH_WARN` durumunda kalır. Bunu düzeltmek için CRUSH haritasını `osd` seviyesine indirmen gerekir (Production'da yapılmaz).

### 3. Client Keyring

İstemcilerin (Örn: Proxmox veya başka bir Linux sunucu) Ceph'e bağlanması için admin yetkisine ihtiyacı yoktur. Her istemci için ayrı "Keyring" oluştur:

```bash
ceph auth get-or-create client.proxmox mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=vms'
```

### 4. Quorum (Split-Brain)

3 Monitor (MON) sunucusu ile başladık. Eğer 2 sunucu kapanırsa, kalan 1 sunucu "Ben çoğunluk muyum?" diye bakar. Oyların %50'sinden fazlasını alamazsa Cluster kendini kilitler (Read-Only bile olmaz, durur). Buna **Quorum** denir. Veri bozulmasını önlemek için bu davranış hayati önem taşır.

---

## 🏁 8. Test Etme

Kurulum bitti, her şey yeşil. Test edelim.

### Bir Havuz (Pool) Oluştur

```bash
ceph osd pool create testpool 32 32
ceph osd pool application enable testpool rbd
```

### Bir Blok Disk (Image) Oluştur

```bash
rbd create disk1 --size 10G --pool testpool
```

### Diski Map Et (Linux'a bağla)

```bash
rbd map disk1 --pool testpool --name client.admin
# Çıktı: /dev/rbd0
```

### Formatla ve Kullan

```bash
mkfs.ext4 /dev/rbd0
mount /dev/rbd0 /mnt
```

**Tebrikler!** Artık kendi kendine yeten, kendini iyileştirebilen (Self-healing), Enterprise seviyesinde bir depolama kümen var. Dashboard'a (`https://192.168.1.10:8443`) girip o meşhur Ceph grafiğini izleyebilirsin.

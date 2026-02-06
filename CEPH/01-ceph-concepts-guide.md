# 🐙 Ceph Master Guide: Nedir, Ne Değildir, Nasıl Çalışır?

Bu doküman, Ceph depolama sistemini en temelinden en derin mimarisine kadar, bir sistem mühendisinin bilmesi gereken detaylarla anlatmak için hazırlanmıştır. Kurulumdan önce "Neye bulaşıyoruz?" sorusunun tam cevabıdır.

---

## 1. Ceph Nedir?

Ceph; **açık kaynaklı**, **dağıtık (distributed)** ve **yazılım tabanlı (software-defined)** bir depolama platformudur.

En büyük özelliği **"Unified Storage" (Bütünleşik Depolama)** olmasıdır. Yani dünyadaki üç ana depolama yöntemini tek bir kümede (cluster) sunabilir:

1. **Blok Depolama (Block Storage):** Sanal makineler (VM) ve diskler için (AWS EBS veya SAN gibi).
2. **Nesne Depolama (Object Storage):** Büyük, yapısal olmayan veriler için (AWS S3 gibi).
3. **Dosya Sistemi (File System):** Klasör paylaşımı ve POSIX uyumlu erişim için (NAS/NFS gibi).

### Ceph Ne Değildir?

* **Basit bir NAS değildir:** Evdeki QNAP veya Synology gibi "tak-çalıştır" bir cihaz değildir.
* **RAID kullanmaz:** Geleneksel donanım RAID kartlarına (RAID 5, RAID 10 vb.) ihtiyaç duymaz, hatta onlardan nefret eder. Kendi yazılımsal korumasını kullanır.
* **Tek sunuculuk iş değildir:** En az 3 sunucu ile gerçek gücünü gösterir. Tek sunucuda çalışır ama Ceph'in mantığına aykırıdır.

---

## 2. Tarihçe ve Ekosistem

"Bu teknoloji kimin eseri?" diye merak ediyorsanız, işte kısa bir tarihçe:

* **Kurucu:** Sage Weil.
* **Doğuş Yeri:** University of California, Santa Cruz (UCSC). Sage Weil'in doktora tezi olarak başladı (2003-2007).
* **İlk Sürüm:** 2006'da açık kaynak (LGPL) olarak yayınlandı.
* **Şirketleşme:** 2012'de Sage Weil, Ceph'i geliştirmek için **Inktank** şirketini kurdu.
* **Büyük Satın Almalar:**
  * 2014: **Red Hat**, Inktank'i 175 Milyon Dolar'a satın aldı.
  * 2019: **IBM**, Red Hat'i satın alarak teknolojinin en büyük hamisi oldu.
* **İsmin Kökeni:** "Cephalopod" (Ahtapot, mürekkep balığı sınıfı) kelimesinden gelir. Dağıtık kolları olan, merkezi olmayan yapıyı temsil eder. UCSC'nin maskotu "Sammy" (bir Banana Slug) ile karıştırılsa da, logo ahtapot temasını işler.

Bugün Linux çekirdeğinin (Kernel) yerleşik bir parçasıdır ve CERN, Cisco, Bloomberg gibi dev yapılar tarafından kullanılmaktadır.

---

## 3. Temel Mimari: RADOS ve CRUSH

Ceph'in kalbinde **RADOS** (Reliable Autonomic Distributed Object Store) yatar. Her şeyi bu yönetir. Üstteki Blok, Nesne ve Dosya servisleri aslında RADOS'un müşterileridir.

### 🧠 CRUSH Algoritması (Sihirli Değnek)

Geleneksel depolamada "Dosya A nerede?" diye sorulduğunda, merkezi bir veritabanına (Metadata Server) bakılır. Bu darboğaz yaratır.

Ceph ise **CRUSH** (Controlled Replication Under Scalable Hashing) algoritmasını kullanır.

* **Mantık:** Verinin nerede duracağını **hesaplar**, "sormaz".
* İstemci (Client) matematiksel bir işlem yapar ve "Bu dosya Node 3'teki Disk 5'e gitmeli" der.
* Bu sayede merkezi bir darboğaz (bottleneck) olmadan Exabyte'larca veriyi yönetebilir.

---

## 4. Ceph Bileşenleri (Diksiyonu)

Bir Ceph kümesi şu 5 temel parçadan oluşur:

### 1. OSD (Object Storage Daemon) - "İşçiler"

* **Görevi:** Veriyi diske yazan, okuyan, çoğaltan ve disk bozulursa iyileştiren (recovery) servistir.
* **Kural:** Genelde her fiziksel disk (HDD/SSD) için 1 adet OSD servisi çalışır. 10 diskin varsa 10 OSD'n vardır.

### 2. MON (Monitor) - "Beyin Takımı"

* **Görevi:** Kümenin haritasını (Cluster Map) tutar. Kim ayakta, kim çöktü, veri nerede olmalı?
* **Kural:** Tek sayı olmak zorundadır (1, 3, 5). "Quorum" (Oylama) usulü çalışır. Çoğunluk sağlanamazsa (Split-Brain) sistemi kilitler.

### 3. MGR (Manager) - "İstatistikçi"

* **Görevi:** Performans metriklerini toplar, Dashboard'u sunar ve orkestrasyonu sağlar.
* **Kural:** En az 1 aktif, 1 yedek (standby) olması önerilir.

### 4. MDS (Metadata Server) - "Kütüphaneci"

* **Görevi:** *Sadece CephFS (Dosya sistemi)* kullanıyorsan gereklidir. Dosya isimleri, izinler ve dizin yapısını tutar.
* **Not:** Blok ve Object storage için MDS gerekmez.

### 5. RGW (Rados Gateway) - "Tercüman"

* **Görevi:** HTTP isteklerini (S3 veya Swift API) Ceph'in anlayacağı dile (RADOS) çevirir. Object Storage kullanacaksan gereklidir.

---

## 5. Veri Nasıl Korunur? (Replica vs Erasure Coding)

Ceph, verilerinizi kaybetmemek için iki yöntem sunar:

### A. Replikasyon (Varsayılan)

* Verinin kopyasını farklı sunuculara yazar.
* **Örnek (Size=3):** 1 GB veri yazarsan, fiziksel olarak 3 GB yer kaplar.
* **Avantajı:** Çok hızlıdır, iyileşme (recovery) süresi kısadır.
* **Dezavantajı:** Pahalıdır (Disk alanının 3'te 1'ini kullanırsın).

### B. Erasure Coding (EC)

* RAID 5 veya RAID 6'nın matematiksel karşılığıdır. Veriyi parçalar ve parite (koruma) kodları ekler.
* **Örnek (4+2):** Veriyi 4 parçaya böl, 2 tane de koruma parçası ekle. Toplam 6 parça farklı yerlere dağılır.
* **Avantajı:** Verimlidir (Disk alanının %66'sını kullanırsın).
* **Dezavantajı:** Yazma işlemi yavaştır (CPU kullanır), iyileşme süresi uzundur. Genelde arşiv/yedekleme için kullanılır.

---

## 6. Ceph Trafik Akışı (Life of an I/O)

Bir dosya yazmak istediğinde arka planda şunlar olur:

1. **İstemci:** Dosyayı (Object) havuza (Pool) atmak ister.
2. **Hash:** Dosya isminin Hash'ini alır.
3. **PG (Placement Group):** Hash sonucuna göre dosyanın hangi PG'ye (Sanal Kova) gireceğini bulur.
4. **CRUSH Hesaplaması:** "Bu PG şu an hangi OSD'lerde (Disklerde) durmalı?" sorusunu CRUSH algoritması ile hesaplar.
5. **Yazma:** İstemci, birincil (Primary) OSD'ye veriyi yazar.
6. **Replikasyon:** Birincil OSD, veriyi alır ve diğer 2 kopya OSD'ye (Secondary) gönderir (varsayılan size=3 için).
7. **Onay (Ack):** Diğer 2 OSD "Yazdım" dediğinde, Birincil OSD istemciye "İşlem Tamam" der.
    * *Bu sayede veri tutarlılığı (consistency) %100 garanti altına alınır.*

---

## 7. Kullanım Senaryoları ve Mimari Yaklaşım

Ceph'in gücü, 3 farklı depolama teknolojisini tek bir platformda sunmasından gelir. Gerçek dünyada bu teknolojiler şöyle kullanılır:

### 🅰️ 3 Temel Depolama Türüne Göre Kullanım

| Tür | Protokol | Gerçek Dünya Senaryoları |
| :--- | :--- | :--- |
| **Block Storage** | **RBD** | • Sanal Makineler (Proxmox, VMware, OpenStack)<br>• Kubernetes StatefulSets (PostgreSQL, Kafka)<br>• Fiziksel sunuculara ek disk (iSCSI/RBD) |
| **File Storage** | **CephFS** | • Ortak çalışma alanları (Departman paylaşımları)<br>• Medya işleme (Render farm)<br>• Log toplama (ELK Stack)<br>• HPC Cluster'ları |
| **Object Storage** | **RGW (S3)** | • Yedekleme (Veeam, Restic)<br>• Statik Web İçeriği (CDN Origin)<br>• Big Data (Spark/Hadoop)<br>• Arşiv (WORM) |

### 🅱️ Sektörel Kullanım Örnekleri

1. **Bulut Sağlayıcılar & ISP:**
    * *Kullanım:* IaaS altyapısında müşterilere disk satmak.
    * *Neden:* Multi-tenant yapı ve izolasyon yeteneği.

2. **Finans & Bankacılık:**
    * *Kullanım:* Değiştirilemez yedekler (Immutable Backup).
    * *Neden:* S3 Object Lock (WORM) ile yasal uyumluluk ve Ransomware koruması.

3. **Medya & Eğlence:**
    * *Kullanım:* 4K/8K video düzenleme havuzu.
    * *Neden:* CephFS ile yüzlerce editörün aynı anda ham görüntülere erişebilmesi.

4. **Kamu & Güvenlik:**
    * *Kullanım:* MOBESE / Güvenlik kamerası kayıtları.
    * *Neden:* Petabyte ölçeğinde maliyet etkin depolama.

### 🌐 Modern Trendler (2024-2026)

* **Kubernetes-First:** Rook operatörü ile Ceph'in tamamen K8s içinde yönetilmesi.
* **Edge Computing:** Şubelerde küçük Ceph cluster'ları, merkezde devasa cluster (Replikasyon ile).
* **AI/ML Pipeline:** Eğitim verisetlerinin CephFS üzerinden GPU sunucularına beslenmesi.
* **Green Storage:** Erasure Coding (EC) kullanımıyla %50 enerji tasarrufu.

### ⚠️ Ceph Nerede KULLANILMAMALI?

Ceph "her derde deva" değildir. Şu senaryolarda geleneksel çözümler daha iyidir:

* **❌ Ultra-Düşük Gecikme (HFT):** Yüksek frekanslı borsa işlemleri gibi *mikrosaniye* hassasiyeti gereken yerler (NVMe-oF veya local disk kullanın).
* **❌ Mikro Kurulumlar:** Sadece 2-3 sunucu ve basit dosya paylaşımı için Ceph'in bakım yüküne değmez (TrueNAS/Samba kullanın).
* **❌ Windows-Native Ortamlar:** Eğer %100 Windows ve Active Directory odaklıysanız, SMB desteği için ek katmanlar gerekir (Native Windows Server daha az baş ağrıtır).

### 6️⃣ Özet: "Herkesin Verisi Tek Yerde"

## 8. Özet

Ceph, donanım bağımsızlığı sunan, kendi kendini yönetebilen ve iyileştirebilen **"Geleceğin Depolama teknolojisidir"**. Öğrenme eğrisi diktir (zordur), ancak bir kez kavradığınızda veri merkezinizin en güvenilir parçası olur. Donanım bozulur, diskler yanar, sunucular çöker; ama **Ceph hayatta kalır.**

> **Sıradaki Adım:** Teori bittiyse, gerçek hayatta nasıl kullanacağınızı görmek için **[14-ceph-scenario-cookbook.md](14-ceph-scenario-cookbook.md)** dosyasındaki 14 Farklı Senaryoyu inceleyin.

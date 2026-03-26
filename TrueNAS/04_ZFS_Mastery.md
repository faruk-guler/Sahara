# Modül 04: ZFS Mastery (VDEV, Pools) ve Disk Mimarisi Doğruları

Sistem artık çalışıyor ve ağda ulaşılabilir durumda. "Storage" (Depolama) -> "Pools" (Havuzlar) sekmesine tıkladığınızda karşınıza on binlerce dolarlık verinizi emanet edeceğiniz ZFS yaratma ekranı gelir. Bu modül ZFS'in alt beynini ve felaket önleme kurgularını anlatır.

## 1. Terminoloji: Pool (Havuz) ve VDEV (Sanal Aygıt)

Birçok acemi yönetici 10 tane diski seçip doğrudan "Havuz (Pool)" yapıyorum diyerek sistemi kurar. ZFS böyle çalışmaz.

- **Disk / Drive:** En alt fiziksel seviye katmandır (Örn: Seagate 16TB).
- **VDEV (Virtual Device):** Disklerin mantıksal olarak birleştiği Raid yapısı kümesidir (Örn: 5 disklik bir VDEV, 2 disklik başka bir Mirror VDEV). **ZFS VDEV seviyesinde hata tolere eder (Redundancy sağlar).**
- **ZPool (Havuz):** Birden fazla VDEV'in birleşip (Stripe edilip) en tepede tek bir Harddisk gibi (Örn: `Tank` isminde) görünmesini sağlayan üst yığına denizdir.

**Kritik Kural:** Eğer tek bir Zpool oluştururken içerisine 3 farklı VDEV eklerseniz ve bu VDEV'lerin SADECE BİRİ arızalanıp çökerse (Tüm yedek diskleri biterse), O BOZUK VDEV BÜTÜN ZPOOL'u GÖTÜRÜR. Tüm ZPool'unuz (Diğer sağlam olan VDEV'ler dahil) yok olur. Havuzun gücü, içindeki en zayıf VDEV kadardır!

## 2. ZFS Çalışma Seviyeleri (RAID Karşılıkları)

ZFS donanım RAID seviyelerinden (RAID 5, 10, 6) farklı isimlendirmeler ve çok daha zeki bir checksum/parity algoritması kullanır.

### 2.1 Stripe (Stripe / RAID-0 Karşılığı)

- Hiçbir koruma (Parity) yoktur. 1 disk bile bozulsa tüm veri ölür. Hız muazzamdır. Geçici/Test ortamları harici kesinlikle yasaktır.

### 2.2 Mirror (Tam Aynalama / RAID-1 veya RAID-10 Karşılığı)

- Diskin tam kopyasının diğerlerinde olmasıdır. (2 Yönlü Mirror veya 3 Yönlü Mirror).
- IOPS (Saniye Başına İşlem Hızı) arayan Veritabanları (Databases) ve Sanal Makine iSCSI depoları için tartışmasız tek seçenektir.
- 10 diski 2'şerli (ikili) 5 Adet `Mirror VDEV` yaparsanız, her VDEV'den 1 disk yanma (Toplam 5 disk yanma) garantiniz olur ve sarsılmaz bir hız elde edersiniz (Kapasitenizin tam yarısı (%50) çöpe gider).

### 2.3 RAID-Z1 (Eski Uygulama: RAID-5 Karşılığı)

- Minimum 3 disk gerektirir. Sadece **1 disklik hata payı** verir.
- **Neden Ölü?** Modern diskler 16TB, 20TB boyutundadır. Z1 havuzunda bir 16TB disk bozulduğunda çıkarıp yerine sıfır disk taktığınızda "Resilvering (Tekiden Dağıtım / Düzeltme)" işlemi başlar. Kalan mekanik disklerin okuma hızından dolayı ZFS'in o 16TB'ı doğrulaması "Günler" sürer. O günler boyunca sağlam kalan ve stres altında tam güç (%100 IO) çalışan diğer dev diskin de bozulması (URE - Unrecoverable Read Error) an meselesidir. 2. disk bozulduğu saniye tüm poolunuzu kaybedersiniz. (Z1 sadece SSD pool'larda veya çok küçük <2TB disklerde kabul edilebilir).

### 2.4 RAID-Z2 (RAID-6 Karşılığı - Depo Standartı)

- Minimum 4 disk gerektirir. Tam bağımsız **2 disklik hata payı** verir.
- Güvenlidir (Resilvering esnasında 2. bir disk ölse bile 3. canınız olduğu için veri kurtulur). Kapasiteden sadece 2 disk kaybedersiniz (Data Capacity vs Striping Oranı). Ancak IOPS performansı çok düşüktür. Sadece şirket dosya deposu ve Video/Film Kurgu (Sequential Data) arşivi olarak kullanılmalıdır. Veritabanı (VM) buraya atılmaz.

### 2.5 RAID-Z3 (Mega Arşiv)

- Minimum 5 disk, 3 disk hata toleransı. 20 diskten fazla VDEV oluşturan Mega DataCenter yığınlarında, derin ve yavaş yedek (Backup Target) olarak kullanılır.

### 2.6 dRAID (Distributed RAID) / 2026 Kurumsal Yeniliği

- Klasik RAID-Z'de yedek (Hot Spare) disk boşta uykuda bekler. Disk arızalanınca veri günlerce o tek yavaş yedek diske yazılmaya çalışılır.
- ZFS **dRAID** teknolojisinde: Parity (Hata kodu) ve boş alan (Spare) bir diske ayrılmaz, 90 disklik devasa havuzun "Bütün disklerinin içindeki dilimlere" mikro seviyede dağıtılır. Her disk aynı anda rebuild okuyup yazarak 48 Saat sürecek bir Resilvering (Kurtarma) işlemini 2-3 saate indirger! Enterprise firmaların 100+ disklik yığınlarında (JBOD) standarttır.

## 3. Scrubbing (Veri Temizliği / Fırçalama)

ZFS veriyi yazdığında Checksum'ını (Özet değerini) ayrı bir ağaca yazar. Sessiz bit çürümesini (Magnetic Rot) anlamak için dosyanın kullanıcı tarafından tesadüfen açılmasını beklemek felakettir.
TrueNAS "Scrub Task" oluşturur (Örn: Ayda 1 Kez çalışacak cron job).

**Görev:** Scrubbing başlatıldığında ZFS, bütün VDEV içerisindeki TB'larca veriyi baştan sona okur, orijinal Checksum ile karşılaştırır. 0 ile 1 arasına sızmış bir yanlışlık bulduğu anda alarmı hiç çalmadan diğer Parity bloğundan (veya Mirror diskten) doğru 1'i çeker ve çürümüş bloğu anında Overwrite ile ezer ve onarır. IT yöneticisi sadece raporlarda "2 Blok Onarıldı" yazar, sistem mükemmel sağlığına devam eder.

---
[Önceki Modül: Modül 03](./03_Kurulum_Network.md) | [Sonraki Modül: Modül 05 - Dataset, Zvol ve Optimizasyon](./05_Dataset_Zvol_Islemleri.md)

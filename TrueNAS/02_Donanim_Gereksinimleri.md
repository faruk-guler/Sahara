# Modül 02: Donanım Anatomisi ve Strict (Kesin) Kurallar

TrueNAS (ZFS), donanım hatalarını affetmez. Çünkü ZFS "Tüm verinin güvenliğinden ben sorumluyum" der. Bu yüzden, rastgele bir bilgisayar toplayıp TrueNAS kurduğunuzda, en ufak bir bileşen zafiyeti yüzünden Petabyte'larca veriniz saniyeler içinde kilitlenebilir (Pool Faulted).

## 1. HBA Kartı (Host Bus Adapter) ve "IT Mode" Zorunluluğu

ZFS'in 1 Numaralı Altın Kuralı: **ZFS ile diskler arasına ASLA Donanımsal RAID kartı koymayın!**

Elinizde HP SmartArray veya Dell PERC serisi bir RAID Controller varsa, diskleri RAID-0 veya RAID-5 yapıp işletim sistemine tek disk gibi gösterirsiniz. ZFS tek bir disk görür. Arka planda donanım diski kontrol eder. Disk bozulduğunda ZFS'in bundan haberi olmaz, ZFS'in Kendi Kendini Onarma (Self-Healing) mekanizması çuvallar.

**Çözüm: IT Mode (Initiator Target)**
LSI (Broadcom) çipli (Örn: LSI 9211-8i, 9300-8i) kartlar alınır. Bu kartlara "IT Mode Firmware" yüklenerek veya native HBA kartlar kullanılarak diskler "Saf ve aptal (Pass-through)" bir şekilde işletim sistemine iletilir.

- 8 adet diskiniz varsa TrueNAS `sda, sdb, sdc...` olarak 8 farklı disk görmelidir.

## 2. RAM: ECC (Error-Correcting Code) Gerçekten Şart Mı?

Bu konu internetteki en büyük tartışmalardan biridir.

- **Non-ECC (Standart) RAM:** Bir ev NAS'ı (Film, dizi barındıran) kuruyorsanız kullanılabilir ancak risklidir. RAM'e bir kozmik ışın çarpıp veriyi bozduğunda ZFS bunu bozuk bir veri olarak algılamaz çünkü "Ben zaten RAM'e yolladım" der ve yanlış veriyi diske kalıtsal olarak kazıyıp checksumları günceller (Scrub of Death).
- **ECC RAM (Enterprise Standardı):** Bir işletme (Database, VM, kritik evraklar) barındıracaksanız ECC RAM **Zorunludur**. ZFS, RAM'i birinci derece önbellek (ARC) olarak kullanır. Bütün veri akışı RAM üzerinden diske iner.

**RAM Miktarı Kuralı:** (Rough Rule of Thumb)
"Her 1 TB Depolama alanı için 1 GB ECC RAM." Bu kural ev kullanıcıları için biraz esnemeye başlamış olsa da, özellikle Deduplication (Tekilleştirme) kullanacaksanız asgari kuraldır.

## 3. Diskin Kalbi: CMR vs SMR Ayrımı

Harddisk (HDD) alırken RPM (7200) ve kapasiteye (18TB vb.) bakarız. Ancak TrueNAS mimarisinde en hayati soru şudur: "Yazma Teknolojisi Nedir?"

- **CMR (Conventional Magnetic Recording):** İzler birbirinden ayrıdır. Veri sırayla hatasız ve hızlı bir şekilde yazılır. TrueNAS için **Kusursuz ve Zorunlu** teknolojidir (WD Red Plus/Pro, Seagate IronWolf serileri gibi).
- **SMR (Shingled Magnetic Recording):** İzler çatı kiremiti gibi birbirinin üzerine biner. Evde arşiv yapanlar için harikadır ama ZFS için **Kanserojen**dir. Bir disk bozulup yerine yenisini taktığınızda ZFS (Resilvering - Senkronizasyon) işlemine başlar. SMR disk saniyede 1-2 MB'a kadar yavaşlar, 16TB'lık bir ZFS Pool'unun toparlanması (Rebuild) Aylar sürer.

## 4. Boot Sürücüleri (İşletim Sisteminin Kurulacağı Diskler)

Geçmişte (FreeNAS zamanında) işletim sistemi USB bellek (Flash Drive) üzerine kurulurdu çünkü sistem sadece Boot anında USB'ye okuma yapar, sonrasında tamamen RAM'de çalışırdı.
Ancak TrueNAS SCALE ve modern CORE ile birlikte sistem çok fazla log yazar (Syslog dataset) ve Swap dosyasını boot diskine atar.

**Yeni Kural:**
Boot diski olarak asla USB bellek kullanmayın (Çok kısa sürede RAW olacak ve çökecektir). Asgari olarak ucuz, küçük boyutlu (Örn: 120GB) 2 adet SATA veya NVMe SSD kullanın ve Sistem Kurulumunda (03. Modül) bunları "Mirror (RAID-1)" olarak kurun.

## 5. ZFS Özel Önbellek ve Hızlandırıcı Diskler (Opsiyonel)

Kritik ve devasa hız (IOPS) gerektiren ortamlarda Array (Disk yığınını) hızlandırmak için şu eklemeler yapılır:

- **SLOG (ZIL - ZFS Intent Log):** Sisteme aniden yüksek yoğunluklu Senkron (Synchronous) yazma işlemi (örneğin NFS üzerinden VMware VM diskleri veya Veritabanı transactionları) geldiğinde hızını RAM'den sonra disklerden önce karşılayan NVMe disklerdir. Elektrikkesintisinde güç kaybını önlemek için PLP (Power Loss Protection) destekli Enterprise SSD (Örn: Intel Optane) olmalıdır.
- **L2ARC (Level 2 ARC):** RAM (ARC) dolduğunda, verilerin çöpe gidip yavaş HDD'lerden tekrar okunması yerine "Isınmış/Sık kullanılan" verilerin tutulduğu SSD Read-Cache (Okuma Önbelleği) diskidir. RAM miktarınız Max (Örn: 256GB veya 512GB) olmadan L2ARC eklemek performansı DÜŞÜRÜR (Çünkü L2ARC'nin indeksi de RAM'de yer kaplar).
- **Special Metadata VDEV:** ZFS, 150 milyon adet küçük resim dosyasının sadece "İsim, Boyut ve Yol (Metadata)" bilgilerini aramak için bile ağır mekanik diskleri döndürür ve saniyeler kaybeder. Special VDEV adında bir SSD Mirror kümesi yapıp bunu Pool'a eklerseniz, ZFS tüm dizin ağacını ve küçük yapılı (Örn: <1M) blockları saf NVMe SSD'lere alır. `ls -la` yazdığınızda klasör ekrana milisaniyede dökülür.

---
[Önceki Modül: Modül 01](./01_Giris_ve_Mimari.md) | [Sonraki Modül: Modül 03 - Kurulum ve Ayarlar](./03_Kurulum_Network.md)

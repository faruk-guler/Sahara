# Modül 01: TrueNAS Mimarisini Anlamak ve CORE vs SCALE Savaşı

Veri depolama (Storage) dünyasında geleneksel donanım tabanlı RAID kartları ve kapalı kutu (Vendor lock-in) NAS cihazları artık miadını doldurdu. TrueNAS, ev kullanıcılarından devasa Veri Merkezlerine kadar "Software-Defined Storage" (Yazılım Tanımlı Depolama) ve OpenZFS gücünü getiren bir devrimdir.

## 1. Neden TrueNAS? Sadece Bir "Dosya Sunucusu" mu?

Klasik bir Windows Server üzerine açılan "Paylaşım" klasörü veya ev tipi 2 diskli standart NAS cihazları, elektrik kesildiğinde verinizin bozulmasını (Silent Data Corruption) engelleyemez.
TrueNAS'in arkasındaki asıl güç İşletim sistemi değil, sahip olduğu **OpenZFS (ZFS)** dosya sistemidir. ZFS; dosya sistemi, mantıksal birim yöneticisi (LVM) ve RAID kontrolcüsünü tek bir potada eritir.

**TrueNAS'in Göz Alıcı Özellikleri:**

- **Copy-On-Write (CoW):** Veri hiçbir zaman kendi üzerine (Overwrite) yazılmaz. Elektrik gitse bile sistem ya yeni veriyi yazar, ya da eski veriyi korur; asla yarıda kalmış bozuk (Corrupted) veri üretmez.
- **Bit-Rot (Sessiz Veri Çürümesi) Koruması:** Harddisklerde yıllar geçtikçe `1`lerin `0`a dönüşmesi olayını kendi Checksum (Sağlama toplamı) mekanizması ile tespit eder ve bozuk veriyi diğer sağlam diskten okuyarak anında onarır (Self-Healing).
- **RAM is the New Cache (ARC):** TrueNAS, sunucunuzun RAM'ini doğrudan devasa ve aşırı hızlı bir SSD önbelleği gibi kullanır (ARC - Adaptive Replacement Cache).

## 2. TrueNAS Sürümleri ve Mimari Farklılıkları (2026 Gerçekleri)

Uzun yıllar boyunca TrueNAS denildiğinde akla **FreeBSD** işletim sistemi gelirdi. Ancak modern IT altyapısı, Linux ve Konteyner mimarisine (Kubernetes/Docker) kayınca mimari kökten değişti.

### 2.1 TrueNAS CORE (Eski Adıyla FreeNAS)

- **Taban:** FreeBSD işletim sistemi.
- **Özelliği:** Rock-solid (Kaya gibi sağlam), yıllardır test edilmiş en stabil ZFS kodunu barındırır. Sanallaştırma olarak "Bhyve", konteyner olarak ise "Jails" (FreeBSD Jails) kullanır.
- **Kaderi (2026 İtibariyle):** iXsystems (TrueNAS'i üreten şirket) CORE sürümünün yavaş yavaş "Bakım (Maintenance)" moduna girdiğini, yeni devrimsel özelliklerin buraya gelmeyeceğini duyurmuştur. Yeni kurulumlarda kesinlikle **Önerilmez**. Sadece eski tip, saf depolama amacı güden ve Docker/K8s ihtiyacı olmayan sistemlerde tutulur.

### 2.2 TrueNAS SCALE (Geleceğin Standartı)

- **Taban:** Debian Linux.
- **Özelliği:** İsmindeki SCALE kelimesi "Scale-Out" mimarisinden (Yatay Büyüme - GlusterFS Clustering vb.) gelir.
- **Modern Ekosistem:** Linux tabanlı olduğu için donanım uyumluluğu (Yeni nesil ağ/ekran kartları) muazzamdır.
- **Sanallaştırma & Uygulamalar:** KVM (Kernel-based Virtual Machine) hypervisor'u ile Windows/Linux sanal makinelerini (VM) rakipsiz performansla çalıştırır. Uygulamalar (Apps) kısmında ise geçmişte K3s (Kubernetes) kullanan sistem, performans sorunlarından dolayı en güncel (Electric Eel ve sonrası) sürümlerde doğrudan **Native Docker (Docker Compose)** altyapısına geçiş yaparak Container dünyasını ZFS ile evlendirmiştir.

### 2.3 TrueNAS Enterprise

Tamamen ticari, donanımla birlikte (Dual-Controller High Availability vb.) satılan işletme sürümüdür. Özünde SCALE veya CORE kodunu kullanır ancak iXsystems tarafından donanımsal destek ve SLA garantisi sunulur.

## 3. Storage Ekosisteminde ZFS'in Yeri

Standart bir EXT4 veya NTFS dosya sisteminde, veritabanınızı tutarken alt katmana (Donanımsal bir RAID kartı veya LVM) güvenirsiniz. Dosya sisteminin RAID kartının diskleri nasıl dağıttığından haberi yoktur (Kör uçuş).

**ZFS'de ise Mimar (Architect) ZFS'in kendisidir:**
ZFS, disklere doğrudan konuşur (Direct Access). SMART verilerini ZFS okur, hangi diskin ne kadar hızlı olduğunu ZFS hesaplar. Arada hiçbir "Donanımsal RAID (MegaRAID vb.)" kartının olmasına tahammül edemez. Bu yüzden TrueNAS kurmadan önce Donanım mimarisini anlamak, sistemin çökmemesi için en büyük zorunluluktur. (Modül 02'ye geçiniz).

---
[Sonraki Modül: Modül 02 - Donanım Gereksinimleri](./02_Donanim_Gereksinimleri.md)

# Modül 03: TrueNAS Kurulumu ve Ağ (Network) Temelleri

Doğru donanım mimarisini oluşturduktan sonra, sıra TrueNAS SCALE'i (2026 kurumsal tavsiye) izole bir depolama canavarı olarak devreye almaya gelmiştir. Bu modül kurulum aşamasını ve sistemin beyni konumundaki Ağ konfigürasyonlarını içerir.

## 1. Kurulum Öncesi Hazırlık ve İmaj Yazdırma

1. TrueNAS resmi sitesinden (truenas.com) en güncel `TrueNAS SCALE .iso` kalıbı indirilir.
2. Rufus veya BalenaEtcher yardımıyla boş bir USB belleğe imaj "DD" modunda yazdırılır.
3. Sunucuya USB takılır ve BIOS/UEFI üzerinden CSM uyumluluğu kapatılarak saf "UEFI Boot" aktif edilir. Ağ (Network) ayarlarının hız sınırlarına takılmaması için SR-IOV ve IOMMU (VT-d / AMD-V) sanallaştırma destekleri mutlaka "Enabled" yapılmalıdır.

## 2. Kurulum Ekranı ve Kritik Adımlar (Setup)

Kurulum aşaması metin bazlı (Console) bir ekrandır. Gözden kaçırılmaması gereken adımlar:

- **Install/Upgrade Seçimi:** `Install` seçilir.
- **Disk Seçimi (Çok Önemli):** ZFS havuzunuzun (Örn: 8x 16TB HDD) bulunduğu sistemde, kurulum programı size "İşletim sistemini HANGİ SÜRÜCÜYE (Boot Drive) kurayım?" diyecektir.
  - *Asla depo disklerini seçmeyin!* Zira ZFS o diskin tamamını kendine rezerve edip depolar için alanı %100 kapatacaktır.
  - Sadece Boot için ayırdığınız 2 Adet 120GB SSD'yi Space tuşu ile işaretleyin.
  - Birden fazla disk seçtiğiniz için sistem "Bunu Mirror (Yedekli) yapayım mı?" diyecektir; kesinlikle "Evet" demelisiniz. Biri yansa bile işletim sistemi diğeriyle yoluna kayıpsız devam eder.
- **Admin Parolası:** Güçlü bir şifre (GUI ve SSH erişimi için) belirlenir. Yada doğrudan Admin kullanıcısını seçmek yerine "admin" isimli standart yetkili kullanıcı hesabı üzerinden devam edilmesi sağlanır (Yeni sürümlerde *root* olarak girmek web UI üzerinden kısıtlanmıştır).
- Kurulum tamamlandığında sunucu yeniden başlatılır ve ekrana bir "IP Adresi" gelir: `http://192.168.1.50/`

## 3. İlk Web Arayüzü Eğitimi ve Temel Ayarlar

Ekranda yazan IP adresiyle tarayıcıya girdiğinizde TrueNAS'ın Dashboard'uyla karşılaşırsınız (Dashboard Modül 13'e benzemez ancak widget bazlı takip sistemi sunar).

**Devreye Alma Parametreleri (First-Boot Tasks):**

1. **Network (Ağ) -> Global Configuration:**
   - Hostname (Örn: `Storage-Master-01`)
   - Domain (Örn: `corp.local`)
   - Ağ geçidi (Gateway) ve DNS IP'leri girilerek "Save" tuşuna basılır.
2. **System Settings -> General:** Timezone, bölge veya ofisin yerel saatine (Örn: `Europe/Istanbul`) ve Localized Date formatına göre düzenlenir (Logların zaman tutarlılığı ve Snapshot/Replika görevleri için NTP saniye şaşmamalıdır).

## 4. Gelişmiş Network (Ağ) Konfigürasyonları

Depolama saniyede GigaBaytlarca (GB/s) veri pompalar, dolayısıyla standart bir 1 Gbps port ZFS'i boğar. Kurumsal mimari şunları emreder:

### 4.1 LACP (Link Aggregation - Bağlantı Kümeleme)

Eğer sunucunuzda 2 veya 4 adet (Örn: 10GbE SFP) port varsa Network -> Interfaces menüsü altından:

- `Add` -> `Link Aggregation`
- **LACP (802.3ad):** Bağlanan Switch'in LACP desteklemesi koşuluyla, iki kartı mantıksal olarak birleştirir (Bond). Ağ performansını hem yük dengeleme (Load Balancing) hem de yedeklilik (Failover) ile ikiye/dörde katlar. A portu koparsa Switch B portundan depolamayı ayakta tutmaya devam eder.

### 4.2 VLAN Tagging (Mantıksal Segmentasyon)

Bir NAS cihazını şirket ağına (Herkesin IP aldığı havuza) açık bırakmak facia (Ransomware kurbanı olmak) demektir.
Network ayarlarına gelerek:

- Yönetim (Management) Arayüzü: VLAN 10
- iSCSI (Sanal Makine Diskleri) Ağı: VLAN 20
- SMB Kullanıcı Paylaşımı (Windows File Sharing): VLAN 30

Bir Interface altına (Örn: `bond0`) bu VLAN ID'leri (Subinterface) olarak eklenir. Şirketin Switch'i ve Firewall'u üzerinden güvenlik tüneli kusursuz parçalanır.

### 4.3 MTU 9000 (Jumbo Frames) - Performans Artışı

Tırların (Disklerin) taşıdığı kasa boyutu (Frame) varsayılan (Default) olarak 1500 byte'tır. Özellikle 10/40/100 GbE (Fiber) ağlarda, iSCSI ve NFS ile devasa VM dosyaları taşırken her paketi 1500 byte'a çekmek CPU'ya kesilmeler (Interrupts) yaşatır.

**Ağ kartı limitini kırma:**
TrueNAS Network UI üzerinden Interface MTU değeri `9000` (Jumbo Frame) olarak set edilir. (Bunun çalışması için Ağ Anahtarınızın / Switch ve Karşı Sunucunun (Client/ESXi) da MTU 9000 desteklemesi ve açılması zorunludur. Aksi halde paketler Droplanıp tamamen kesintiye sebep olur!).

---
[Önceki Modül: Modül 02](./02_Donanim_Gereksinimleri.md) | [Sonraki Modül: Modül 04 - ZFS Havuzu ve Master Derecesi](./04_ZFS_Mastery.md)

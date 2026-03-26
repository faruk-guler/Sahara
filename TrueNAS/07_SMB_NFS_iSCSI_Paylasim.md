# Modül 07: Dosya Paylaşım Protokolleri: SMB, NFS ve iSCSI Yönetimi

Havuzu inşa ettiniz ve Datasetleri açtınız. Sırada bu verileri diğer sunucuların veya kurum çalışanlarının bilgisayarlarının (Client) erişimine açmak (Sharing) var. ZFS bir dosya sistemidir, o dosyayı ağdan kimin/nasıl okuyacağını ise Paylaşım (Sharing) protokolleri belirler.

## 1. SMB (Samba/CIFS) - Windows, Mac Dosya Sunucusu

SMB, dünya üzerindeki kurumsal paylaşımların (Klasik Muhasebe Dosyası, Y ve Z sürücüleri Ortak Dosya Sunucuları vs.) kralıdır.

- TrueNAS `Sharing` -> `Windows (SMB) Shares` menüsünden yeni bir paylaşım açılır.
- **Path:** Önceden hazırladığımız bir Dataset'i `/mnt/Tank/Shared_Docs` seçeriz.
- **Shadow Copies Mimarisi:** ZFS Snapshotları (Modül 09), Windows'un "Previous Versions (Önceki Sürümleri Geri Yükle)" menüsü ile entegre bir şekilde çalışabilmesi için `Enable Shadow Copies` ibaresi tıklatılır. Personel sildiği Excel dosyasını size Bilet (Ticket) bile açmadan kendi bilgisayarındaki klasöre sağ tıklayıp "Geri Getir" diyerek, Snapshot anındaki haline döner!

*Master Performans İpucu:* Kurumda MacOS kullanan bir dizayner veya yazılımcı ordusu varsa, MacOS (Apple) SMB versiyonundan nefret eder (Yavaştır). `Afp` artık kullanımdan kalktı ancak SMB ayarlarında "Apple SMB2/3 extensions" tikini açarsanız klasöre tıklar tıklamaz devasa hız farkı hissedeceklerdir. Ayrıca Spotlight (Search) performansı için Elasticsearch (TrueNAS Elastic arayüz) entegrasyonuna bakılabilir.

## 2. NFS (Network File System) - Linux ve VMware Ortamı

Windows'tan uzağız, ortamdaki 30 adet Ubuntu Sanal Makinesi veya AWS EKS (Kubernetes) Node'larından, TrueNAS'teki büyük veri havuzuna dev dizinler bağlayacağız (Persistent Volumes). SMB hantal kalır. NFS devreye girer.

- Sadece Unix platformlarında çalışır, Linux Kernel'e entegredir, hız kayıpsız ve ultra performanslıdır (Layer 3-4 de native akar).
- `Sharing` -> `UNIX (NFS) Shares` kullanırız.
- Ayarlarda, güvenlik duvarını yırtmamak adına kural koyarız. Paylaşımın **Network (Allowed Networks)** menüsüne: Sadece ESXi Sunucularının VLAN Subnet'ini (`10.10.10.0/24`) girer, yetki veririz. Aksi taktirde kurum ağındaki rastgele bir Linux makinesi `/etc/fstab` içerisinden o havuzu kolaylıkla hortumlayabilir (Güvenlik zafiyeti). Kurumsal ortamda *NFSv4* zorunlu tutulmalı (Kimlik doğrulamalı ve güvenliğedir, ancak konfigurasyonu uğraştırır). Modern ITIL standartlarında *NFSv3* (Basit IP kimlik kısıtlaması) hızdan dolayı çoğunlukla gözardı edebilmektedir.

## 3. iSCSI (Internet Small Computer Systems Interface) Block Depolama

SMB ve NFS "Dosya Seviyesinde" paylaşımdır (Ortadoğu tabiriyle, ağdaki bir klasördür, içine dosya .pdf atarsınız).
**iSCSI ise Blok (Formatlanmamış) Veri iletir.** TrueNAS bir disk uzantısını, sanki sunucunun içine fiziksel sata kablosuyla SSD takılmışçasına Windows Server'ınıza tanıttırır. Modül 05'te bahsettiğimiz `ZVol` ler iSCSI üzerinden gönderilir (Target-Initiator Modeli ile).

### iSCSI Kurgusu ve Ayarları

1. **Zvol Oluşturulur:** `/mnt/Tank/iscsi-disk1` (Örn. 500GB)
2. **Sharing -> Block (iSCSI):** 
3. **Portal:** SAN Ağında dinlenecek IP konfigürasyonu (VLAN 20: 192.168.50.10).
4. **Initiator Group:** Windows makinenin iqn.xx kodlu güvenliği girilir veya boş (ALL) bırakılır (Güvensiz!).
5. **Target ve Extent:** Zvol ile Windows Server eşleştirilir.

- *Kullanım Durumu:* SQL Server kendi Log/Data diskini bir Network kablosu üzerinden TrueNAS'dan alırken hızı kayıpsız bulur, çünkü iSCSI, TCP/IP ağını sanki bir SCSI (Disk kontrolcü) kablosuymuş gibi kullanır. Cluster Ortamlarında (Failover Clustering) zorunluluktur.

---
[Önceki Modül: Modül 06](./06_Network_VLAN_LACP.md) | [Sonraki Modül: Modül 08 - ACL ve Kullanıcı Yönetimi](./08_ACL_ActiveDirectory.md)

# Modül 06: TrueNAS SCALE Gelişmiş Ağ (Network) Yapılandırması ve Bridge Yönetimi

Modül 03'te sadece Storage trafiği için kurduğumuz LACP ve VLAN ayarlarına ek olarak, TrueNAS SCALE içindeki Linux altyapısından ötürü (VM'ler ve Uygulamalar) farklı ağ arayüzlerine sahip olması zorunludur.

## 1. Network Bridge (Köprü) Mecburiyeti

Eğer TrueNAS SCALE üzerinde Ubuntu (Sanal Makine/VM) veya Pi-hole/Nextcloud (Gelişmiş Uygulama - Apps) koşturacaksanız, Uygulamanın bağlı olduğu sanal ağ portu ile sizin dışarıya açılan fiziksel (enp1s0) portunuzu birbirine entegre etmeniz gerekir. Aksi halde TrueNAS dışına ağ paketleri gidemez.

Bunun tek çözümü Kurulum biter bitmez bir **Bridge vSwitch** (Sanal Anahtar) yaratmaktır.

### Konfigürasyon:

1. TrueNAS Network -> Interfaces Paneline gidilir.
2. `Add` butonuna basılarak `Bridge` (Örn: `br0`) eklenir.
3. Bridge'in altı "Members (Üyeler)" tabına asıl Fiziksel Portunuz (veya Modül 3'te kurulan Link Aggregation `bond0`) seçilerek eklenir. (Yani trafik br0'a gelecek, oradan fiziksel port bond0'a geçerek dışarı atılacaktır).
4. Eskiden Fiziksel Porta (bond0) atadığınız IP Adresi / DHCP'yi silip, bu adresi **kesinlikle** `br0` üzerine atamalısınız. Artık anahtar deliğiniz Bridge (Köprü)'dür.

## 2. Konteyner Ağ Maskesi (Kubernetes / Docker) Çakışması

TrueNAS SCALE, uygulamaları ve konteyner bloklarını (Eskiden K3s, şu an yerel Native konteyner havuzunu) kendi iç DHCP adreslemesi olan `172.16.x.x` altından bir IP atayarak yalıtır.
Bunu `Apps` -> `Settings` -> `Advanced Settings` -> `Cluster/Container Network` üzerinden görebilirsiniz.

**Mimari Tehlike (Master Failsafe):**

Eğer dışarıdaki ofis IP (LAN) deponuz fiziksel Switch (Cisco/HPE) üzerinde `172.16.0.x` kullanıyorsa, konteyner trafiği ile ofis subnetiniz çakışacak; TrueNAS paketleri dışarı yönlendiremeyecektir (Routing Table döngüsü). Kuruluşunuz 172'li IP subnetini yönetiyorsa, TrueNAS Konteyner Network limitlerini `10.88.0.0/16` vb. bir diğer rezerve edilmiş CIDR'a kaydırmalısınız. (Sıfırdan tasarlanan bir IT ortamında LAN 10.x.x.x olmalıdır).

## 3. Storage Gateway ve Route (Yönlendirme) Çözümleri

Büyük organizasyonlarda 3 farklı VLAN (Farklı Subnet) vardır:

- Subnet 1: Admin, Yönetim Portu (`10.0.1.x`)
- Subnet 2: iSCSI Ağı, Veritabanı Serverları (`192.168.50.x`) - *İnternete açık olmamalı.*

Eğer ZFS Pool'larına bulut yedeklemesi (AWS S3) yapılacaksa, TrueNAS Scale'e giren trafiğin hangi kapıdan geçip dış internete ulaşacağının statik rotası (Static Route) belirlenmelidir.

Network -> Global Configuration -> **Default Gateway (IPv4)** olarak `10.0.1.1` (Admin portunun güvenlik duvarı yönlendiricisi) atanır. Bu sayede Update, Bulut senkronizasyon ve benzeri trafikler güvenli Admin VLAN üzerinden dışarı sızdırılır (Egress Traffic Tuning).

---
[Önceki Modül: Modül 05](./05_Dataset_Zvol_Islemleri.md) | [Sonraki Modül: Modül 07 - Paylaşım (SMB/NFS) Yapılandırması](./07_SMB_NFS_iSCSI_Paylasim.md)

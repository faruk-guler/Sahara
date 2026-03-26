# Modül 08: Yetkilendirme Standartları, ACL ve Active Directory

Güvenli bir depolamanın iki bileşeni veri yedekliliği (Modül 04 ZFS) ve Erişilebilirliktir (Availability). Veriye Kim Ulaşacak? "İK Departmanı kendi dosyasını görsün, Muhasebe Klasörüne hiç giremesin. Muhasebeci de Muhasebe klasörünü okusun ama silemesin (Read-Only)." Bütün bunlar, IT yöneticilerini ZFS'den daha fazla uğraştıran **Dosya Yetkilendirme (Permissions / ACL)** bölümüdür.

## 1. ZFS Üzerindeki Dosya Sistemi İzinleri Mimarisi

Geleneksel Linux sistemleri dosyaları (CHMOD sistemi; 777 veya 755 - Kullanıcı/Grup/Diğerleri) bazlı kilitler. TrueNAS üzerinde SMB (Windows) çalışacaksa Unix kodlaması yetersiz kalır (Çünkü Windows klasör yetki matrisinde devasa komplekslikler olan Full Control, Travers, Change Attributes, vb. kavramlar yer alır).

Bu çelişkiyi TrueNAS SCALE **POSIX ve NFSv4 ACL (Access Control Lists)** mekanizmaları ile çözer.

- Yeni Dataset kurulumunda `Share Type` kısmı `SMB` yapıldığında... TrueNAS alt yapıya ZFS'e şunu bildirir: *"Bundan sonra chmod kullanmayı bırak, ACL dosya yönetim formatına geç."*
- `Storage` menüsünden klasörün hemen yanındaki "Edit Permissions (ACL)" ekranı açıldığında tüm gruplar için erişim düzeyi Windows Server arayüzü kadar karmaşık listelenir. Bu listeden, bir "Muhasebe Grubu" eklersiniz ve yetkilerini kısarsınız.

## 2. Active Directory (LDAP) Domain'ine Katılmak

Binlerce çalışanın (Örn: Huseyin_K, Ali_Y, Zeynep_C) tek tek TrueNAS Local User olarak kaydını açamazsınız. Sistem yöneticisi, bu klasör yetkilerini Active Directory üzerinden (Windows Security Tabı ile) yönetmelidir.

### Active Directory'ye Katılım (Join) Süreci

1. `Credentials` -> `Directory Services` -> `Active Directory` menüsüne gidilir.
2. **Domain Name:** `corp.local` (Firmanın AD etki alanı)
3. **Domain Account Name / Password:** Administrator (Veya AD Join yetkisi olan başka servis hesabı (Service Account)).
4. **NetBIOS name:** `TRUENAS`
5. Kaydedilir ve servis çalıştırılır (Join).

Artık TrueNAS klasör izinlerine (ACL Editöre) girdiğinizde, Active Directory'niz devrede olduğu için karşınıza doğrudan Windows Server üzerindeki "HR_Grup", "IT_Admins", "Finance_Writers" gibi Domain bazlı gruplar düşer. Hatta TrueNAS'in lokal ACL ekranına hiç dokunmadan, Windows üzerindeki herhangi bir Bilgisayardan `\\192.168.1.10\Muhasebe` yazıp özellikleri (Properties -> Security) girdiğinizde, NTFS izinlerinden eklediğiniz "Domain User" profilli kişiler arka planda (Samba aracılığıyla) anında TrueNAS ZFS ACL üzerine "Mapping" edilir. Bu Active Directory gücüdür.

## 3. Local Authorization ve İzolasyon Sorunları (Kalıtsal Yetkiler)

Active Directory yapısında klasör (Dataset) `Inheritance (Kalıtsallık / Miras)` yöntemi ile çalışır.
Eğer Root klasöre (Müdüriyet) "Ali okunur/yazılır" yetkisi (Allow) verilmişse; altındaki Müdür_Belgeleri dosyasına Ali girmeye çalıştığında otomatik izin verilir.
Bazen Mimarlar "Drop/Deny" (Yasakla) kuralları eklerler.

*Master Kuralı:* Eğer bir gruba **Deny** eklerseniz, ZFS bu Deny (Engel) yetkisini Allow'un her zaman üstüne koyar. Bir insan 20 gruba sahip olsa ve bir grubu (Kısmi-Yetkili) "Deny" olsa bile kapalı kalır. Bu yüzden kurum yetki mimarilerinde her zaman yasaklayıcı (Deny) felsefe yerine (Kimlik belirtilmeyenleri otomatik Default Deny) yetkilendirici (Grant/Allow) felsefesi üzerinden ACL'leri işlemeye dikkat edin. Aksi halde klasör içindeki kördüğüm izin yapısıyla (File Locklar veya Access Denied'larla) sistem yönetilemez.

---
[Önceki Modül: Modül 07](./07_SMB_NFS_iSCSI_Paylasim.md) | [Sonraki Modül: Modül 09 - Veri Koruması (Snapshot/Replication)](./09_Snapshot_Replikasyon.md)

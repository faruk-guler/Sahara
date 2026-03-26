# Modül 09: Veri Koruması: ZFS Snapshots ve Replikasyon

Geleneksel sistemlerde "Yedekleme (Backup)", verinin X klasöründen Y cihazına saatler süren kopyalanması işlemidir. Bu esnada sunucu yavaşlar, CPU yorulur. ZFS mimarisinde işler tamamen farklıdır.

## 1. ZFS Snapshot Mimarisi (Zaman Makinesi)

ZFS Snapshot'ları dosya kopyalamaz! ZFS'teki Copy-on-Write (CoW) mantığı sayesinde, bir anlık görüntü (Snapshot) aldığınızda boyut tam olarak **0 KB'dır.**
ZFS sadece blokların zaman içindeki adreslerini dondurur (Freeze).

**Nasıl Çalışır?**

- `Dataset-A` içinde 100 GB veriniz var. Snapshot aldınız (Süre: 1 milisaniye).
- Kullanıcı 1 GB boyutunda bir dosyayı silip yenisini (Farklı Veri) yazdı.
- Snapshot dosyasının boyutu şimdi 1 GB'a çıkar. Çünkü ZFS yeni veriyi yazdı, ancak Snapshot kuralları gereği eski 1GB "üzerine yazılmak veya silinmek" yerine diskte donduruldu.
- Sisteme Ransomware (Fidye Yazılımı) bulaşır ve 100GB'ın tamamı şifrelenirse... Snapshot'a "Rollback (Geri Dön)" dediğiniz an, sistem dosyaları çözmez; doğrudan eski temiz blokların adres tablosunu aktif eder. 100GB'lık Cryptolocker felaketi kelimenin tam anlamıyla **1 Saniyede** yok edilir.

**En İyi Kurgu (Periodic Snapshot Task):**

- Data Protection -> Periodic Snapshot Tasks üzerinden kural yazılır.
- **Her 15 Dakikada Bir:** Geçici Snapshot al, 2 gün tut ve sil. (Personel kurtarmaları için).
- **Her Gece 00:00:** Günlük Snapshot al, 30 Gün tut.
- **Her Ayın 1'i:** Aylık Snapshot al, 1 Yıl tut.

## 2. Replication (Replikasyon) Görevleri

Bir disk veya sunucu anakartı (Fiziksel Felaket - Yangın/Su Baskını) tamamen yanarsa Snapshotlar da yanar (Çünkü verinin kendisiyle aynı diskte dururlar).
İşte bu yüzden TrueNAS'ın ikinci bir TrueNAS sunucusuna (Veya Off-site bir lokasyona) veriyi replike etmesi zorunludur.

**ZFS Send/Receive Devrimi:**

- RSync veya Xcopy gibi yazılımlar 200 Milyon dosyayı tek tek tarayıp (Göz atıp) hangisinin değiştiğini bulmaya çalışır. İşlem günler sürebilir.
- ZFS Replikasyonu alt seviyededir. ZFS "15:00 Snapshot'ı ile 16:00 Snapshot'ı arasında SADECE 522 numaralı blok değişti" der ve karşı TrueNAS sunucusuna sadece o küçücük Byte bloğunu (TCP SSH üzerinden şifreli) gönderir. 500TB verinin güncel replikasyonu bile 3-4 saniye sürer.

**Replication Task Ayarı:**

1. İki TrueNAS sunucusu arasında SSH Key değiş tokuşu (SSH Connection) yapılır.
2. `Data Protection` -> `Replication Tasks` üzerinden Kaynak (Source Dataset) ve Hedef (Destination) belirtilir.
3. Kural, yerel Snapshot taskı biter bitmez (Run automatically after Snapshot) çalışacak şekilde ayarlanır. (Asenkron Disaster Recovery).

## 3. Cloud Sync (Bulut Eşitleme)

İkinci bir TrueNAS kuracak bütçeniz yoksa, üçüncü felaket senaryosu (3-2-1 Kuralı) veriyi Bulut'a (AWS S3, Backblaze B2, Google Cloud) atmaktır.

- ZFS Send protokolü bulutta (Object Storage) çalışmaz. Burada `Cloud Sync Tasks` devreye girer.
- Amazon S3 veya Minio bucket adresiniz (Credentials menüsünden) eklenir.
- Görev "PUSH" yönünde (İterek) ayarlanır. Verileriniz gece 03:00'te ucuz soğuk depolamaya (AWS Glacier vb.) aktarılır. Bu senaryoda dosyalar rsync mantığıyla klasör klasör iletilir. Transfer öncesi "Client-side Encryption" seçeneği ile veriyi şifreleyerek göndermek (AWS Hacklense bile verinizin okunmaması için) 2026 kuralıdır.

---
[Önceki Modül: Modül 08](./08_ACL_ActiveDirectory.md) | [Sonraki Modül: Modül 10 - Sanallaştırma ve Konteynerler](./10_SCALE_Apps_VMs.md)

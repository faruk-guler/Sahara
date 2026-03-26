# Modül 11: Performans Tuning (Optimizasyon) ve Darboğaz Analizi

Ev NAS'ı yavaştır çünkü ucuzdur. Ancak Veri Merkezindeki All-Flash (Tamamı NVMe SSD) bir TrueNAS yavaş çalışıyorsa, Mimaride korkunç bir yazılım/ayar darboğazı (Bottleneck) vardır. Master yeteneği bu tıkanıklıkları (I/O, RAM, Network) teşhis edebilmek ve gidermektir.

## 1. ZFS `sync=always` Sendromu (Yazma Hızı Çöküşü)

Bir Windows kopyalama (SMB) hızınız 10 Gbps (Saniyede 1 GB) iken, bir Sanal Makine (ESXi - NFS veya iSCSI Zvol) bağladığınızda yazma hızınız aniden 30 MB/s'ye çakılıyorsa sebebi ağınız değil Protokol Kurallarıdır (Synchronous Writes).

- SMB varsayılan olarak Asenkron (Asynchronous) yazar. Veri önce RAM'e (ARC) girer "Yazdım tamam" deyip işlemi bitirir, ZFS rahatça arka planda bunu yavaş yavaş disklere döker (Transaction Group).
- NFS (ESXi vmware) veya Veritabanları ise Senkron (Synchronous) emir verir. "Veriyi önce doğrudan fiziksel Diske yazacaksın, disk tırnağıyla kazıyacak, bana 'Fiziksel HDD'ye kalıcı kazıdım' onayı döneceksin, sonraki işleme geçeceğim" der.

**Çözümler:**

1. **Risksiz Çözüm (SLOG Eklemek):** Modül 02'de anlatılan Optane/NVMe SLOG disk (ZIL) eklenir. NFS'in "Bana kalıcı kaydettim de!" çağrısına ZFS; veriyi anında hızlıca NVMe'ye yazarak cevap verir (Saniye hızı 1 GB/s'lere uçar) ve daha sonra mekanik disklere basar.
2. **Yasaklı (Dirty) Çözüm:** Dataset veya Zvol ayarlarına gidilip `Sync=Disabled` yapılır. Performans Mükemmelleşir ama eğer o saniye elektrik giderse Veritabanınızın ve Sanal Makinenizin VHD dosyası %100 bozulur (Corrupted). Şirketlerde kesinlikle yasaktır.

## 2. ARC ve L2ARC Darboğazları

Bellek (RAM) ZFS'in motorudur. CPU düşük, I/O Wait (Zamanlayıcı) tavan yapmışsa diskler veriyi yetiştiremiyordur ve Hit Ratio (RAM'den Cevap Verilme Oranı) düşüktür.

- TrueNAS SCALE Linux temelli olduğu için bellek sınırını Linux Kernel belirler (Bazen ZFS'e RAM'in maksimum %50'sini verir, KVM sanallaştırma payı kalsın diye).
- Konsola düşüp komut satırından `arcstat` komutunu yazın.
- Çıkan tabloda `hit%` (Okuma Isteklerinin RAM'den karşılanma yüzdesi) **%80'in altındaysa**, diskler okumaktan eziyet çekiyordur. Sisteme o an RAM (Örn: 128GB -> 256GB) basılmalıdır.
- ARC kapasitesi fiziksel Max seviyeye dayandığında L2ARC NVMe eklenmesi o an (ve yalnızca o an) anlamlı olur.

## 3. Komut Satırı Hekimliği (`zpool iostat`)

GUI grafikleri (Dashboard) 10 saniyede bir veri yansıtır, saliselik I/O darboğazını oradan göremezsiniz. SSH üzerinden bağlanıp gerçek zamanlı röntgen çekmelisiniz.

```bash
zpool iostat -v 1
```

Bu efsane komut, Zpool'un içindeki VDEV'lerin herbirine ayrı ayrı gider, "Saniyede her disk kaç işlem (IOPS) ve kaç bandwidth yapıyor?" bilgisini ekrana (Saniyede 1 kere yenileyerek) basar.

- Çıktıda `c2t1d0` isimli bir diskin okuma işlemini 1 saattir `0` olarak yapıyor ama diğerlerinin `500` olduğunu görüyorsanız... O disk (Tüm poolun boğazını elinde tutan kişi) sessiz sedasız ölmek üzeredir (Pending Sector arızası). ZFS ve GUI henüz diski Fault duruma çekmemiştir ama komut satırı bunu ispiyonlamıştır. O disk fiziksel olarak sökülüp değiştirilmek zorundadır.

## 4. Fragmentasyon (Dağılma) Tehlikesi

- `zpool list` komutunu vurduğunuzda tabloda `FRAG` sütununu göreceksiniz.
- ZFS Copy on Write nedeniyle çok fazla boşluk atlar ve diski bir İsviçre peynirine çevirir.
- Havuz doluluğu (Capacity) **%80'i aştıktan sonra**, ZFS bir veriyi (Örn 1GB dosyayı) diske sığdırmak için yüzlerce parça deliği teker teker bulmaya (Free Space Map) çalışırken devasa CPU tüketir ve pool komple durma noktasına (Stall) gelir. ZFS mimarisinde Altın Kural, Havuzu asla %80'in üzerinde dolu kullanmamaktır! Darboğaz (Frag>50%) oluşmuşsa acilen havuza yeni bir VDEV atılıp Striping ile havuz geneline boş hava (Space) solutulmalıdır.

---
[Önceki Modül: Modül 10](./10_SCALE_Apps_VMs.md) | [Sonraki Modül: Modül 12 - Güvenlik ve Hardening (Final)](./12_Guvenlik_ve_Hardening.md)

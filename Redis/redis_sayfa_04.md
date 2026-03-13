# 📄 Sayfa 4: Client-Side Caching ve Bellek Derinliği (Ultra-Detay)

Redis 6+ ile gelen devrimsel özellikler ve RAM yönetiminin matematiksel detayları.

## 1. Client-Side Caching (Tracking)
Normalde uygulama veriyi Redis'ten ister. Veri değiştiğinde uygulamanın haberi olmaz.
- **`CLIENT TRACKING ON`**: Redis, istemcinin hangi anahtarları okuduğunu takip eder.
- **Invalidation Message:** Bir anahtar değiştiğinde, Redis o anahtarı okumuş olan tüm istemcilere "Bu veri eskiydi, sil!" mesajı gönderir.
- **Sonuç:** Uygulama veriyi kendi RAM'inde tutabilir ama veri tutarlılığını Redis sağlar. Ağ trafiği %90 azalır.

## 2. Bellek Parçalanması (Fragmentation Ratio)
`INFO memory` komutundaki `mem_fragmentation_ratio` en kritik değerdir.
- **Ratio > 1.5:** İşletim sisteminin ayırdığı bellek ile Redis'in kullandığı veri arasında uçurum var demektir (Bellek israfı).
- **Çözüm:** `active-defrag yes`. Redis, CPU'nun boş olduğu anlarda belleği arka planda "defrag" ederek (verileri yan yana dizerek) işletim sistemine RAM iade eder.

## 3. İleri Seviye Tahliye (Eviction) Ayarları
- **`maxmemory-policy volatile-lfu`**: Sadece süresi dolacak (TTL set) olanlar arasında en az sıklıkta kullanılanları (LFU) sil. 
- **`maxmemory-eviction-tenacity`**: Redis'in veri silmek için ne kadar CPU harcayacağını belirler. Yazma yükü çok ağırsa bu değeri artırmanız gerekebilir.

## 4. I/O Threads Konfigürasyonu
Eğer 10Gbps+ bir ağ kartınız varsa ve CPU yetmiyorsa:
```text
io-threads 4
io-threads-do-reads yes
```
Bu ayar, ağ paketlerinin işlenmesini ana thread'den alıp yardımcı thread'lere dağıtır.


# Modül 05: Dataset ve Zvol Yönetimi (Sıkıştırma & Kayıt Boyutları)

Zpool'u tek bir büyük "Depo" olarak kullanmak Windows'un C: sürüsünü içine her şeyi döküp karmakarışık bırakmaktan farksızdır. ZFS Mimarisi veriyi Klasörler değil, *Dataset* ve *Zvol* adı verilen mantıksal ayrımlarla böler.

## 1. Dataset Nedir? Neden Sadece Klasör Değildir?

**Dataset:** Kendine has özellikleri (Sıkıştırma, Quota, Recordsize, Snapshots) taşıyan alt dosya sistemleridir.

- Örneğin `Tank` isimli Zpool'un içerisinde `Finance_Docs`, `VM_Drives`, `Media_Server` adında 3 farklı Dataset açtınız.
- İsterseniz "Finance_Docs" datasetine 1 TB Kotayı koyarsanız, o dataset daha dolmasa bile diğerlerine müdahale edemez.
- İsterseniz "Media_Server"ın Backup/Snapshot kuralını haftalık (Çünkü fazla değişmez) yapıp, "Sertifikalar" datasetini 5 dakikada bire alabilirsiniz. Sadece ilgili Datasetlerin kopyası (Delta) diskte tutulur.

## 2. Zvol (ZFS Volume) Nedir?

Datasetler dosya (File-level) veriler barındırır.
Ancak **iSCSI** protokolü ile bir Windows veya ESXi sunucuya "Sen bunu doğrudan kendi C: diskin gibi, Saf Block cihazı gibi biçimlendir" diyerek bir pay atamak isterseniz, Dataset kullanamazsınız.
Gidip `Tank` havuzunda 200 GB boyutunda bir **ZVol (Block Storage)** oluşturursunuz. (Neredeyse sanal bir Harddisk bölüntüsüdür).

- Sanal Makinelerin (VM) Harddiskleri genelde raw, qcow2 veya Zvol formatında tutulur.
- TrueNAS SCALE'de Sanal Makineler için en yüksek I/O performansını ZVol üretir.

## 3. Performans ve Tuning Optimizasyonları (Master Yönergeleri)

Bir Dataset oluştururken yapılan 3 tercih, tüm Storage'in hayatına yön verir.

### 3.1 Compression (Sıkıştırma)

Varsayılan algoritma ZFS'de **LZ4**'tür.

- *Korku:* "Sıkıştırma açarsam CPU çok yorulur mu?"
- *Gerçek:* LZ4 o kadar inanılmaz hızlı ve hafiftir ki, Sıkıştırılmamış dosyayı diskten (Eski yavaş HDD'den) okumaya çalışmak CPU'yu daha çok bekletir. Sıkıştırma (LZ4) mutlaka açık bırakılmalıdır.
- **2026 Yeniliği:** Yedekleme depolamalarında LZ4 yerine "ZSTD" (Zstandard) açmak CPU'yu biraz kullansa da veriyi %30 ila %60'a varan oranda daha da küçültür.

### 3.2 Deduplication (Tekilleştirme Tuzağı)

"Ofisteki 50 çalışan, 1GB'lık bir Excel dosyasının kendi ana dizinlerinde saklanıyorsa... Benim diskimde 50 GB yer gider. Dedup açayım sadece 1GB gitsin."
Bu cümle ZFS dünyasında **İntihar Nedenidir.**

- ZFS Deduplication, anlık olarak her bir veri bloğunun (Block) hash'ini (Tablosunu) hesaplayıp RAM'de "Dedup Table" denilen bir indekse gömer. Bu tablo RAM'de olmazsa havuzunuz bir USB bellek hızına kadar çöker.
- *Kural:* Her 1 TB Tekilleştirilmiş veri için 5 GB **Ekstra** RAM gerektirir (Örn: 200TB Depolamaya ~1TB RAM gerektirir). 2026 itibariyle Dedup, sadece dev gibi Fast-Flash (All Flash) ortamlarda RAM bolluğuyla önerilir. Kapalı bırakın! (Bir poola uyguladıktan sonra kapatmak zordur, veriyi taşıyıp pool'u baştan kurmanız gerekir).

### 3.3 Recordsize (Blok Boyutu Optimizasyonu)

Bir dosya yazılırken ZFS onu kaç paralık (Örn: 128 Kilobayt) tır kasalarına ayırarak diskte tutsun?

- **Varsayılan Değer:** ZFS Default `128K`dır.
- **Büyük Dosya / Torrent Tipi Veriler / 4K Videolar (Media):** Recordsize **1MB** yapılmalıdır. Diskte milyonlarca küçük parçacık yaratıp Fragmantasyon (Dağılma) hızını frenler. Disk kafası gidip 1MB okur geçer.
- **Veritabanları (SQL/PostgreSQL/Oracle):** Recordsize Veritabanının "Page Size"ına eşitlenmelidir (**16K** veya **8K**). Eğer MySQL, verisinde 8K'lık küçük bir yazma yaptığında, ZFS alt katmanda 128K'lık koca bir bloğu diskten okuyup, içini 8K değiştirip, koca bloğu geri kaydetmeye zorlarsa, saniyedeki veritabanı okuyup okuması (Write Amplification) felakete döner! Veritabanı datasetleri küçük (16k/64k) tutulmalıdır. Veritabanı ile medya dosyaları aynı Dataset'te asla durmaz (Master Rule).

---
[Önceki Modül: Modül 04](./04_ZFS_Mastery.md) | [Sonraki Modül: Modül 06 - İleri Ağ Ayarları](./06_Network_VLAN_LACP.md)

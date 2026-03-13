# 📄 Sayfa 6: İleri Kalıcılık ve Replication Internals (Ultra-Detay)

Redis'in diske yazma ve replikasyon süreçlerinin arka planındaki işletim sistemi etkileşimleri.

## 1. Copy-on-Write (COW) ve RAM Yan Etkisi
Redis `BGSAVE` veya `BGREWRITEAOF` için `fork()` yaptığında, işletim sistemi belleği kopyalamaz, sadece işaret eder.
- **Sorun:** Eğer yazma işlemi sırasında uygulama veri eklemeye devam ederse, işletim sistemi o bellek sayfalarının yeni kopyasını oluşturur (**Copy-on-Write**).
- **Sonuç:** Çok yoğun yazma varken Persistence işlemi başlarsa, Redis kullandığı RAM'in 2 katına kadar RAM tüketebilir. Sunucunuzda her zaman veri miktarının 2 katı kadar RAM boşluğu bırakmak bu yüzden kritiktir.

## 2. Diskless Replication (Disk Olmadan Replikasyon)
Normalde bir Replica bağlandığında Master önce diske RDB yazar, sonra onu gönderir. Disk yavaşsa Master kilitlenebilir.
- **`repl-diskless-sync yes`**: Master, RDB'yi diske hiç dokunmadan doğrudan ağ soketi üzerinden Replica'ya aktarır. 
- **Avantaj:** Disk I/O darboğazı yaşanmaz, replikasyon çok daha hızlı başlar.

## 3. RDB Veri Doğrulama (Checksums)
RDB dosyalarının sonunda 64-bitlik bir CRC64 checksum bulunur. 
- **`rdbchecksum yes`**: Dosya yazılırken ve okunurken veri bozulması olup olmadığını kontrol eder. Performansı %10 düşürebilir ama veri güvenliği için üretim ortamında vazgeçilmezdir.

## 4. AOF Multi-Part (Redis 7+)
Geleneksel AOF'de, rewrite sırasında Master hem eski dosyaya hem yeniye yazmak zorundaydı (AOF buffer). Redis 7 ile gelen **Multi-Part AOF**:
- Veriyi bir "base" dosya ve bir "increment" dosyası olarak ayırır. Rewrite işlemi bittiğinde sadece dosyaların isimleri değiştirilir. Bu, disk yazma yükünü (Write Amplification) ciddi oranda azaltır.


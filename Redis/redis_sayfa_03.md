# 📄 Sayfa 3: İleri Veri Yapıları ve SCAN Mekanizması (Ultra-Detay)

Redis'te veri sorgularken sistemi kilitlememek ve en verimli yolu seçmek için bu teknikleri öğrenmelisiniz.

## 1. Neden `KEYS *` Kullanmamalısınız? (`SCAN` Cursor)
`KEYS *` komutu tüm anahtarları bir kerede döner ve milyarlarca anahtarda Redis'i kilitler.
- **Çözüm:** `SCAN` komutu bir "Cursor" (İmleç) kullanır. Redis'ten her seferinde küçük bir parça anahtar çeker ve sistemi asla kilitlemez.
    ```bash
    SCAN 0 MATCH user:* COUNT 100
    ```
    (0: Başlangıç imleci. Size bir sonraki imleci döner, o değeri kullanarak devam edersiniz).

## 2. SORT Komutu: Gizli Güç
`SORT` komutu sadece listeleri değil, Hash'lerdeki verilere göre listeleri sıralamanızı sağlar.
- **`SORT tasks BY task_score:* GET task_name:*`**: Görevleri puanlarına göre sıralayıp isimlerini getirir.
- **Dikkat:** `SORT` işlemi CPU yoğun bir iştir. Sık kullanılıyorsa sonucu başka bir anahtara `STORE` ile kaydetmek (Caching) daha iyidir.

## 3. Geospatial (Coğrafi) Indeksler
`GEOADD` ile koordinatları (longitude, latitude) saklayıp `GEODIST` ile mesafe ölçebilir, `GEOSEARCH` ile "yakınımdakiler" özelliğini 1 mili saniyede yapabilirsiniz.
- **Arka Plan:** Koordinatlar bir `Sorted Set` içine 52-bitlik bir sayıya dönüştürülerek (`Geohash`) saklanır.

## 4. RedisBloom: Bloom ve Cuckoo Filters
Eğer veritabanımızda "Bu kullanıcı adı daha önce alındı mı?" gibi bir soru varsa, veritabanına bakmadan önce **Bloom Filter** kullanırız.
- **Bloom Filter:** "Kesinlikle yok" veya "Olabilir" cevabı verir. %0 hata payı ile "yok" diyebilir, böylece gereksiz disk/veritabanı sorgularını önler.
- **Cuckoo Filter:** Bloom filter'dan farkı, eklenen öğelerin silinebilmesine izin vermesidir.

> [!IMPORTANT]
> Tüm bu yapılar RAM dostudur. Örneğin, 10 milyon öğelik bir Bloom Filter sadece birkaç MB yer kaplar.


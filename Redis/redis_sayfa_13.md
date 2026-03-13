# 📄 Sayfa 13: RESP3 Protokolü ve Internals II (Expert Edition)

Redis'in iletişim kurma biçimi ve veriyi bellekte saklama teknolojileri her versiyonda daha da evrimleşiyor.

## 1. RESP3 Protokolü: Modern İletişim
Redis 6 ile gelen **RESP3**, eski RESP2'nin yerini alan daha zengin bir iletişim protokolüdür.
- **Push Messages:** İstemci bir istek yapmadan sunucunun veri göndermesini sağlar. (Örn: Client-side tracking geçersiz kılma mesajları).
- **Attributes:** Bir komutun cevabına ek meta veriler (gecikme, cache durumu vb.) eklenebilir.
- **Maps ve Sets:** Protokol artık sadece toplu dizileri (Arrays) değil, yerel olarak Hash Map ve Set tiplerini tanır. Bu sayede istemci tarafında veri işleme kolaylaşır.

## 2. Listpacks: Ziplist'in Yerine Geçen Yapı
Redis 7 ile birlikte, yıllardır kullanılan `Ziplist` yapısı emekli edilip yerine **Listpack** getirildi.
- **Neden?** Ziplist'lerde bir eleman değiştiğinde tüm listenin yeniden hesaplanması (Cascading Update) gerekebiliyordu ve bu performans kaybına yol açıyordu.
- **Listpack:** Her elemanın boyut bilgisini kendi içinde saklayarak daha güvenli ve hızlı bir bellek yönetimi sunar.

## 3. Radix Tree (Rax): Hızlı Anahtar Arama
Redis'in anahtar deposu (Keyspace) aslında bir Hash Table'dır. Ancak Streams ve Keyspace Notifications gibi özelliklerde **Radix Tree** (Rax) adı verilen özel bir ağaç yapısı kullanılır.
- **Prefix Matching:** "user:" ile başlayan tüm anahtarları hızlıca bulmak için ağaç yapısı, string'in ortak kısımlarını (prefix) birleştirerek bellek tasarrufu ve hızı birleştirir.
- **Streams:** Mesaj ID'leri (Örn: `16213423-0`) Radix Tree içinde saklandığı için belirli bir zaman aralığındaki mesajı bulmak çok hızlıdır.

## 4. Jemalloc ve Bellek Parçalanması 2.0
Redis, C'nin standart `malloc`'u yerine **Jemalloc** kullanır.
- **Arenas:** Belleği küçük parçalara (Arenas) bölerek çoklu thread'lerin birbirini beklemesini engeller.
- **Thread Caching:** Her thread kendi küçük bellek havuzunu tutar, böylece global kilit (Global Lock) maliyeti azalır.

> [!IMPORTANT]
> `OBJECT ENCODING <key>` komutunu kullandığınızda gördüğünüz `listpack` veya `hashtable` ibareleri, Redis'in o veriyi hangi çekirdek yapıyla sakladığını gösterir.


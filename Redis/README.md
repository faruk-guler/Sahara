# 🚀 Redis: Modern Veri Dünyasının Hız Motoru (Giriş & Rehber)
<img src="./img/redis-banner-1544x500.png" alt="Moodle Preview" width="50%">

Redis (Remote Dictionary Server), dünyadaki en popüler açık kaynaklı, bellek içi (in-memory) veri yapısı deposudur. Genellikle bir veritabanı, önbellek (cache), mesaj kuyruğu (message broker) ve akış motoru (streaming engine) olarak kullanılır.

## ❓ Redis Nedir?
Redis, verileri disk yerine doğrudan **RAM** (Rastgele Erişimli Bellek) üzerinde tutar. Bu sayede milisaniyeler (hatta mikrosaniyeler) düzeyinde erişim hızı sağlar. Sadece "anahtar-değer" (key-value) değil; listeler, kümeler, sıralı kümeler, hash tabloları, bitmapler ve coğrafi (geospatial) indeksler gibi karmaşık veri yapılarını destekler.

## 🌟 Neden Redis Kullanmalıyız?
1. **Ultra Yüksek Performans:** RAM odaklı yapısı ve optimize edilmiş C kodu sayesinde saniyede milyonlarca işlemi işleyebilir.
2. **Esnek Veri Yapıları:** Sadece dizelerle sınırlı kalmaz, uygulamanızın ihtiyacına göre en doğru veri tipini seçmenize izin verir.
3. **Kalıcılık:** Bellek içi olmasına rağmen verileri periyodik olarak diske yazar (RDB/AOF), böylece elektrik kesintisinde veri kaybolmaz.
4. **Zengin Ekosistem:** Pub/Sub, İşlemler (Transactions), Lua Scripting ve Streams gibi gelişmiş özelliklerle tam teşekküllü bir platformdur.

## 🎯 Temel Kullanım Senaryoları
- **Önbellek (Caching):** Veritabanı yükünü azaltmak ve yanıt süresini düşürmek.
- **Oturum Yönetimi:** Milyonlarca kullanıcı oturumunu hızlıca yönetmek.
- **Liderlik Tabloları:** Oyunlarda veya yarışmalarda gerçek zamanlı sıralama.
- **Hız Sınırlama (Rate Limiting):** API'lere gelen aşırı isteği kontrol altına almak.
- **Gerçek Zamanlı Analiz:** Canlı trafik takibi ve log işleme.

---
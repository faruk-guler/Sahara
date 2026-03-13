# 📄 Sayfa 10: İleri Tasarım Desenleri ve Redlock (Ultra-Detay)

Redis'in gücünü en zorlu senaryolarda kullanmak için son dokunuşlar.

## 1. Sliding Window Rate Limiting (Kayan Pencere)
Sabit pencereli (fixed window) hız sınırlama, pencerelerin kesiştiği anlarda (Örn: 59. saniye ve 01. saniye) limiti aşmanıza izin verebilir.
- **Çözüm (ZSet):** Her isteği bir `Sorted Set` içine `timestamp` skoruyla atın. 
- **Mantık:**
    1. `ZREMRANGEBYSCORE limit:key 0 (now-window)` (Eski istekleri sil).
    2. `ZADD limit:key now timestamp`.
    3. `ZCARD limit:key` (Kalan istek sayısı limitinizi aşıyor mu?).
    4. Bu işlemi bir Lua betiği içinde atomik yapın.

## 2. Redlock Algoritması ve Pitfalls (Zayıf Yönler)
Redlock, dağıtık kilitler için bir standart kabul edilir ancak bazı riskleri vardır.
- **Clock Drift (Saat Kayması):** Sunucuların saatleri birbirlerinden milisaniye düzeyinde farklıysa kilidin süresi bittiği sanılabilir.
- **GC Pauses (Çöp Toplama Duraklamaları):** Uygulama tarafında dilin (Örn: Java) GC duraklaması, kilidin süresinin dolmasına ve verinin bozulmasına neden olabilir.
- **Tavsiye:** Fencing Token kullanarak veritabanı tarafında da çek (check) mekanizması kurun.

## 3. Akıllı Önbellek: "Bypassing" ve "Pre-warming"
- **Cold Start:** Yeni açılan bir sunucunun önbelleği boştur. Trafiği bir kerede verirseniz veritabanı çöker.
- **Pre-warming:** Sunucu trafiğe açılmadan önce popüler verilerin bir script ile Redis'e doldurulmasıdır.

## 4. Architectural Gold Rules (Altın Kurallar)
1. **İsimlendirme:** `departman:nesne:id:alan` şeklinde hiyerarşik anahtar isimleri kullanın (Örn: `marketing:user:1001:email`).
2. **Büyük Veriden Kaçının:** Tek bir anahtarda 100MB veri tutmayın. Mümkünse bölün.
3. **Pipelining:** 100 komutu tek tek göndermek yerine `Pipelining` ile toplu gönderin. RTT (Round Trip Time) maliyetini 100 kat azaltın.


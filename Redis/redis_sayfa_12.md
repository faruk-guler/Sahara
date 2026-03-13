# 📄 Sayfa 12: Çok Bölgeli Mimari ve CRDT (Expert Edition)

Küresel ölçekli uygulamalarda verinin sadece bir merkezde olması, dünyanın diğer ucundaki kullanıcılar için yüksek gecikme (latency) demektir. Çözüm: **Active-Active** mimari.

## 1. Active-Active vs Active-Passive
- **Active-Passive:** Veri bir ana merkeze yazılır, diğerleri sadece okur. Yazma gecikmesi yüksektir.
- **Active-Active:** Veri en yakın veri merkezine (Örn: İstanbul, New York, Tokyo) yazılır. Tüm merkezler birbirini günceller.

## 2. CRDT (Conflict-free Replicated Data Types)
İki farklı kıtadaki kullanıcı aynı anda aynı veriyi değiştirirse ne olur? Geleneksel veritabanları kilitlenir veya veri kaybeder. Redis, bu sorunu **CRDT** ile çözer.
- **Matematiksel Çözüm:** Veri tipleri (Counter, Set vb.) öyle tasarlanmıştır ki, güncellemeler hangi sırayla gelirse gelsin sonuç tüm sunucularda aynı olur.
- **LWW (Last Write Wins):** Çakışma durumunda "zaman damgası" en yeni olan veri kazanır.
- **Örn:** Bir sepete ürün ekleme işlemi (Set), her iki bölgede de yapılırsa, replikasyon sonrası her iki ürün de sepette görünür (Kaybolmaz).

## 3. Global Replikasyon ve Veri Tutarlılığı
- **Global Database (Redis Enterprise):** Bölgeler arası replikasyonu otomatik yönetir.
- **Düşük Gecikme:** Kullanıcı her zaman kendisine en yakın (Local) Redis'e yazar. Veri arka planda diğer kıtalara senkronize edilir.

## 4. Kullanım Senaryosu: Global Oturum Yönetimi
Bir kullanıcı İstanbul'dan giriş yapıp New York'a uçtuğunda, oturum bilgisi New York veri merkezinde zaten hazır bulunur. Kullanıcı tekrar giriş yapmak zorunda kalmaz.

> [!TIP]
> **Active-Active** sistemlerde "Saat Senkronizasyonu" (NTP) hayati önem taşır. `LWW` mekanizmasının doğru çalışması için sunucu saatlerinin milisaniye düzeyinde tutarlı olması gerekir.


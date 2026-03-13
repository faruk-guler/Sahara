# 📄 Sayfa 8: Cluster Sharding ve HA Internals (Ultra-Detay)

Redis'in yatayda nasıl ölçeklendiğinin ve hata anında nasıl hayatta kaldığının matematiksel arka planı.

## 1. Hash Slot Migration (Çalışırken Veri Taşıma)
Cluster modunda yeni bir node eklendiğinde veya çıkarıldığında veriler nasıl taşınır?
- **ASK ve MOVED:** Veri taşınırken (Migration) istemciye önce `ASK` döner. Bu, "Veri şu an taşınıyor, geçici olarak şu node'a sor" demektir. Taşıma bittiğinde ise kalıcı olarak `MOVED` döner.
- **`CLUSTER SETSLOT`**: Slotların el ile bir node'dan diğerine taşınmasını sağlayan düşük seviyeli komuttur.

## 2. Replica Migration (Otomatik Dengeleme)
Bir Cluster yapısında bazı Master'ların 2 replikası varken bazılarının replikası ölebilir.
- **Özellik:** Redis Cluster, replikası fazla olan Master'dan bir replikayı alıp, replikası olmayan Master'a otomatik olarak bağlar (**Replica Migration**). 
- **Fayda:** Sistem genelindeki hata toleransını (Fault Tolerance) dinamik olarak dengeler.

## 3. Sentinel vs Cluster Konsensüs (Oy Birliği)
- **Sentinel:** Bir "Lider" seçmek için **Raft Benzeri** bir algoritma kullanır. Sentinel'lerin çoğu "Master öldü" derse failover başlar.
- **Cluster:** Dış bir izleyiciye (Sentinel) ihtiyaç duymaz. Node'lar birbirlerine **Gossip Protocol** ile PING/PONG atar. Master node'ların çoğunluğu bir node'un öldüğüne karar verirse o node'un replikası Master ilan edilir.

## 4. Hash Tags: İlgili Verileri Aynı Node'da Tutma
Normalde `user:1:profile` ve `user:1:orders` farklı node'lara düşebilir. Ama bazen bu verileri aynı node'da toplu işlem (MULTI/EXEC) yapmak istersiniz.
- **Çözüm:** `{user:1}:profile` ve `{user:1}:orders`. Süslü parantez içindeki kısım (`user:1`) hashlenir. Parantez içi aynı olduğu için iki anahtar da **garantili olarak aynı node'a** düşer.


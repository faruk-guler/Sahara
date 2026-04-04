# 13 - Uzman Mimar Rehberi (Black Belt Level)

Bu rehber, OpenSearch platformunu sadece işleten değil, onun **iç mekanizmalarını (internals)** bir mimar titizliğiyle tasarlayan uzmanlar için hazırlanmıştır.

## 📦 Shard Allocation Internals (Shard Yerleşimi)

Kümenize bir shard eklendiğinde veya bir düğüm düştüğünde, OpenSearch bu shard'ı nereye koyacağına nasıl karar verir?

### Allocation Deciders (Karar Vericiler)
OpenSearch, her shard hareketi için aşağıdaki "Decider"lardan onay alır:
*   **DiskThresholdDecider:** Eğer bir düğümün disk doluluğu `%85` (Low Watermark) üzerindeyse oraya yeni shard koymaz. `%90` (High Watermark) üzerinde ise mevcut shard'ları oradan taşımaya başlar.
*   **ThrottlingAllocationDecider:** Aynı anda çok fazla shard taşınarak ağın (network) tıkanmasını önler.
*   **MaxRetryAllocationDecider:** Bir shard atanırken 5 kez üst üste hata alırsa, o shard'ı "Unassigned" bırakır (Retry limit).
*   **AwarenessAllocationDecider:** Shard'ları farklı kabinlere (racks) veya bölgelere (zones) dağıtarak fiziksel felaketlere karşı koruma sağlar.

## 💾 Dağıtık Sistem Teorisi ve Quorum

Bir OpenSearch kümesinin "kararlı" (stable) kalması için Master (Cluster Manager) düğümlerinin **Quorum** sayısını koruması gerekir.

*   **Küme Durumu (Cluster State):** Tüm indekslerin, shard'ların ve düğümlerin listesi. Sadece Cluster Manager bu listeyi güncelleyebilir.
*   **Master Election:** Master düştüğünde, kalan düğümler arasından yeni bir master seçilir. Bu seçim için `n/2 + 1` kuralı (Quorum) esastır. 2026 versiyonlarında bu süreç milisaniyeler mertebesindedir.

## 🚀 1M+ Log/Sn Yazma Stratejisi

Büyük ölçekli kurumlarda saniyede milyonlarca kayıt gelirken sistemi ayakta tutmanın sırları:

1.  **Index Buffer Tuning:** `indices.memory.index_buffer_size` parametresiyle, verilerin diske yazılmadan önce RAM'de ne kadar biriktirileceği belirlenir (Öneri: %20-30).
2.  **Refresh Interval:** `refresh_interval: -1` yaparak veri girişi sırasında Lucene segmentlerinin sürekli oluşturulmasını engelleyin. Veri alımı bittiğinde veya her 10-30 saniyede bir manuel tetikleyin.
3.  **Cross-Region Scaling:** Veriyi bölgesel olarak bölüp (Sharding by Region) CCS üzerinden tek bir noktadan sorgulatın.

---

[🏠 Ana Sayfaya Dön](../../README.md)

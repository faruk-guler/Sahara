# 09 - Bellek Yönetimi ve Önbellekleme (Expert Caching)

OpenSearch'te sorgu performansını milisaniyelerden mikro saniyelere düşürmenin yolu, bellek katmanlarını (caching) doğru yönetmekten geçer.

## 💾 Önbellek Katmanları (Cache Layers)

OpenSearch, farklı düzeylerde veriyi RAM'de saklamak için üç ana önbellek mekanizması kullanır:

### 1. Request Cache (İstek Önbelleği)
Bir arama isteği aynı parametrelerle (size=0 ve sadece aggregation içeren sorgular) tekrar gelirse, OpenSearch veriyi tekrar hesaplamak yerine sonucu RAM'den döner.
*   **Varsayılan Boyut:** `%1` (Toplam heap bellek miktarına göre).
*   **Yapılandırma:** `indices.queries.cache.size: 512mb`.

### 2. Query Cache (Sorgu Önbelleği)
Bitset tabanlıdır. `bool` sorgusu içindeki `filter` alanları burada saklanır. Hangi belgelerin belirli bir filtreyle eşleştiğini (0/1 şeklinde) hızlıca bulur.
*   **LRU (Least Recently Used):** En az kullanılan filtreler bellekten atılır.

### 3. Fielddata Cache (Alan Verisi Önbelleği)
`text` alanları üzerinde alfabetik sıralama veya toplulaştırma (aggregation) yapmak istediğinizde, OpenSearch tüm alan değerlerini RAM'e yükler.
*   **Dikkat:** `Fielddata` bellek kullanımı kontrol edilmezse, kümede "OutOfMemory" hatalarına neden olabilir. Çözüm olarak `doc_values` kullanımı her zaman tercih edilmelidir.

## ☕ JVM ZGC (Zero Garbage Collector) Derinlemesine

2026 yılı standartlarında, 32GB+ heap boyutları için **ZGC** mutlak tercihtir.

### ZGC'nin Avantajları:
*   **Pause Times:** Bellek ne kadar büyük olursa olsun (1TB bile), çöp toplama duraklamaları 10 milisaniyenin altındadır.
*   **Colored Pointers:** Nesne referanslarını bit düzeyinde işaretleyerek, bir nesnenin taşınıp taşınmadığını (Relocation) CPU seviyesinde hızlıca anlar.
*   **Load Barriers:** Bellekten bir nesne çekerken "güncel mi?" kontrolünü yapan çok hafif bir kod parçasıdır.

```bash
# Uzman ZGC Ayarları
-XX:+UnlockExperimentalVMOptions
-XX:+UseZGC
-XX:ZCollectionInterval=300
-XX:ZAllocationSpikeTolerance=3.0
```

## 📉 Off-heap Bellek (filesystem cache) Kullanımı

OpenSearch'ün performansı sadece JVM heap'e bağlı değildir. Lucene, dizin dosyalarını okumak için işletim sisteminin **Filesystem Cache (Page Cache)** mekanizmasını kullanır.

> [!IMPORTANT]
> Heap belleği hiçbir zaman toplam RAM'in %50'sini aşmamalıdır. Kalan %50, işletim sisteminin dizinleri cache'lemesi için boş bırakılmalıdır. Aksi halde sürekli disk I/O yaparak sistem darboğaza (thrashing) girer.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

# 09 - Performans Optimizasyonu

OpenSearch kümenizin performansını artırmak için donanım ve yazılım seviyesinde yapabileceğiniz ayarlar.

## ☕ JVM (Java Virtual Machine) Tuning

OpenSearch bir Java uygulamasıdır ve bellek (heap) yönetimi kritiktir.

*   **JVM Heap Boyutu:** Toplam RAM'in %50'sini (maksimum 32GB) atayın.
*   **JDK 21+ & ZGC:** 2026 standartlarında, düşük duraklama süreli **ZGC (Z Garbage Collector)** kullanımı önerilir. Büyük heap boyutlarında (64GB+) bile milisaniye altı duraklamalar sağlar.

```bash
# jvm.options
-XX:+UseZGC
-Xms32g
-Xmx32g
```

## 🚀 Donanım Hızlandırma (Hardware Acceleration)

OpenSearch v3, CPU ve GPU komut setlerinden en iyi şekilde yararlanır.

*   **AVX-512 / AMX:** Intel ve AMD sunucu işlemcilerindeki bu komut setleri, vektör işlemlerini 10 kata kadar hızlandırabilir.
*   **GPU Offloading:** Bazı k-NN motorları, çok büyük boyutlu vektör aramalarını GPU'ya devrederek CPU üzerindeki yükü azaltabilir.

## 💽 Disk ve I/O Optimizasyonu

*   **SSD Kullanımı:** Mekanik disklerden (HDD) kaçının.
*   **RAID 0:** Maksimum performans (ama 0 yedeklilik, replicaya güvenin).
*   **Swapping'i Kapatın:** Linux üzerinde `swapoff -a` komutunu çalıştırın.

## 🔍 Sorgu Optimizasyonu (Query Tuning)

*   **Filter vs Query:** Arama puanı (score) gerekmeyen yerlerde `filter` kullanın.
*   **Size Sınırı:** Bir kerede binlerce belge dönmeye çalışmayın (Pagination).
*   **Mapping:** `keyword` ve `text` alanlarını doğru seçerek gereksiz dizin boyutundan kaçının.

### 💡 İpucu: Shrink API
Çok fazla boş alanı olan veya küçük shard'lara bölünmüş dizinleri `Shrink API` ile birleştirerek okuma performansını artırabilirsiniz.

---

> [!TIP]
> **Uzmanlar İçin:** **OpenSearch Benchmark (OSB)** ile yük testi, **Cluster Insights** tanıları ve Linux çekirdek optimizasyonlarını [09 - Performans Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: İzlenebilirlik ve Analiz](../08-Izlenebilirlik-ve-Analiz/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Yedekleme ve Kurtarma ➡️](../10-Yedekleme-ve-Kurtarma/README.md)

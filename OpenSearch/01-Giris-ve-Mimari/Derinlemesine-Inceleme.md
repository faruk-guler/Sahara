# 01 - Mimari Derinlemesine İnceleme (Lucene 10 & gRPC)

OpenSearch'ün kalbini ve 2026 yılındaki devrimsel mimari değişikliklerini teknik detaylarıyla keşfedin.

## ⚙️ Apache Lucene 10 Kalbi

OpenSearch, veri saklama ve arama için **Apache Lucene 10** motorunu kullanır. Bu sürümle gelen yenilikler:

### 1. Segment Merge Politikaları
*   **TieredMergePolicy:** Lucene, verileri RAM'den diske "Segment"ler halinde yazar. Çok fazla küçük segment performansı düşürür. Lucene 10, segmentleri boyutlarına göre gruplandırarak birleştirir (merge).
*   **Merge Scheduler:** Arka planda çalışan merge işlemleri artık I/O darboğazı yaratmamak için dinamik olarak ayarlanır (Dynamic Merge Scheduling).

### 2. İndeks Yapıları
*   **FST (Finite State Transducer):** Terim sözlüğü (Term Dictionary) artık bellekte çok daha az yer kaplayan FST veri yapısı ile saklanır.
*   **B-D/KD-Trees:** Sayısal veriler ve coğrafi (geo) veriler için kullanılan bu ağaç yapıları, Lucene 10 ile çok boyutlu sorgularda %30 daha hızlı sonuç verir.

## 🚀 JSON'dan gRPC ve Protobuf'a Geçiş

2026 yılındaki en büyük değişikliklerden biri, düğümler arası (Inter-node) ve istemci (Client) iletişiminde **gRPC** protokolünün varsayılan hale gelmesidir.

| Özellik | Eski (REST/JSON) | Yeni (gRPC/Protobuf) |
|:---|:---|:---|
| **Serileştirme** | Metin tabanlı (Yavaş) | İkili (Binary) (Hızlı) |
| **Bağlantı** | HTTP/1.1 (Short-lived) | HTTP/2 (Multiplexing) |
| **Gecikme** | Orta | Çok Düşük (Low Latency) |

## 🏗️ Reader / Writer Separation (Stateless Architecture)

Geleneksel küme yapısında her veri düğümü hem yazma hem okuma yapardı. v3 ile birlikte:
1.  **Indexing Nodes (Yazıcılar):** Sadece veri girişi ve segment oluşturma yapar. Veriyi doğrudan **Remote-backed Storage**'a (S3/MinIO) yazar.
2.  **Search Nodes (Okuyucular):** Yerel diskinde veri tutmaz ("Stateless"). Remote storage'daki segmentleri önbelleğe alarak (caching) okuma yapar.

> [!TIP]
> Bu mimari sayesinde, okuma trafiği arttığında veri kopyalamaya gerek kalmadan saniyeler içinde yeni "Search Node"lar ekleyerek yatayda sınırsız ölçeklenebilirsiniz.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

# 01 - Giriş ve Mimari

OpenSearch, esnek ve ölçeklenebilir bir arama, analiz ve observability platformudur. 2026 yılı itibarıyla **v3.x** serisi ile birlikte modern AI ve agentic iş yükleri için tam destek sunmaktadır.

## 🏗️ OpenSearch Mimarisi

OpenSearch, dağıtık bir yapıya sahiptir ve verilerini **Shard** (parçacık) adı verilen birimlerde saklar.

### Temel Bileşenler:

1.  **Nodes (Düğümler):** Bir OpenSearch kümesini (cluster) oluşturan her bir çalışma birimine denir.
2.  **Indices (Dizinler):** Benzer özelliklere sahip verilerin mantıksal bir grupta saklandığı yapıdır (SQL'deki tablolar gibi).
3.  **Shards (Parçacıklar):** Dizinlerin yatayda bölünmüş halidir. Veri ölçeklenebilirliği için gereklidir.
4.  **Replicas (Yedekler):** Shard'ların kopyalarıdır, yüksek kullanılabilirlik ve arama performansı sağlar.

### 🚀 v3.x'in Önemli Özellikleri (2026):

*   **Read/Writer Separation (Okuma/Yazma Ayrımı):** Düğüm türlerinin daha net ayrımı ve performans artışı.
*   **Segment Replication:** Veri kopyalamada belge düzeyinden segment düzeyine geçiş (v3 standardı), %40'a varan yazma hızı artışı.
*   **Remote-backed Storage:** Dizin verilerinin S3 veya MinIO gibi nesne depolarında saklanarak düğümlerin "Stateless" hale getirilmesi.
*   **Native Vektör Desteği:** k-NN (k-Nearest Neighbors) aramaları için donanım hızlandırmalı motor.
*   **Agentic Search:** AI agent'ları için Model Context Protocol (MCP) üzerinden veri sunma yeteneği.
*   **Composable Query Pipelines:** Sorguların parçalara bölünüp ardışık işlenmesi.

## 💡 Neden OpenSearch?

*   **Tamamen Açık Kaynak:** Apache 2.0 lisansı ile ticari kısıtlama olmaksızın kullanım.
*   **Yüksek Ölçeklenebilirlik:** Petabaytlarca veri üzerinde gerçek zamanlı sorgulama.
*   **Zengin Eklenti Ekosistemi:** Güvenlik, ISM, Alerting ve k-NN gibi yerleşik özellikler.

---

> [!TIP]
> **Uzmanlar İçin:** Bu konunun mimari iç yapılarını (Lucene 10, gRPC, Segment Merge vb.) [01 - Mimari Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Kurulum ve Yapılandırma ➡️](../02-Kurulum-ve-Yapi-landirma/README.md)

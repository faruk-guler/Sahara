# 03 - Düğüm ve Küme Yönetimi

OpenSearch kümenizi (cluster) ölçeklendirmek ve yönetmek, sistemin istikrarı için kritiktir.

## 🏢 Düğüm Türleri (Node Roles)

Kümenizdeki her düğümün belirli bir görevi olabilir.

| Rol | Açıklama |
|:---|:---|
| `cluster_manager` | Küme koordinasyonundan sorumlu ana düğümler. |
| `data` | Verileri saklayan ve süzgeçleyen düğümler. |
| `ingest` | Veri girişinden önce `ingest pipeline` işlemlerini yapan düğümler. |
| `ml` | Makine öğrenimi işlemlerini gerçekleştiren düğümler. |
| `search` | Veri aramalarını koordine eden düğümler (2026 AI-Optimized). |

## 🏥 Küme Durumu (Cluster Health)

Küme sağlığını izlemek için API çağrısı yapabilirsiniz:

```bash
GET /_cluster/health
```

*   🟢 **Green:** Her şey yolunda, tüm shard'lar atanmış.
*   🟡 **Yellow:** Birincil shard'lar atanmış ancak bazı yedekler (replicas) eksik. 
*   🔴 **Red:** Bazı birincil shard'lar eksik, veri kaybı veya erişilemezlik söz konusu.

## 🧩 Shard ve Replikalar (Shard Management)

Shard sayısını belirlerken dikkatli olmalısınız.

### 🔄 Replication Stratejileri (v3)

1.  **Document Replication:** Her belge her replica'da ayrı ayrı dizinlenir. CPU maliyeti yüksektir.
2.  **Segment Replication (Önerilen):** Birincil shard (primary) segment oluşturur ve bunu replica'lara kopyalar. Replica tarafında CPU kullanımını ciddi oranda azaltır.

```json
PUT /my-index
{
  "settings": {
    "index": {
      "replication.type": "SEGMENT",
      "number_of_shards": 5,
      "number_of_replicas": 1
    }
  }
}
```

### İpucu: Shard Boyutlandırması
*   **Küçük Dizinler:** Tek bir shard ve bir replica yeterlidir.
*   **Büyük Dizinler (Günlükler):** Shard başına 10GB - 50GB arası veri önerilir.

---

> [!TIP]
> **Uzmanlar İçin:** Çoklu küme yönetimi (**CCS/CCR**) ve **Segment Replication** detaylarını [03 - Küme Yönetimi Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Kurulum ve Yapılandırma](../02-Kurulum-ve-Yapi-landirma/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Veri Girişi ve Dizinleme ➡️](../04-Veri-Giris-ve-Dizinleme/README.md)

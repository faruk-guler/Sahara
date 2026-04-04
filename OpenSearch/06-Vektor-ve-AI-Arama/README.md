# 06 - Vektör ve AI Arama (AI-Native)

2026 yılı itibarıyla OpenSearch, geleneksel aramanın ötesine geçerek tamamen **AI-Native** bir platform haline gelmiştir.

## 🧠 k-NN (k-Nearest Neighbors) Plugin

Vektör araması, metinlerin anlamca benzerliğini (Semantic Search) bulmanıza olanak tanır.

```json
PUT /vector-index
{
  "settings": { "index": { "knn": true } },
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "knn_vector",
        "dimension": 1536,
        "method": {
          "name": "hnsw",
          "space_type": "l2",
          "engine": "nmslib"
        }
      }
    }
  }
}
```

*   **HNSW:** Yüksek performanslı vektör arama algoritması.
*   **L2 / Cosine Similarity:** Mesafe ölçüm yöntemleri.

## 🤖 RAG (Retrieval Augmented Generation) Entegrasyonu

OpenSearch, LLM'lere (OpenAI, Anthropic, local modeller) zengin bağlam sağlamak için ideal bir veri kaynağıdır.

### İşleyiş:
1.  **Kullanıcı Sorgusu:** LLM tarafından vektöre çevrilir.
2.  **OpenSearch:** Vektör araması ile en alakalı ilk 5 dökümanı bulur.
3.  **Prompt:** Dökümanlar ve kullanıcı sorgusu birleştirilip LLM'e gönderilir.
4.  **Yanıt:** LLM, dökümanlara dayalı doğru yanıtı üretir.

## 🛡️ Search Pipelines (v3)

Search Pipelines, bir arama isteği OpenSearch'e ulaştığında veya yanıt dönmeden önce yapılacak işlemleri (örnek: Reranking, Hybrid Search birleştirme) tanımlamanıza olanak tanır.

```json
PUT /_search/pipeline/hybrid-search-pipeline
{
  "description": "Vektör ve metin aramasını birleştirir",
  "processors": [
    {
      "normalization-processor": {
        "normalization_technique": "min_max"
      },
      "combination-processor": {
        "combination_technique": "arithmetic_mean"
      }
    }
  ]
}
```

## 🔌 Model Context Protocol (MCP) Desteği

2026'da gelen **MCP**, AI agent'larının (Claude, Gemini vb.) OpenSearch kümenize güvenli bir şekilde bağlanıp veri çekmesini sağlar.

### MCP Konnektörü Örneği:
OpenSearch artık yerleşik bir MCP sunucusu olarak çalışabilir. Bir agent `list_indices` veya `execute_knn_query` araçlarını (tools) kullanarak doğrudan veri tabanınızla konuşabilir.

> [!TIP]
> MCP üzerinden agent'lara sadece "Read-Only" yetkili bir kullanıcı tanımlayarak güvenliği en üst düzeyde tutabilirsiniz.

---

> [!TIP]
> **Uzmanlar İçin:** **Hybrid Search**, **Search Pipelines** ve **Model Serving** mimarilerini [06 - AI Arama Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Arama ve Sorgulama](../05-Arama-ve-Sorgulama/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Güvenlik ve Erişim Denetimi ➡️](../07-Guvenlik-ve-Erisim-Denetimi/README.md)

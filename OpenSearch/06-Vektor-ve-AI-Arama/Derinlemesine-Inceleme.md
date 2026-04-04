# 06 - AI Arama Derinlemesine İnceleme (Hybrid Search & Reranking)

Modern arama motorlarının zirvesi olan hibrit arama (Hybrid Search) ve yeniden sıralama (Reranking) mimarilerini öğrenin.

## 🔀 Hybrid Search (Hibrit Arama)

Sadece anahtar kelime (Text) veya sadece anlam (Vector) değil; ikisini birden kullanarak en doğru sonuçları elde edin.

### Mekanizma:
1.  **Metin Araması (BM25):** "SQL Nedir" sorgusuyla tam eşleşmeleri bulur.
2.  **Vektör Araması (k-NN):** "Veritabanı yönetimi" kavramına yakın dökümanları bulur.
3.  **Normalization & Combination:** İki farklı skor türü (örneğin 0.8 ve 12.5) normalleştirilir ve birleştirilir.

```json
# Hibrit sorgu için Search Pipeline kullanımı
GET /article-index/_search?search_pipeline=hybrid-search-pipeline
{
  "query": {
    "hybrid": {
      "queries": [
        { "match": { "text": "OpenSearch AI" } },
        { "knn": { "vector": { "vector": [0.1, 0.2, ...], "k": 10 } } }
      ]
    }
  }
}
```

## ⬆️ Model Serving Framework (MSF)

OpenSearch artık bir model sunucusu (Model Server) gibi davranabilir.

### Model Entegrasyon Seçenekleri:
*   **Local Models:** ONNX veya TorchScript modellerini doğrudan OpenSearch düğümlerine yükleyin.
*   **Remote Connectors:** OpenAI, Anthropic, Bedrock veya SageMaker üzerindeki modelleri birer API olarak OpenSearch'e bağlayın.

### Kullanım:
Bu modelleri `ingest pipeline` içinde (veriyi kaydederken vektöre çevirme) veya `search pipeline` içinde (gelen aramayı vektöre çevirme) kullanabilirsiniz.

```json
# Uzak model konnektörü kaydı (OpenAI)
POST /_plugins/_ml/connectors/_create
{
  "name": "OpenAI Connector",
  "description": "Text Embedding Connector",
  "protocol": "http",
  "parameters": { "endpoint": "api.openai.com", "model": "text-embedding-3-small" ... }
}
```

## 🔄 Cross-Encoder ile Reranking (Yeniden Sıralama)

İlk aşamada (First Stage) binlerce aday arasından hızlıca ilk 100 belgeyi bulun. İkinci aşamada (Second Stage) daha zeki ama yavaş olan bir **Cross-Encoder** modeli ile bu 100 belgeyi en doğru sıraya (Precision) koyun.

> [!TIP]
> Yeniden sıralama (Reranking), RAG (Retrieval Augmented Generation) sistemlerinde LLM'e (Large Language Model) gönderilen bilginin kalitesini dramatik ölçüde artırır.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

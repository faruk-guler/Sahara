# 04 - Veri Girişi ve Dizinleme

OpenSearch'te veri saklamanın (Indexing) ve veriyi yapılandırmanın (Mapping) en iyi yollarını öğrenin.

## 📝 Dizin Eşleme (Mapping)

Dizin eşleme, bir dizindeki alanların nasıl saklandığını ve dizinlendiğini tanımlar.

```json
PUT /books
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "author": { "type": "keyword" },
      "publish_date": { "type": "date" },
      "price": { "type": "float" }
    }
  }
}
```

*   **Text:** Serbest metin araması için (Analyzer kullanır).
*   **Keyword:** Tam değer aramaları, sıralama ve gruplama için.

## 🏗️ Ingest Pipelines (İşleme Boru Hatları)

Veri dizine girmeden önce üzerinde değişiklik yapmak için (örneğin timestamp ekleme veya alanı küçük harfe çevirme) Ingest Pipelines kullanılır.

```json
PUT /_ingest/pipeline/timestamp-pipeline
{
  "description": "Her belgeye timestamp ekler",
  "processors": [
    {
      "set": {
        "field": "ingest_at",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}
```

## 🔄 Index State Management (ISM)

Dizinlerin yaşam döngüsünü (örneğin 30 gün sonra silme) otomatik yönetmek için kullanılır.

### Örnek Senaryo:
*   **Hot:** Veri yazılıyor.
*   **Warm:** Veri okunuyor ancak güncellenmiyor.
*   **Cold:** Veri nadiren okunuyor.
*   **Delete:** Veri 90 gün sonra siliniyor.

---

> [!TIP]
> **Uzmanlar İçin:** **Data Streams**, **Composed Templates** ve karmaşık **Alias** stratejilerini [04 - Veri Girişi Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Düğüm ve Küme Yönetimi](../03-Dugum-ve-Kume-Yonetimi/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Arama ve Sorgulama ➡️](../05-Arama-ve-Sorgulama/README.md)

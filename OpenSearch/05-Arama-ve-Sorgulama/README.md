# 05 - Arama ve Sorgulama

OpenSearch'te veriyi bulmak için kullanılan araçlar ve sorgulama teknikleri.

## 🔍 Query DSL (Domain Specific Language)

JSON tabanlı sorgulama dilidir.

```json
GET /books/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "OpenSearch" } }
      ],
      "filter": [
        { "range": { "price": { "lte": 100 } } }
      ]
    }
  }
}
```

*   **Match:** Serbest metin araması.
*   **Term:** Tam değer araması.
*   **Bool:** Birden fazla sorguyu birleştirme.

## 📊 SQL ve PPL (Piped Processing Language)

Daha aşina olduğunuz SQL komutları ile sorgulama yapabilirsiniz.

```sql
POST /_plugins/_sql
{
  "query": "SELECT * FROM books WHERE price < 100 ORDER BY publish_date DESC"
}
```

**PPL (Piped Processing Language):** Log analitiği için idealdir. 

```ppl
source=logs | where level='ERROR' | stats count() by service
```

## 📈 Aggregations (Toplulaştırmalar)

Verilerinizi analiz etmek ve istatistikler çıkarmak için kullanılır.

*   **Metrics:** Ortalama, toplam, min, max değerler.
*   **Buckets:** Verileri kategorilere ayırma (örneğin ay bazında satışlar).

### Örnek: Yazar bazında kitap sayısı
```json
GET /books/_search
{
  "size": 0,
  "aggs": {
    "authors_count": {
      "terms": { "field": "author" }
    }
  }
}
```

---

> [!TIP]
> **Uzmanlar İçin:** **Star-tree** indeksleri, **PIT (Point-in-Time)** ve ileri seviye **Custom Scoring** tekniklerini [05 - Sorgulama Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Veri Girişi ve Dizinleme](../04-Veri-Giris-ve-Dizinleme/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Vektör ve AI Arama ➡️](../06-Vektor-ve-AI-Arama/README.md)

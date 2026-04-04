# 05 - Sorgulama Derinlemesine İnceleme (Star-tree & PIT)

OpenSearch'te en karmaşık sorguları milisaniyeler içinde çalıştırmak ve devasa veri setlerinde güvenli gezinmek için ileri teknikler.

## 🌲 Star-tree İndeksleri (v3)

Geleneksel toplulaştırma (aggregation) sorguları, milyarlarca satırda yavaşlayabilir. **Star-tree** indeksleri, bu sorguları önceden hesaplayarak (pre-aggregate) çözüm sunar.

### Nasıl Çalışır?
*   **Boyutlar (Dimensions):** Sık sorgulanan alanlar (örn: `region`, `status`).
*   **Metrikler (Metrics):** Hesaplanacak değerler (örn: `sum(sales)`, `avg(latency)`).
*   **Yapı:** Lucene içinde saklanan bu özel yapı, 100 kat daha hızlı sonuç dönebilir.

```json
# Star-tree içeren bir dizin oluşturma
PUT /sales-index
{
  "mappings": {
    "index": {
      "star_tree": {
        "dimensions": [ "region", "product_category" ],
        "metrics": [ "sum(total_amount)", "count(sale_id)" ]
      }
    }
  }
}
```

## ⏳ Point-in-Time (PIT) ve Search After

Derin sayfalama (`from` ve `size` parametreleri ile binlerce sayfa ilerlemek) hem performansı öldürür hem de veriler değiştiği için tutarsız sonuçlar verir.

### Çözüm: PIT
*   **PIT (Point-in-Time):** Dizinin o anki halinin bir "fotoğrafını" çeker (hafif bir mekanizmadır).
*   **Search After:** Bir sonraki sayfaya geçmek için son belgenin benzersiz kimliğini (sort value) kullanır.

```json
# 1. PIT oluşturma
POST /my-index/_search/point_in_time?keep_alive=1m

# 2. PIT ile sayfalama
GET /_search
{
  "size": 100,
  "query": { "match_all": {} },
  "pit": { "id": "PIT_ID_BURAYA", "keep_alive": "1m" },
  "sort": [ { "_doc": "asc" } ],
  "search_after": [ "SON_BELGE_SORT_DEGERI" ]
}
```

## 📏 İleri Seviye Skorlama (Custom Scoring)

Sorgu sonuçlarını sadece metin eşleşmesine göre değil, kendi ağırlıklarınıza göre sıralayın.

*   **Script Score:** Kendi formülünüzü (Painless script) yazarak her belgenin skorunu hesaplatın.
*   **Rank Feature:** Belirli alanların (örn: "popülerlik" veya "tıklanma oranı") arama sonuçlarını yukarı çekmesini sağlayın.

```json
# Popülerlik alanına göre skoru artırma
GET /products/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "name": "laptop" } },
      "script": { "source": "_score * Math.log(1 + doc['popularity'].value)" }
    }
  }
}
```

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

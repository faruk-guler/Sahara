# 04 - Veri Girişi Derinlemesine İnceleme (Data Streams & Templates)

Zaman serisi (Time-series) verileri ve devasa günlükleri (logs) yönetmek için profesyonel veri akış (Data Stream) mimarilerini öğrenin.

## 🌊 Data Streams (Veri Akışları)

Data Streams, OpenSearch'te zaman damgalı (timestamp) verileri saklamanın en modern yoludur.

### Bileşenler:
1.  **Backing Indices:** Verilerin gerçekte saklandığı gizli (hidden) dizinler.
2.  **Rollover:** Belirli bir boyut (örn: 50GB) veya süre (örn: 30 gün) dolduğunda yeni bir backing index otomatik oluşturulur.
3.  **İsimlendirme:** `logs-myapp-prod` gibi tek bir isimle veri yazıp okursunuz; arka plandaki onlarca dizinle uğraşmazsınız.

```json
# Bir data stream oluşturma
PUT /_data_stream/logs-system-prod
```

## 🏗️ Composed Index Templates (Birleşik İndeks Şablonları)

Artık tek bir devasa şablon yerine, modüler (Reusable) şablon parçaları kullanabilirsiniz.

### Avantajları:
*   **Component Templates:** `settings`, `mappings` ve `aliases` gibi bölümleri ayrı ayrı tanımlayıp farklı indekslerde birleştirebilirsiniz (Mix-and-match).

```json
# Bileşen şablonu (mappings)
PUT /_component_template/runtime-mappings
{
  "template": {
    "mappings": { "properties": { "host.ip": { "type": "ip" } } }
  }
}

# Birleşik şablon
PUT /_index_template/final-template
{
  "index_patterns": ["logs-*"],
  "composed_of": ["runtime-mappings", "base-settings"]
}
```

## 🖇️ Aliases (Takma Adlar) Derinlemesine Bakış

İndeks takma adları sadece isim değişikliği değil, güçlü birer soyutlama katmanıdır:
*   **Zero-Downtime Reindexing:** Uygulamanızı durdurmadan arka plandaki dizini değiştirebilirsiniz.
*   **Filtered Aliases:** Bir takma ad üzerinden sadece belirli bir kullanıcı grubuna ait verileri süzerek gösterebilirsiniz.

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-jan-2026",
        "alias": "current-logs",
        "filter": { "term": { "status": "ERROR" } }
      }
    }
  ]
}
```

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

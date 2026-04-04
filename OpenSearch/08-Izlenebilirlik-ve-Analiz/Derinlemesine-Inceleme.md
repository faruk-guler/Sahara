# 08 - İzlenebilirlik Derinlemesine İnceleme (Anomaly Detection & RCF)

OpenSearch'te makine öğrenimi tabanlı anomali tespitinin (Anomaly Detection) teknik temellerini ve gelişmiş yapılandırmalarını keşfedin.

## 📈 Random Cut Forest (RCF) Algoritması

OpenSearch Anomali Tespiti eklentisi, AWS tarafından geliştirilen **Random Cut Forest (RCF)** algoritmasını kullanır.

### RCF Neden Güçlüdür?
*   **Streaming Veri İçin Uygundur:** Veri girişi sırasında anlık (real-time) çalışabilir; tüm geçmiş veriye ihtiyacı yoktur.
*   **Sezonsallığı Anlar:** Günlük, haftalık veya aylık periyodik değişimleri öğrenerek "normal"i buna göre günceller.
*   **Unsupervised:** Sizin için "bu bir hatadır" diyerek veriyi etiketlemenize (Labeling) gerek kalmaz; anormallikleri kendi bulur.

## 🔍 Anomali Dedektörü Yapılandırması

Bir dedektör oluştururken aşağıdaki parametrelerin derinliğini anlamak önemlidir:

*   **Detector Interval (Dakika):** Ne sıklıkla veri kontrol edilecek? (Örn: 1 dakika).
*   **Category Fields:** Veriyi bir kerede değil, parçalara ayırarak (örneğin: `host_name` bazında ayrı ayrı) analiz etmek için kullanılır.
*   **Anomaly Score:** Bir verinin ne kadar anormal olduğunu gösteren değerdir (Yüksek = Anormal).
*   **Anomaly Confidence:** Algoritmanın bu anomali skoruna ne kadar güvendiğinin yüzdesidir.

```json
# Bir anomali dedektörü oluşturma
POST /_plugins/_anomaly_detection/detectors
{
  "name": "cpu-anomaly-detector",
  "description": "Her düğümün CPU kullanımını ayrı ayrı izler",
  "time_field": "timestamp",
  "indices": [ "node-metrics-*" ],
  "feature_attributes": [
    {
      "feature_name": "avg_cpu",
      "feature_enabled": true,
      "aggregation_query": { "avg_cpu": { "avg": { "field": "cpu_usage" } } }
    }
  ],
  "filtering_field": "host_name"
}
```

## 🚨 Alerting (Uyarı Sistemi) ile Entegrasyon

Anomali tespiti, tek başına sadece bir veridir. Bunu bir **Alerting Monitor** ile bağladığınızda:

1.  **Düşük Güven (Confidence < 0.7):** Sadece loglara yaz.
2.  **Yüksek Skor (Score > 2.0):** Slack kanalına uyarı gönder.
3.  **Çok Kritik Anomali (Grade > 0.9):** On-call ekibini pager üzerinden ara.

> [!TIP]
> **Multivariate Analysis:** birden fazla alanı (örneğin hem CPU hem Bellek) tek bir "durum" olarak analiz ederek, tek başına anomali olmayan ama birlikte sorun teşkil eden durumları yakalayabilirsiniz.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

# 11 - OpenSearch Dashboards

Verilerinizi görselleştirmek ve analiz etmek için kullanılan web tabanlı arayüzdür.

## 📊 Dashboard Oluşturma

Bir bakış açısı yakalamak için farklı görselleştirme türlerini kullanın.

*   **Discover:** Verileri ham halde filtreleyin ve inceleyin.
*   **Visualize:** Pasta grafikler, çizgi grafikler ve ısı haritaları oluşturun.
*   **Dashboard:** Birçok görselleştirmeyi tek bir ekranda toplayın.

## 📐 Dizin Desenleri (Index Patterns)

Verilerinizi Dashboards'da görebilmek için önce bir index pattern oluşturmalısınız.

Örneğin: `logs-*` deseni, `logs-2026-01`, `logs-2026-02` gibi tüm dizinleri kapsar.

## 🔔 Uyarılar (Alerting)

Belirli koşullar oluştuğunda (örneğin CPU > %90 veya ERROR sayısı > 100) bildirim alın.

### Bildirim Kanalları:
*   **Slack:** Operasyon ekipleri için hızlı uyarılar.
*   **Webhook:** Dış sistemleri (Jira, Service-now) tetikleme.
*   **Email:** Detaylı raporlar.

## 🛠️ Dev Tools

Doğrudan OpenSearch API'sine JSON sorguları atmak için Dashboards üzerindeki **Dev Tools** aracını kullanın. 

### İpucu: Otomatik Tamamlama
Dev Tools, yazarken API endpoint'lerini ve parametreleri otomatik tamamlayarak işinizi kolaylaştırır.

---

[⬅️ Önceki: Yedekleme ve Kurtarma](../10-Yedekleme-ve-Kurtarma/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Sorun Giderme ve SSS ➡️](../12-Sorun-Giderme-ve-SSS/README.md)

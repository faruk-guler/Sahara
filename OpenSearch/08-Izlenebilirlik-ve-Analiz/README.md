# 08 - İzlenebilirlik ve Analiz (Observability)

Sistemlerinizin sağlığını izlemek ve performansını analiz etmek için OpenSearch'ün observability araçlarını kullanın.

## 📊 Log Analitiği

OpenSearch, petabaytlarca log verisini gerçek zamanlı olarak dizinlemek ve sorgulamak için tasarlanmıştır.

*   **Fluentd / Fluent Bit:** Log verilerini toplamak ve OpenSearch'e göndermek için yaygın kullanılır.
*   **Logstash:** Verileri dönüştürmek (enrichment) ve filtrelemek için güçlü bir araçtır.

## 📈 Metrik İzleme

Metrik verileri (CPU, RAM, istek sayıları vb.) zaman serisi (Time-series) olarak saklanabilir.

### OpenTelemetry (OTel) Desteği
2026 yılı itibarıyla OpenSearch, **OpenTelemetry** protokolü ile tam uyumludur. Uygulamalarınızdan gelen izleri (Tracing) ve metrikleri doğrudan kabul eder.

## 🔍 Uygulama Performans İzleme (APM)

OpenSearch Dashboards içindeki APM eklentisi ile şunları görebilirsiniz:
1.  **Hizmet Haritası:** Mikroservisler arasındaki etkileşim.
2.  **Hata Oranları:** Hangi servislerde darboğazlar var?
3.  **Trace Detayları:** Tek bir isteğin tüm servislerdeki yolculuğu.

---

> [!TIP]
> **Uzmanlar İçin:** **Anomali Tespit (ML)** motoru, **Random Cut Forest (RCF)** algoritması ve otomatik uyarı mekanizmalarını [08 - İzlenebilirlik Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Güvenlik ve Erişim Denetimi](../07-Guvenlik-ve-Erisim-Denetimi/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Performans Optimizasyonu ➡️](../09-Performans-Optimizasyonu/README.md)

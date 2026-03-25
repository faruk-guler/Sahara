# Modül 15: Dashboard Özelleştirme ve Raporlama

Wazuh Dashboard (OpenSearch Dashboards tabanlı), verilerinizi görselleştirmenizi ve yönetici raporları hazırlamanızı sağlar.

## Hazır Dashboard'lar

Wazuh, kurulumla birlikte birçok ön tanımlı dashboard sunar:

- **Security Events:** Genel güvenlik olayları özeti.
- **Integrity Monitoring:** Dosya değişiklikleri takibi.
- **Vulnerabilities:** Zafiyet tarama sonuçları.
- **Regulatory Compliance:** PCI DSS, GDPR vb. uyumluluk raporları.

## Özel Dashboard Oluşturma

İhtiyacınıza yönelik dashboard'lar oluşturmak için:

1. **Discover:** Verileri filtreleyin ve arayın.
2. **Visualize:** Grafik türlerini (Pie chart, Bar chart, Heatmap) seçin.
3. **Dashboard:** Görselleri bir araya getirin.

## Veri Süzme (DSL Query)

Wazuh Dashboard'da KQL (Kibana Query Language) kullanılır.

**Örnek:** `rule.level: >10 AND data.srcip: 192.168.*`

## Raporlama

- **Zamanlanmış Raporlar:** Haftalık veya aylık özetlerin otomatik olarak e-posta ile gönderilmesini sağlayabilirsiniz.

## Rol Tabanlı Erişim Kontrolü (RBAC)

Wazuh, farklı kullanıcılar için farklı yetki seviyeleri tanımlamanıza olanak tanır:

- **ReadOnly:** Sadece alarmleri görebilen kullanıcılar.
- **AgentManager:** Ajan ekleme/silme yetkisi olanlar.
- **Admin:** Tam yetkili yöneticiler.

Bu ayarlar Dashboard üzerindeki "Security" sekmesi altından yönetilir.

---

[README'ye Dön](README.md)

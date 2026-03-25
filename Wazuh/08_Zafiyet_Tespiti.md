# Modül 08: Zafiyet Tespiti (Vulnerability Detection)

Wazuh, uç noktalarda yüklü olan yazılımları tarayarak bilinen zafiyetleri (CVE) tespit eder.

## Çalışma Mantığı

1. **Envanter Toplama:** Agent, sistemde yüklü paketlerin listesini (versiyonlarıyla birlikte) `Syscollector` aracılığıyla toplar ve Manager'a gönderir.
2. **Veritabanı Güncelleme:** Manager; Canonical (Ubuntu), Red Hat, Debian, Microsoft ve NVD (National Vulnerability Database) gibi kaynaklardan CVE verilerini çeker.
3. **Eşleştirme:** Manager, envanter verilerini CVE veritabanıyla karşılaştırır ve eşleşen zafiyetleri raporlar.

## Konfigürasyon (Manager Tarafı)

Zafiyet tarayıcısını etkinleştirmek için Manager üzerindeki `ossec.conf` dosyasında:

```xml
<vulnerability-detector>
  <enabled>yes</enabled>
  <interval>5m</interval>
  <min_full_scan_interval>6h</min_full_scan_interval>
  <run_on_start>yes</run_on_start>

  <!-- İşletim sistemi beslemeleri -->
  <provider name="canonical">
    <enabled>yes</enabled>
    <os>trusty</os>
    <os>xenial</os>
    <os>bionic</os>
    <os>focal</os>
    <os>jammy</os>
    <update_interval>1h</update_interval>
  </provider>

  <provider name="nvd">
    <enabled>yes</enabled>
    <update_interval>1h</update_interval>
  </provider>
</vulnerability-detector>
```

## Önem Dereceleri (Severity Levels)

Tespit edilen zafiyetler şu şekilde sınıflandırılır:

- **Critical:** Acil müdahale gerektiren, uzaktan kod çalıştırma (RCE) riski taşıyan zafiyetler.
- **High:** Sistem güvenliğini ciddi şekilde tehdit eden zafiyetler.
- **Medium / Low:** Daha düşük riskli veya istismarı zor zafiyetler.

## Çözüm Süreci

Wazuh, sadece zafiyeti söylemekle kalmaz, hangi paketin hangi versiyona yükseltilmesi gerektiğini de belirtir.

---
[README'ye Dön](README.md)

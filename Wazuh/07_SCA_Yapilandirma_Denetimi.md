# Modül 07: Güvenlik Yapılandırma Değerlendirmesi (SCA)

Security Configuration Assessment (SCA), sistemlerin belirli güvenlik standartlarına (CIS Benchmarks, NIST vb.) ne kadar uyumlu olduğunu denetler.

## SCA'nın Amacı

Sistemdeki yanlış yapılandırmaları, zayıf şifre politikalarını ve açık bırakılmış gereksiz servisleri tespit ederek saldırı yüzeyini azaltmaktır.

## Nasıl Çalışır?

Wazuh Manager, Agent'lara YAML formatında poliçe dosyaları gönderir. Agent bu poliçelerdeki kuralları (check) çalıştırır ve "Pass" veya "Fail" sonucunu Manager'a raporlar.

## Örnek SCA Kontrolü (YAML)

```yaml
policy:
  id: "password_policy"
  description: "Şifre karmaşıklığı kontrolü"
  
checks:
  - id: 1001
    title: "Minimum şifre uzunluğu 12 olmalı"
    condition: all
    rules:
      - 'f:/etc/login.defs -> r:^PASS_MIN_LEN\s+12'
```

## Konfigürasyon (`ossec.conf`)

```xml
<sca>
  <enabled>yes</enabled>
  <scan_on_start>yes</scan_on_start>
  <interval>12h</interval>
  <policies>
    <policy>/var/ossec/etc/shared/cis_ubuntu22-04_v1.0.0.yml</policy>
  </policies>
</sca>
```

## Dashboard Üzerinden İzleme

Wazuh Dashboard'daki "SCA" sekmesi üzerinden:

- Toplam uyumluluk skoru (%).
- Başarısız olan kontrollerin detayları ve çözüm önerileri.
- Zaman içindeki değişim grafikleri izlenebilir.

---
[README'ye Dön](README.md)

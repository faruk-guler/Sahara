# Modül 11: Regülasyon Uyumluluğu (Compliance)

Wazuh; PCI DSS, GDPR, HIPAA, NIST ve SOC2 gibi uluslararası regülasyonlara uyum sürecini otomatize eder.

## Uyumluluk Nasıl Sağlanır?

Wazuh, topladığı olayları (events) otomatik olarak regülasyon maddeleriyle eşleştirir. Örneğin, bir dosya değişikliği hem "FIM" kuralını tetikler hem de "PCI DSS 11.5" maddesiyle ilişkilendirilir.

## Desteklenen Standartlar

1. **PCI DSS:** Kartlı ödeme sistemleri güvenliği.
2. **GDPR:** Kişisel verilerin korunması (Avrupa Birliği).
3. **HIPAA:** Sağlık verileri güvenliği.
4. **NIST 800-53:** ABD kamu güvenliği standartları.
5. **TSC (SOC2):** Hizmet organizasyonları için güvenilirlik prensipleri.

## Örnek Kural Eşleştirmesi

Bir kuralın içinde regülasyon etiketi şu şekilde görünür:

```xml
<rule id="5715" level="3">
  <if_sid>5700</if_sid>
  <match>^sshd: ssmp: auth password</match>
  <description>sshd: authentication success.</description>
  <group>authentication_success,pci_dss_10.2.5,gpg13_7.1,gdpr_12.2,</group>
</rule>
```

## Otomatik Etiketleme

Wazuh kuralları varsayılan olarak regülasyon etiketleriyle gelir. Bu sayede özel bir kural yazmasanız bile, sistemdeki olaylar otomatik olarak ilgili regülasyon maddesiyle (Örn: `pci_dss_10.2.5`) eşleştirilir.

## Dashboard Üzerinden Filtreleme

Wazuh Dashboard'da "Inventory" veya "Events" sekmelerinde regülasyon maddesine göre filtreleme yaparak, denetim sırasında sadece ilgili kanıtları (logs) listeleyebilirsiniz.

---

[README'ye Dön](README.md)

# Modül 12: Bulut Güvenliği (Cloud Security)

Wazuh, popüler bulut sağlayıcıları (AWS, Azure, GCP) ile entegre olarak "Cloud Security Posture Management" (CSPM) sağlar.

## 1. AWS Entegrasyonu

Wazuh, AWS kaynaklarını izlemek için "CloudWatch" ve "S3" servislerini kullanır.

- **İzlenebilen Servisler:** CloudTrail (API logları), VPC Flow Logs (Ağ trafiği), GuardDuty (Tehdit algılama), S3 (Dosya erişimi).
- **Gereksinimler:** Wazuh'un S3 bucket'ına erişebilmesi için `s3:GetObject`, `s3:ListBucket` ve `s3:DeleteObject` (isteğe bağlı) izinlerine sahip bir IAM kullanıcısı/rolü gerekir.
- **Kurulum:** Wazuh Manager tarafında bir `module` tanımlanır ve AWS IAM Role/Key bilgileri girilir.

```xml
<wodle name="aws-s3">
  <disabled>no</disabled>
  <interval>10m</interval>
  <run_on_start>yes</run_on_start>
  <bucket name="my-cloudtrail-bucket">
    <access_key>YOUR_ACCESS_KEY</access_key>
    <secret_key>YOUR_SECRET_KEY</secret_key>
  </bucket>
</wodle>
```

## 2. Microsoft Azure Entegrasyonu

Azure Log Analytics ve Microsoft Graph API üzerinden veri toplar.

- **Kapsam:** Azure AD (Giriş logları), Azure Activity (Kaynak değişiklikleri), Microsoft Defender.
- **Yöntem:** Azure Event Hubs veya Storage Blobs üzerinden loglar çekilir.

## 3. Google Cloud (GCP) Entegrasyonu

GCP Pub/Sub mekanizmasını kullanarak logları gerçek zamanlı olarak alır.

- **Kapsam:** Audit Logs, VPC Flow Logs, Firewall Logs.

## Bulut Güvenliğinin Avantajları

- **Görünürlük:** Tüm bulut hesaplarınızın güvenliğini tek bir merkezden izleyin.
- **Yanlış Yapılandırma Tespiti:** Herkese açık (public) bırakılmış S3 bucket'larını veya zayıf firewall kurallarını algılayın.

---

[README'ye Dön](README.md)

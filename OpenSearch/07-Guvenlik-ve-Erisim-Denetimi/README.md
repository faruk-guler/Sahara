# 07 - Güvenlik ve Erişim Denetimi

Verilerinizin güvenliğini sağlamak için OpenSearch Güvenlik Eklentisi (Security Plugin) kullanılır.

## 🔐 TLS/SSL Yapılandırması

Düğümler arası (`transport`) ve istemci-düğüm (`rest`) iletişiminde şifreleme zorunludur.

```yaml
plugins.security.ssl.transport.enforce_hostname_verification: true
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: esnode.pem
plugins.security.ssl.http.pemkey_filepath: esnode-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem
```

## 👥 Rol Tabanlı Erişim Denetimi (RBAC)

Kullanıcılara sadece ihtiyaç duydukları verilere erişim yetkisi verin.

| Kavram | Açıklama |
|:---|:---|
| **User (Kullanıcı):** | Kişi veya servis hesabı. |
| **Role (Rol):** | İzinlerin toplandığı yapı (örneğin: `read_only`). |
| **Role Mapping:** | Kullanıcının bir role atanması. |

### Örnek: `read_only` Rolü
```json
{
  "cluster_permissions": [ "cluster_composite_ops_ro" ],
  "index_permissions": [
    {
      "index_patterns": [ "logs-*" ],
      "allowed_actions": [ "read" ]
    }
  ]
}
```

## 🆔 Kimlik Doğrulama (Authentication)

OpenSearch birçok yöntemi destekler:
*   **Basic Auth:** Kullanıcı adı/şifre.
*   **LDAP/Active Directory:** Kurumsal dizin entegrasyonu.
*   **SAML / OpenID Connect:** Tek oturum açma (SSO) için.

> [!CAUTION]
> Tüm varsayılan şifreleri (`admin`, `kibanaserver`) kurumsal standartlara göre güncelleyin.

---

[⬅️ Önceki: Vektör ve AI Arama](../06-Vektor-ve-AI-Arama/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: İzlenebilirlik ve Analiz ➡️](../08-Izlenebilirlik-ve-Analiz/README.md)

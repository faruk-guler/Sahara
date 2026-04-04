# 07 - Kurumsal Güvenlik Derinlemesine İnceleme (Advanced Security)

OpenSearch'te güvenliği sadece açmak değil, onu kurumsal kimlik yönetimi (IAM) ve ince taneli erişim (Fine-Grained Access Control) ile entegre etmek uzmanlık gerektirir.

## 🔑 Kurumsal Kimlik Entegrasyonu (SAML & OIDC)

OpenSearch Dashboards'ı kurumunuzun SSO (Single Sign-On) sistemine bağlamak için şu v3 parametreleri kritiktir:

```yaml
# config.yml
authc:
  saml_auth_domain:
    http_enabled: true
    transport_enabled: true
    order: 1
    http_authenticator:
      type: saml
      challenge: true
      config:
        idp:
          metadata_url: "https://your-idp.com/metadata"
          entity_id: "your-sp-entity-id"
        sp:
          entity_id: "opensearch-dashboards"
        roles_key: "groups" # SAML içindeki grup bilgisi
```

## 🛡️ İnce Taneli Erişim Denetimi (FGAC)

Verileri sadece dizin (index) seviyesinde değil, alan (field) ve belge (document) seviyesinde de koruyun.

### 1. Alan Seviyesinde Güvenlik (FLS - Field Level Security)
Bir kullanıcı `employees` dizinini görebilir ancak `salary` alanını görmesi engellenebilir.

```json
# Role definition
{
  "index_permissions": [
    {
      "index_patterns": ["employees-*"],
      "allowed_actions": ["read"],
      "fls": {
        "exclude": ["salary", "bank_account_number"]
      }
    }
  ]
}
```

### 2. Belge Seviyesinde Güvenlik (DLS - Document Level Security)
Bir bölge müdürü sadece kendi bölgesine (`region: "TR"`) ait belgeleri görebilir.

```json
# Role definition
{
  "index_permissions": [
    {
      "index_patterns": ["sales-*"],
      "allowed_actions": ["read"],
      "dls": "{\"term\": {\"region\": \"TR\"}}"
    }
  ]
}
```

## 👥 Multi-Tenancy (Çoklu Kiracılık)

OpenSearch Dashboards içerisinde her departmana (BT, IK, Finans) izole bir çalışma alanı (Tenant) sağlayın.

*   **Global Tenant:** Tüm yöneticilere açık ortak alan.
*   **Private Tenant:** Sadece kullanıcının kendine özel alanı.
*   **Custom Tenants:** Belirli bir grup (rol) için paylaşılan alan.

> [!CAUTION]
> Multi-tenancy yapılandırmasında `kibana_server` kullanıcısının yetkilerini kısıtlamayın; aksi takdirde index pattern'leri ve görselleştirmeler senkronize edilemez.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

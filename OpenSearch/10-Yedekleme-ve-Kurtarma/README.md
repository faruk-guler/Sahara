# 10 - Yedekleme ve Kurtarma (Snapshot)

Veri güvenliği için OpenSearch Snapshot (Anlık Görüntü) mekanizmasını kullanmak elzemdir.

## 💾 Snapshot Repository (Depo) Kaydı

Snapshot'lar bir repo içinde saklanmalıdır. En yaygın olanı S3 veya MinIO'dur.

```json
PUT /_snapshot/my_s3_repository
{
  "type": "s3",
  "settings": {
    "bucket": "my-opensearch-backups",
    "region": "us-west-2"
  }
}
```

## 📸 Manuel Snapshot Oluşturma

Hızlıca bir yedek almak için:

```json
PUT /_snapshot/my_s3_repository/snapshot_1
{
  "indices": "index_1,index_2",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

## 🔄 Otomatik Yedekleme (SM - Snapshot Management)

Snapshot management özelliği ile belirli periyotlarda otomatik yedekleme yapabilirsiniz.

*   **Pazartesi 02:00:** Haftalık tam yedekleme.
*   **Her Gün:** Artımlı (Incremental) yedekleme.

## 🆘 Veri Kurtarma (Restore)

Kayıp bir dizini geri yüklemek için:

```json
POST /_snapshot/my_s3_repository/snapshot_1/_restore
{
  "indices": "index_1",
  "rename_pattern": "index_(.+)",
  "rename_replacement": "restored_index_$1"
}
```

> [!IMPORTANT]
> Geri yükleme işlemi sırasında hedef dizin zaten varsa hata alırsınız. Dizin ismini değiştirmek için `rename_replacement` kullanabilirsiniz.

---

> [!TIP]
> **Uzmanlar İçin:** **Disaster Recovery (DR)** seviyeleri, **CCR (Kopyalama)** senaryoları ve bozuk shard kurtarma tekniklerini [10 - Yedekleme Derinlemesine İnceleme](./Derinlemesine-Inceleme.md) dosyasından öğrenebilirsiniz.

---

[⬅️ Önceki: Performans Optimizasyonu](../09-Performans-Optimizasyonu/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: OpenSearch Dashboards ➡️](../11-OpenSearch-Dashboards/README.md)

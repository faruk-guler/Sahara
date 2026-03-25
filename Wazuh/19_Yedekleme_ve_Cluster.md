# Modül 19: Cluster Yapısı ve Yedekleme

Büyük çaplı ortamlarda sürekliliği sağlamak için Cluster yapısı ve yedekleme stratejileri hayati önem taşır.

## 1. Wazuh Manager Cluster

Birden fazla Manager sunucusunu birbirine bağlayarak yük dengeleme (load balancing) yapabilirsiniz.

- **Master Node:** Konfigürasyon ve kural merkezi.
- **Worker Nodes:** Ajanlardan gelen logları işler ve Master ile senkronize olur.

### Cluster Kurulumu (`ossec.conf`)

  </nodes>
</cluster>
```

### Senkronizasyon Detayları

Cluster yapısında şu veriler otomatik olarak senkronize edilir:

- **Client Keys:** Tüm ajanların yetkilendirme anahtarları Master'dan Worker'lara dağıtılır.
- **Rules/Decoders:** Master üzerinde yapılan her kural değişikliği Worker'lara anlık iletilir.
- **Agent Status:** Ajanların hangi node'a bağlı olduğu bilgisi cluster genelinde paylaşılır.

## 2. Indexer (Veri) Yedekleme

Indexer verilerini (alarmleri) yedeklemek için "Snapshot" mekanizması kullanılır.

- **S3 Bucket:** Bulut üzerinde yedekleme.
- **NFS / Local:** Yerel depolama alanı.

### Snapshot Komutu (API)

```bash
PUT /_snapshot/my_backup_repo
```

## 3. Manager Backup

Kritik dosya ve dizinlerin (rules, decoders, config) periyodik olarak `tar` ile yedeği alınmalıdır:

- `/var/ossec/etc/`
- `/var/ossec/ruleset/`
- `/var/ossec/queue/db/`

---

[README'ye Dön](README.md)

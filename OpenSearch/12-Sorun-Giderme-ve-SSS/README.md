# 12 - Sorun Giderme ve SSS (Troubleshooting)

OpenSearch kümenizde karşılaşabileceğiniz yaygın sorunlar ve bunların çözümleri.

## 🛠️ Yaygın Hatalar

### 1. **Circuit Breaker Hatası (Memory)**
*   **Hata:** `[parent] Data too large`
*   **Neden:** OpenSearch heap belleği dolmuş.
*   **Çözüm:** Bellek kullanımını analiz edin (`GET /_nodes/stats/breaker`), daha fazla RAM atayın veya sorgu karmaşıklığını azaltın.

### 2. **Unassigned Shards (Atanmamış Shard'lar)**
*   **Hata:** Küme durumu "Yellow" veya "Red".
*   **Neden:** Yeterli düğüm (node) yok, disk dolmuş veya shard limitlerine ulaşıldı.
*   **Çözüm:** `GET /_cluster/allocation/explain` komutuyla nedenini öğrenin.

### 3. **401 Unauthorized / Authentication Error**
*   **Hata:** `Security is not initialized`.
*   **Neden:** Güvenlik yapılandırması tamamlanmamış veya `securityadmin.sh` çalıştırılmamış.
*   **Çözüm:** Yapılandırmayı kontrol edin ve gerekirse güvenliği yeniden başlatın.

## 📄 Log Analizi

Hata anında bakılması gereken ilk yer düğüm günlükleridir (logs).

*   **Linux:** `/var/log/opensearch/opensearch.log`
*   **Docker:** `docker logs <container_id>`

## ❓ Sıkça Sorulan Sorular (SSS)

**S: OpenSearch, Elasticsearch ile tam uyumlu mu?**
C: OpenSearch, Elasticsearch 7.10 sürümünden türetilmiştir. Ancak v2.0 ve özellikle v3.0 (2026) sürümlerinde mimari önemli ölçüde farklılaşmıştır. Geriye dönük uyumluluk çoğu API için korunsa da yeni özellikleri (k-NN, PPL vb.) kullanmak için OpenSearch'e özgü istemciler önerilir.

**S: Kaç Cluster Manager düğümüm olmalı?**
C: Yüksek kullanılabilirlik için her zaman **3 (veya tek sayılı)** Cluster Manager (v3 öncesinde Master) düğümü olması önerilir.

---

[⬅️ Önceki: OpenSearch Dashboards](../11-OpenSearch-Dashboards/README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

# 03 & 10 - Yüksek Kullanılabilirlik ve Felaket Kurtarma (HA & DR)

Bir OpenSearch kümesini sadece kurmak değil, kıtalararası ölçekte hatasız (Fault-tolerant) ve kesintisiz (Continuous availability) tasarlamak uzmanlık seviyesidir.

## 🏛️ 3-Bölgeli (3-Region) Mimari Tasarımı

En güvenilir kurumsal (Enterprise) yapı, verilerin üç farklı coğrafi bölgeye (Region) veya üç farklı binaya (AZ - Availability Zone) dağıtıldığı mimaridir.

### Senaryo: Active-Active-Passive (DR)

1.  **Region A (Canlı):** Birincil yazma ve okuma trafiği buradadır. Master (Cluster Manager) düğümlerinin 1 tanesi buradadır.
2.  **Region B (Canlı):** İkincil okuma trafiği buradadır. Master (Cluster Manager) düğümlerinin 1 tanesi buradadır.
3.  **Region C (Gözlemci/Pasif):** Master (Cluster Manager) düğümlerinin sonuncusu buradadır. Bir bölgenin tam çökmesi durumunda Master Quorum (%51) sağlanması için kritik roldedir.

## 🔁 Cross-Cluster Replication (CCR) Derinleşmesi

CCR, verileri fiziksel olarak kopyalar. Ancak uygulama katmanında bu geçişin (failover) nasıl yönetileceği asıl zorluktur.

### Failover Stratejileri:
*   **Active-Passive:** Birincil küme çöktüğünde trafik DNS veya Load Balancer seviyesinde ikincil kümeye yönlendirilir. Uygulama tarafında write-retry mekanizması olmalıdır.
*   **Active-Active with CCR:** İki kümede de okuma yapılabilir. Yazma sadece Leader kümededir. v3 ile birlikte global bir load balancer, en düşük gecikmeye sahip bölgeye okuma trafiğini otomatik yönlendirir.

## 🛰️ Cross-Cluster Search (CCS) Uzmanlığı

CCS, tek bir istek ile on binlerce düğümü taramanızı sağlar. Ancak performans için şu ayarlar kritiktir:

```json
# CCS Performans Ayarları
GET /remote_cluster_name:index_name/_search
{
  "query": { "match_all": {} },
  "ccs_minimize_roundtrips": true, # Gereksiz ağ paketlerini azaltır.
  "pre_filter_shard_size": 128     # Küçük shard'ları sorgulamadan önce elemeyi dener.
}
```

## 🚑 Split-Brain Senaryosu ve Önleme

İki bölge arasındaki ağ bağlantısı koptuğunda, her iki taraf da kendini "Master" ilan ederse veri tutarsızlığı (Split-Brain) oluşur.

*   **Çözüm:** `cluster.initial_cluster_manager_nodes` parametresini her zaman tek sayılı (3, 5, 7) düğümden seçin ve Master rolünü sadece bu düğümlere verin.
*   **Quorum Kontrolü:** Ağ koptuğunda, çoğunluğu (n/2 + 1) sağlayamayan taraf kendini otomatik olarak "ReadOnly" moduna çeker veya arama isteklerini reddeder.

> [!IMPORTANT]
> **RTO (Recovery Time Objective):** Bir felaket anında sistemin yeniden ayağa kalkma süresi. CCR ile tasarlanan sistemlerde RTO milisaniyeler-saniyeler arasındadır. Snapshot (Yedekleme) ile tasarlanan sistemlerde RTO saatlerce olabilir.

---

[🏠 Ana Sayfaya Dön](../../README.md)

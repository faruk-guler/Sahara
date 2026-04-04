# 10 - Yedekleme ve Kurtarma Derinlemesine İnceleme (DR Stratejileri)

Verinizi kaybetmemek ve felaket durumunda (Disaster) en kısa sürede ayağa kalkmak için ileri seviye stratejileri keşfedin.

## 📦 Snapshot Repository (Depo) Seçenekleri

Snapshot'larınızı saklayacağınız yerin hızı ve maliyeti önemlidir.

*   **S3 / MinIO:** Ölçeklenebilir, dayanıklı ve düşük maliyetlidir. En yaygın seçimdir.
*   **GCS (Google Cloud Storage):** Google Cloud üzerinde çalışan kümeler için idealdir.
*   **Azure Blob Storage:** Azure üzerinde bulut-yerel yedekleme sağlar.
*   **HDFS:** Hadoop ekosistemindeki devasa veriler için kullanılır.

## 🛡️ Felaket Kurtarma (Disaster Recovery - DR) Seviyeleri

Kümenizi nasıl koruyacağınızı belirlerken RTO (Recovery Time Objective) ve RPO (Recovery Point Objective) değerleri dikkate alınır.

| Seviye | Adı | Açıklama | Maliyet |
|:---|:---|:---|:---|
| **L1** | Snapshot | Sadece yedek alır, bir felaket anında geri yükleme zaman alır. | Düşük |
| **L2** | Hot Standby | Cross-Cluster Replication (CCR) ile başka bir kümede veriler hazırdır. | Yüksek |
| **L3** | Active-Active | İki küme de canlıdır, Cross-Cluster Search (CCS) ile ikisi de kullanılır. | Çok Yüksek |

### Örnek: L2 (CCR) ile DR Senaryosu
İstanbul (Leader) kümenizdeki tüm yazma işlemleri gerçek zamanlı olarak Londra (Follower) kümenize kopyalanır. İstanbul'da bir elektrik kesintisi olduğunda, Londra kümesini "Leader" yaparak dakikalar içinde canlıya çıkabilirsiniz.

## 🔄 Shrink ve Split API (Dizin Yönetimi)

Snaphot almadan önce veya sonra dizin yapısını performans için değiştirebilirsiniz:

*   **Shrink API:** Çok fazla shard'ı olan küçük bir dizini (örneğin 10'dan 1'e) birleştirerek okuma performansını artırır.
*   **Split API:** Veri miktarı tahmin edilenden fazla büyüdüğünde shard sayısını artırmak için kullanılır.

## 🚑 Bozuk Shard Kurtarma (Advanced Recovery)

Düğüm (Node) kayıpları sonrası ortaya çıkan bozulmaları çözmek için:

1.  **Allocate Explain:** `GET /_cluster/allocation/explain` ile shard'ın neden atanmadığını bulun.
2.  **Reroute API:** Shard'ı manuel olarak başka bir düğüme atayın (Move) veya boş bir shard oluşturun (Allocate empty).

> [!CAUTION]
> `allocate_empty_primary` komutu veri kaybına neden olur. Sadece son çare olarak, dizini en azından tekrar erişilebilir kılmak için kullanılmalıdır.

---

[⬅️ Kategori Ana Sayfasına Dön](./README.md) | [🏠 Ana Sayfaya Dön](../../README.md)

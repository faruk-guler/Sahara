# 02 - Kurulum ve Yapılandırma

OpenSearch, esnek dağıtım seçenekleri (Docker, Kubernetes, Linux) ile her ortama uyum sağlar.

## 🐳 Docker ile Kurulum

Hızlı testler ve yerel geliştirme için Docker en pratik yoldur.

```bash
docker run -d \
  --name opensearch-node \
  -p 9200:9200 \
  -p 9600:9600 \
  -e "discovery.type=single-node" \
  -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=<Guclu_Sifre>" \
  opensearchproject/opensearch:latest
```

> [!WARNING]
> Varsayılan yönetici şifresini (`admin`) değiştirmeyi unutmayın.

## ☸️ Kubernetes (Helm) ile Kurulum

Üretim ortamları için Helm Chart kullanımı önerilir.

```bash
# Repo ekleme
helm repo add opensearch https://opensearch-project.github.io/helm-charts/
helm repo update

# Kurulum
helm install opensearch-cluster opensearch/opensearch
```

## ⚙️ Yapılandırma (`opensearch.yml`)

Düğüm ayarları `config/opensearch.yml` dosyasından yönetilir.

| Parametre | Açıklama |
|:---|:---|
| `cluster.name` | Kümenizin benzersiz adı. |
| `node.name` | Düğümün adı (örneğin: node-1). |
| `network.host` | Dinlenecek IP adresi (varsayılan 0.0.0.0). |
| `discovery.seed_hosts` | Diğer düğümlerin IP adresleri. |
| `cluster.initial_cluster_manager_nodes` | İlk yönetici (Cluster Manager) adayları. |

---

[⬅️ Önceki: Giriş ve Mimari](../01-Giris-ve-Mimari/README.md) | [🏠 Ana Sayfaya Dön](../../README.md) | [Sıradaki: Düğüm ve Küme Yönetimi ➡️](../03-Dugum-ve-Kume-Yonetimi/README.md)

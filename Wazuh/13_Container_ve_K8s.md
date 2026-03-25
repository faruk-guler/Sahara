# Modül 13: Container ve Kubernetes Güvenliği

Modern altyapılarda Docker ve Kubernetes (K8s) ortamlarının izlenmesi kritiktir.

## 1. Docker İzleme

Wazuh, Docker host'u üzerindeki olayları ve container loglarını izleyebilir.

- **Docker Listener:** Container'ların başlatılması, durdurulması veya imaj değişikliklerini loglar.
- **Konfigürasyon:**

```xml
<wodle name="docker-listener">
  <disabled>no</disabled>
</wodle>
```

## 2. Kubernetes İzleme

Kubernetes ortamında güvenlik iki katmanda sağlanır:

### Host Seviyesi

K8s node'larına (Worker/Master) standart Wazuh Agent kurularak işletim sistemi seviyesinde izleme yapılır.

### Cluster Seviyesi (Audit Logs)

Kubernetes API Server tarafından üretilen "Audit Logs" verileri analiz edilir.

- Kim erişti?
- Hangi pod oluşturuldu?
- Hangi konfigürasyon (ConfigMap) değişti?

## 3. Container İçinde Agent Kullanımı (Sidecar)

Bazı durumlarda, her bir pod'un içine bir agent (sidecar) eklenerek container içi süreçler detaylıca izlenebilir.

## 4. Tehdit Algılama Senaryoları

- Container kaçış (escape) denemeleri.
- Privileged container başlatılması.
- Hassas dosyalara (`/etc/shadow` vb.) container içinden erişim.

---

[README'ye Dön](README.md)

# Modül 02: Kurulum Senaryoları

Wazuh kurulumu, altyapınızın büyüklüğüne ve ihtiyaçlarınıza göre iki ana şekilde yapılabilir.

## 1. All-in-one Kurulum

Tüm bileşenlerin (Indexer, Server, Dashboard) tek bir sunucu üzerine kurulduğu senaryodur.

- **Kullanım Yeri:** Küçük ortamlar, test labları, 100'den az agent.
- **Avantajı:** Hızlı ve kolay kurulum.
- **Dezavantajı:** Kapasite sınırı ve tek nokta hatası (Single Point of Failure).

### Hızlı Kurulum (Quick Installation)

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash wazuh-install.sh -a
```

## 2. Distributed (Dağıtık) Kurulum

Bileşenlerin farklı sunuculara (veya cluster yapılarına) dağıtıldığı senaryodur.

- **Kullanım Yeri:** Kurumsal ortamlar, yüksek yük gerektiren durumlar.
- **Bileşenler:**
  - Multi-node Indexer Cluster
  - Multi-node Manager Cluster
  - Load Balancer (Agent trafiği için)

## Sistem Gereksinimleri (Önerilen)

| Bileşen | CPU | RAM | Disk |
| :--- | :--- | :--- | :--- |
| All-in-one | 8 Core | 16 GB | 100 GB+ |
| Manager Node | 4 Core | 8 GB | 20 GB |
| Indexer Node | 4 Core | 8 GB | 100 GB+ |

Wazuh bileşenleri arasındaki iletişim TLS ile şifrelenir. Dağıtık kurulumda `wazuh-certs-tool.sh` kullanılarak tüm node'lar için sertifika üretilmelidir.

## 4. Güvenlik Sıkılaştırma (Post-Install)

Kurulum bittikten sonra mutlaka yapılması gerekenler:

- **Şifre Değişimi:** Varsayılan `admin` şifresini `wazuh-passwords-tool.sh` ile değiştirin.
- **API Güvenliği:** 55000 portuna erişimi sadece güvenilir IP'lerle kısıtlayın.
- **Log Rotasyonu:** Disk dolmasını engellemek için `/var/ossec/etc/internal_options.conf` üzerinden rotasyon ayarlarını kontrol edin.

---

[README'ye Dön](README.md)

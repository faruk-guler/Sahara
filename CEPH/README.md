# 📚 Ceph Master Dokümantasyon Kütüphanesi

Bu dizin, Ceph depolama kümesinin kurulumu, yönetimi, güvenliği ve sorun giderilmesi için hazırlanmış kapsamlı rehberleri içerir.

---

## 🧭 Hızlı Başlangıç Rotası (Quick Start)

Adım adım Ceph uzmanlığına giden yol:

1. **Öğren:** [Ceph Kavramsal Rehberi](ceph-concepts-guide.md) ile mimariyi kavra.
2. **Kur:** [Ceph Kurulum Rehberi](ceph-installation-guide.md) ile cluster'ı ayağa kaldır.
3. **Yapılandır:** [Pool Yönetimi](ceph-pool-guide.md) ile ilk depolama alanını aç.
4. **Kullan:** [İstemci Rehberi](ceph-client-guide.md) ile diskleri sunuculara bağla.
5. **İzle:** [İzleme Rehberi](ceph-monitoring-guide.md) ile grafikleri takip et.
6. **Yönet:** [Operasyon Rehberi](ceph-operations-guide.md) ile günlük işleri öğren.

---

## 🚀 1. Başlangıç (Start Here)

Ceph'i anlamak ve kurmak için ilk adımlar.

* **[📘 Ceph Kavramsal Rehberi](ceph-concepts-guide.md)** (`ceph-concepts-guide.md`)
  * *İçerik:* Mimari, RADOS, CRUSH, Bileşenler (MON, OSD, MGR), Veri akışı.
  * *Hedef Kitle:* Herkes, Mimarlar.

* **[🛠️ Ceph Kurulum Rehberi](ceph-installation-guide.md)** (`ceph-installation-guide.md`)
  * *İçerik:* Donanım gereksinimleri, `cephadm` ile kurulum, Node ekleme, Servis dağıtımı.
  * *Hedef Kitle:* Sistem Yöneticileri.

---

## ⚙️ 2. Yapılandırma ve Yönetim (Configuration)

Kurulum sonrası cluster'ı şekillendirme.

* **[🎱 Pool Yönetimi Rehberi](ceph-pool-guide.md)** (`ceph-pool-guide.md`)
  * *İçerik:* Replicated vs EC pool'lar, PG hesaplama, Quota, Snapshot.
  * *Hedef Kitle:* Depolama Yöneticileri.

* **[🗺️ CRUSH Map Rehberi](ceph-crush-guide.md)** (`ceph-crush-guide.md`)
  * *İçerik:* Veri dağılımı, Failure Domain (Rack/Host), Device Classes (SSD/HDD), Custom kurallar.
  * *Hedef Kitle:* İleri Seviye Yöneticiler.

* **[🔐 Güvenlik Rehberi](ceph-security-guide.md)** (`ceph-security-guide.md`)
  * *İçerik:* Cephx auth, Kullanıcı/Keyring yönetimi, SSL/TLS, Şifreleme (Encryption).
  * *Hedef Kitle:* Güvenlik Mühendisleri.

---

## 💻 3. İstemci ve Entegrasyon (Usage)

Ceph depolama alanını kullanma.

* **[🚀 İstemci Kullanım Rehberi](ceph-client-guide.md)** (`ceph-client-guide.md`)
  * *İçerik:* RBD (Block), CephFS (File), S3 (Object), iSCSI, Snapshot/Clone, Mirroring.
  * *Hedef Kitle:* Kullanıcılar, DevOps.

* **[☸️ Kubernetes Entegrasyon Rehberi](ceph-kubernetes-guide.md)** (`ceph-kubernetes-guide.md`)
  * *İçerik:* Rook-Ceph operatörü, CSI Driver, StorageClass, PVC yönetimi.
  * *Hedef Kitle:* Kubernetes Yöneticileri, DevOps.

---

## 🛡️ 4. Operasyon ve Bakım (Operations)

Günlük işletim ve bakım süreçleri.

* **[🚑 Operasyon Rehberi](ceph-operations-guide.md)** (`ceph-operations-guide.md`)
  * *İçerik:* Günlük kontroller, Disk değişimi, OSD ekleme/çıkarma, Sunucu bakımı.
  * *Hedef Kitle:* L1/L2 Operasyon Ekipleri.

* **[📊 İzleme (Monitoring) Rehberi](ceph-monitoring-guide.md)** (`ceph-monitoring-guide.md`)
  * *İçerik:* Prometheus, Grafana, Alertmanager kuralları, Log yönetimi.
  * *Hedef Kitle:* Operasyon, NOC Ekipleri.

* **[⬆️ Upgrade Rehberi](ceph-upgrade-guide.md)** (`ceph-upgrade-guide.md`)
  * *İçerik:* Sürüm yükseltme prosedürleri, Rolling upgrade, Legacy uyumluluk.
  * *Hedef Kitle:* Kıdemli Sistem Yöneticileri.

---

## 🆘 5. Acil Durum (Troubleshooting)

İşler ters gittiğinde başvurulacak kaynak.

* **[🔧 Sorun Giderme Rehberi](ceph-troubleshooting-guide.md)** (`ceph-troubleshooting-guide.md`)
  * *İçerik:* HEALTH_WARN/ERR analizi, OSD crash, PG recovery, Performance sorunları.
  * *Hedef Kitle:* Kıdemli Sistem Yöneticileri, Destek Ekipleri.

---

## 📈 İstatistikler

* **Toplam Doküman:** 11 Adet
* **Kapsam:** Uçtan uca Enterprise Ceph Yönetimi
* **Versiyon:** Ceph Reef (v18.2.x) uyumlu

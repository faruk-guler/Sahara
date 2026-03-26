# Modül 10: SCALE Mimarisi: Sanallaştırma (VM) ve Konteyner Ekosistemi

TrueNAS CORE'dan SCALE sürümüne (Linux tabanı) geçişin tek bir ana sebebi vardı: Dünyanın Konteynerlere (Docker/K8s) ve hipervizörlere (KVM) doğru evrilmesi. 2026 itibarıyla SCALE sadece bir "Depo" değil, Hyper-Converged Infrastructure (HCI - Bütünleşik Mimari) üssüdür.

## 1. Sanal Makineler (Virtual Machines - KVM) Mimarisi

Artık harici (pahalı) bir ESXi veya vSphere donanımı almanıza gerek kalmadan Windows, Ubuntu gibi işletim sistemlerini doğrudan TrueNAS üzerinde koşturabilirsiniz.

- İçerisinde dünyanın en köklü Linux sanallaştırma modülü olan **KVM (Kernel-based Virtual Machine)** bulunur.
- VM oluştururken `Storage` sekmesinde bahsi geçen `Zvol` leri (Blok Aygıtları) Harddisk olarak sisteme doğrudan "VirtIO" protokolüyle bağlar.
- Performans, (VirtIO disk ve VirtIO ağ adaptörleri kullanıldığında) donanıma (Bare-Metal) en yakın seviyededir.

### 1.1 GPU Passthrough (Ekran Kartı İletimi)

Mimarinin en büyük kırılımı: TrueNAS sunucunuza taktığınız bir Nvidia Tesla veya RTX kartını (PCIe donanımını) **İzole (Isolated/Passthrough)** ayarıyla, TrueNAS'ı hiç bulaştırmadan doğrudan içerdeki Sanal Makinenin midesine bağlayabilmektir. İçerideki Windows VM, fiziksel bir ekran kartı donanımına sahipmiş gibi tam güç (AI, Deep Learning, Video Render) çalışır.

## 2. Apps (Uygulamalar) ve Konteyner Devrimi (K3s vs Docker)

Uygulama dünyası (Apps) ZFS havuzunuzun içinde yerel (Native) hizmetlerin (Plex, Nextcloud, GitLab, PostgreSQL) 2 tıkla kurulmasını sağlar. Ancak yıllar içinde derin bir mimari savaş verilmiştir.

### 2.1 K3s (Kubernetes) Döneminin Sonu

TrueNAS SCALE ilk devrimini (Cobija/Bluefin) bir Kubernetes türevi olan **K3s** ile yaptı. Ancak K3s, küçük ev ve orta işletme yapısı için fazla hantal (Overhead/RAM yiyen) bir canavardı. Ayrıca Helm Chart mimarisi üzerinden işleyen sistem karmaşıktı (TrueCharts krizi).

### 2.2 Docker/Compose Devrimi (Dragonfish / Electric Eel Sürümleri)

2026 yılında TrueNAS köklerine ve hafifliğe (Native Docker) geri dönmüştür!
Artık Apps sekmesine bastığınızda arka planda hantal kube-letler çalışmaz, doğrudan yalın ve acımasız performansıyla **Docker Engine** (ve Compose) devreye girer.

**Depolama Entegrasyonu (Persistent Volumes - PVC):**

Docker'daki en büyük sorun verinin kalıcı olmamasıdır (Konteyner ölünce veri silinir).
TrueNAS bunu 2 muazzam hamleyle çözer:

- **ix-Volume (HostPath):** TrueNAS, uygulamanın çalışacağı dizini doğrudan bir ZFS Dataset'e bağlar. Örneğin NextCloud uygulamasının "Data" kısmı ZFS içindeki `Tank/Nextcloud_Data` dizinine bağlanır. Uygulama silinse, çökse hatta baştan kurulsa bile, ZFS o datanın düzenli Snapshot'unu aldığı için Veri güvenli kalır.

### 2.3 Custom (Kendi Uygulamanı Kur) Ortamı

Sadece mağazada (Katalogda) sunulan uygulamalara mahkum değilsiniz. `Custom App` (veya Docker Compose) butonu ile Dockerhub üzerindeki herhangi bir `nginx:latest` konteynerini, Port yönlendirmelerini ve ZFS depolama yollarını (`/mnt/tank/...`) belirterek anında yayına alabilirsiniz. (TrueNAS'in iç beynine SSH atıp `docker run` manuel komutuyla uygulama ayağa kaldırmak desteklenmez ve ZFS API'lerini bozar; mutlaka arayüz (UI) kullanılmalıdır).

---
[Önceki Modül: Modül 09](./09_Snapshot_Replikasyon.md) | [Sonraki Modül: Modül 11 - Performans Tuning ve Darboğaz Analizi](./11_Performans_Tuning.md)

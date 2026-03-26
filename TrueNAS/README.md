# TrueNAS Master Series: 2026 Edition

![TrueNAS Logo](https://www.storagereview.com/wp-content/uploads/2020/06/StorageReivew-iXsystems-TrueNAS-CORE-intro.png)

## 🗄️ TrueNAS Nedir?

TrueNAS (Eski adıyla FreeNAS), **ZFS (OpenZFS)** dosya sisteminin muazzam gücünü kullanan, Software-Defined Storage (Yazılım Tanımlı Depolama) devriminin lideridir. Kurumsal mimari standartı olan "TrueNAS SCALE" sürümü (Debian Linux tabanlı), sadece devasa bir Disk/Dosya sunucusu olmakla kalmaz; aynı zamanda KVM Sanallaştırma ve Docker/Konteyner uygulamalarını barındıran Hyper-Converged (Bütünleşik) bir Veri Merkezi merkez üssüdür.

ZFS sayesinde donanımsal RAID kartları (MegaRAID vb.) tarihe karışmış, verinin her bir biti Sessiz Veri Çürümesine (Bit Rot) karşı korumaya alınmış ve %100 oranında (Copy-on-write sayesinde) Ransomware (Fidye) şifrelemelerine karşı 1 saniyelik Snapshot'larla ölümsüzleştirilmiştir.

## 📚 Modüller (Eğitim İçeriği)

Sistemi Donanım seçiminden alıp, Mimar olarak DevSecOps seviyesine kadar getiren 12 Adımlık Master eğitim serisi aşağıdadır:

| Mimar Modülleri | Optimizasyon ve Uygulama Modülleri |
| :--- | :--- |
| 👉 **[Modül 01: TrueNAS Mimarisini Anlamak ve CORE vs SCALE](./01_Giris_ve_Mimari.md)** | 👉 **[Modül 07: SMB, NFS ve iSCSI Yönetimi](./07_SMB_NFS_iSCSI_Paylasim.md)** |
| 👉 **[Modül 02: Donanım Anatomisi ve Strict Kurallar](./02_Donanim_Gereksinimleri.md)** | 👉 **[Modül 08: Yetkilendirme Standartları ve ACL/AD](./08_ACL_ActiveDirectory.md)** |
| 👉 **[Modül 03: Kurulum ve Ağ (Network) Temelleri](./03_Kurulum_Network.md)** | 👉 **[Modül 09: ZFS Snapshots ve Replikasyon](./09_Snapshot_Replikasyon.md)** |
| 👉 **[Modül 04: ZFS Mastery (VDEV, Pools) ve RAID](./04_ZFS_Mastery.md)** | 👉 **[Modül 10: SCALE Mimarisi: Sanallaştırma ve Konteynerler](./10_SCALE_Apps_VMs.md)** |
| 👉 **[Modül 05: Dataset ve Zvol Yönetimi](./05_Dataset_Zvol_Islemleri.md)** | 👉 **[Modül 11: Performans Tuning ve Darboğaz Analizi](./11_Performans_Tuning.md)** |
| 👉 **[Modül 06: Gelişmiş Ağ (Network) Yapılandırması](./06_Network_VLAN_LACP.md)** | 👉 **[Modül 12: Güvenlik, Hardening ve Şifreleme](./12_Guvenlik_ve_Hardening.md)** |

---

**Hızlı İpuçları:** Günlük Komutlar ve sorun giderme için [Mühendislik CheatSheet](./CheatSheet.md)'i inceleyebilirsiniz.

### Hazırlayan: Mimar Düzeyi 2026 Standartları

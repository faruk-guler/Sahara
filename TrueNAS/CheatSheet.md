# TrueNAS & ZFS Master Cheat Sheet (2026)

Bu doküman, TrueNAS arayüzüne ulaşamadığınızda veya SSH üzerinden derinlemesine performans/darboğaz analizi yaparken kullanacağınız hayat kurtarıcı SSH Komutlarını barındırır.

## 🗄️ ZPool (Havuz) Komutları

```bash
# Tüm Havuzların sağlık ve disk durumunu ekrana basar
zpool status -v

# Diskte Sessiz Veri Hatası/Çürümesi aramayı (Scrub) manuel başlatır
zpool scrub Tank

# Havuzların kapasite ve Fragmantasyon (Dağılma) yüzdesini gösterir
zpool list

# Mükemmel Performans Monitörü (Her diskin ve vdev'in I/O okumasını canlı izler)
zpool iostat -v 1
```

## 📁 ZFS (Dataset ve Zvol) Komutları

```bash
# Sistemdeki bütün dosya sistemleri ve özelliklerini listeler
zfs list

# Bir datasetin Compression (Sıkıştırma) oranını gösterir
zfs get compressratio Tank/Muhasebe

# Dataset üzerindeki tehlikeli DEDUPLICATION (Tekilleştirme) durumunu kapatır
zfs set dedup=off Tank/Yedekler

# Senkronizasyon (Virtual Machine/iSCSI) darboğazını atlamak için (Kritik RİSKLİ)
zfs set sync=disabled Tank/VM_Diskleri
```

## 🚀 RAM / ARC ve Performans Profilleme

```bash
# ARC (RAM Önbelleği) istatistiklerini (Örn: Hit, Miss Oranları) gösterir
arcstat

# Sistemdeki tüm önbelleği (Level-2 L2ARC dahil) özetler
arc_summary

# Ağ kartının MTU'sunu veya hız darboğazını (Kablo kalitesini) terminalden ölçmek
iperf3 -s # (Server tarafında çalıştırılır)
```

## 🔓 Güvenlik ve Ağ Temizliği

```bash
# TrueNAS Middleware'in (Arayüz servisi) çökmesi durumunda yeniden başlatma
systemctl restart middlewared

# Active Directory bağlantısında yaşanan önbellek tutarsızlıklarında
systemctl restart winbindd
```

---
[Ana Navigasyon (README)'ye Dön](./README.md)

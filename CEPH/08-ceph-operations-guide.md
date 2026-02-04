# 🚑 Ceph Operasyon ve Sorun Giderme Rehberi (Day 2 Operations)

Bu doküman, çalışan bir Ceph kümesinde **"İşler ters gittiğinde"** veya **"Bakım yapmam gerektiğinde"** başvuracağın reçetedir. Ceph'in kendi kendini iyileştirme yeteneği vardır ama bazen insan müdahalesi şarttır.

---

## 🏥 1. Sağlık Kontrolü (Health Check)

Sisteme girdiğinde her sabah bakman gereken ilk komut:

```bash
ceph -s
```

### Çıktı Analizi

* **HEALTH_OK:** Her şey yolunda. Arkanıza yaslanın.
* **HEALTH_WARN:** Bir şeyler ters gidiyor ama sistem hala çalışıyor. (Örn: Disk doldu, 1 OSD düştü, PG sayısı dengesiz).
* **HEALTH_ERR:** Ciddi sorun. Veri erişimi durmuş veya veri kaybı riski olabilir.

> Detaylı hatayı görmek için:
>
> ```bash
> ceph health detail
> ```

---

## 💀 2. Bozulan Bir Diski (OSD) Değiştirmek

Ceph'te en sık yaşayacağın donanım arızası disk bozulmasıdır.
Diyelim ki `osd.5` bozuldu (DOWN durumda).

### Adım 1: OSD'yi Kaldırmak (Cephadm Yöntemi)

Önce Ceph'e bu diskin artık ölü olduğunu ve onu unutması gerektiğini söylemeliyiz.

```bash
# OSD numarasını (ID) doğrula
ceph osd tree

# OSD'yi orchestrator ile kaldır (bu komut out, stop ve purge işlemlerini otomatik yapar)
ceph orch osd rm 5

# İşlemin durumunu izle
ceph orch osd rm status
```

### Adım 2: OSD Kaldırma İşlemini Bekle

Ceph, veriyi güvenli şekilde diğer OSD'lere kopyalar (rebalance). Bu işlem disk boyutuna göre dakikalar veya saatler sürebilir.

```bash
# Cluster durumunu izle - PG'ler active+clean olmalı
ceph -s
```

### Adım 3: Fiziksel Değişim ve Yeni Disk Ekleme

1. Bozuk diski sunucudan sök.
2. Yeni diski tak.
3. Eğer disk daha önce kullanılmışsa temizle:

   ```bash
   wipefs -a /dev/sdX
   # İnatçı LVM kalıntıları için:
   ceph-volume lvm zap /dev/sdX --destroy
   ```

4. Yeni diski Ceph'e ekle:

   **Yöntem 1 (Otomatik - Önerilen):**

   ```bash
   # Orchestrator tüm boş diskleri otomatik bulur ve ekler
   ceph orch apply osd --all-available-devices
   ```

   **Yöntem 2 (Manuel - Belirli disk için):**

   ```bash
   ceph orch daemon add osd node2:/dev/sdX
   ```

---

## 🔄 3. Sunucu Bakımı ve Yeniden Başlatma (Reboot)

Bir sunucuya RAM takacaksın veya Kernel güncellemesi yaptın, `reboot` atman lazım.
**DİKKAT:** Eğer direkt reboot atarsan, Ceph o sunucudaki OSD'leri "DOWN" görür ve paniğe kapılıp verileri diğer sunuculara kopyalamaya başlar (Rebalance). Gereksiz yere ağı kilitlersin.

### Doğru Yöntem (Maintenance Mode)

**1. Rebalance'ı Durdur:**
Ceph'e "Sakin ol, bu sunucu geri gelecek, verileri kopyalamaya başlama" diyoruz.

```bash
ceph osd set noout
```

**2. Sunucuyu Yeniden Başlat:**
Şimdi sunucuya bakım yap veya reboot at. Ceph, OSD'ler down olsa bile panik yapmaz.

**3. Normale Dön:**
Sunucu açıldıktan ve OSD'ler `UP` olduktan sonra kilidi kaldır.

```bash
ceph osd unset noout
```

---

## 🛠️ 4. Servis Yönetimi

`cephadm` kullandığımız için servisler aslında birer konteynerdir ve `systemd` ile yönetilir.

```bash
# Tüm servisleri listele
ceph orch ps

# Belirli bir servisi yeniden başlat (Örn: MGR sapıttıysa)
ceph orch restart mgr

# Belirli bir OSD'yi loglarıyla izle
cephadm logs --name osd.2
```

---

## 🚨 5. Acil Durum Notları (Cheat Sheet)

### Disk Doluluğu Uyarısı (Near Full)

Ceph disklerin %85'i dolunca **WARNING**, %95'i dolunca **READ-ONLY** moduna geçer.

* **Çözüm:** Ya hemen eski veri sil ya da acilen yeni disk ekle.

### Clock Skew (Saat Kayması)

Monitörler arası saat farkı olursa cluster durur.

* **Çözüm:** `chronyc sources` ile NTP'yi kontrol et. `systemctl restart chrony` ile düzelt.

### Olay Geçmişini Görmek

"Dün gece sistemde ne oldu?" sorusunun cevabı:

```bash
ceph log last 100
```

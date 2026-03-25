# Modül 16: Syscheck ve Syscollector Detayları

Wazuh Agent'ın arka planındaki iki ana motor olan Syscheck ve Syscollector'ın derinliklerine inelim.

## 1. Syscheck (FIM Motoru)

Syscheck dosya sistemini tarayan ve değişiklikleri raporlayan kısımdır. İlk taramada bir "Baseline" (referans) oluşturur, sonraki taramalarda farkları (diff) raporlar.

### DB Depolama

Syscheck verileri Manager tarafında çekirdek bir SQLite veritabanında (`/var/ossec/queue/db/`) saklanır.

### Optimizasyon İpuçları

- **Max Files:** Büyük dizinlerde performansı artırmak için dosya limiti konulmalıdır.
- **Excludes:** Sık değişen log dosyaları `.log`, `.tmp` mutlaka (`ignore`) olarak eklenmelidir.

## 2. Syscollector (Sistem Envanteri)

Syscollector, uç noktadaki donanım, yazılım ve ağ bilgilerini toplar.

### Toplanan Veriler

- Yüklü paketler (Versiyon, vendor).
- İşletim sistemi detayları.
- Donanım (CPU, RAM, Disk).
- Ağ arayüzleri, IP adresleri, açık portlar.

### Konfigürasyon

```xml
<syscollector>
  <disabled>no</disabled>
  <interval>1h</interval>
  <scan_on_start>yes</scan_on_start>
  <hardware>yes</hardware>
  <os>yes</os>
  <network>yes</network>
  <packages>yes</packages>
  <ports all="no">yes</ports>
  <processes>yes</processes>
</syscollector>
```

## Neden Önemli?

Syscollector verileri, zafiyet tespitinin (Module 08) temel kaynağıdır. Eğer Syscollector düzgün çalışmazsa zafiyet taraması yapılamaz.

---

[README'ye Dön](README.md)

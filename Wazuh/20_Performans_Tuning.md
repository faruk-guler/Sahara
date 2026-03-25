# Modül 20: Performans İyileştirme ve Tuning

Manager üzerindeki yükü azaltmak ve saniyedeki olay (EPS) sayısını artırmak için yapılan optimizasyonlardır.

## 1. OSSEC.CONF Tuning

- **Analysisd:** Log işleyen iş parçacığı (threads) sayısını artırın.
- **Remoted:** Ağ üzerinden gelen bağlantı kapasitesini optimize edin.

```xml
<analysisd>
  <thread_stack_size>1024</thread_stack_size>
  <queue_size>131072</queue_size> <!-- Olay yoğunluğuna göre artırılmalı -->
</analysisd>
```

## 2. Ağ ve Bağlantı (Remoted)

Ajan trafiğini yöneten `remoted` servisi için bağlantı limitlerini artırmak performansı doğrudan etkiler.

```xml
<remote>
  <connection_threads>8</connection_threads>
  <queue_size>131072</queue_size>
</remote>
```

- **Heap Size:** RAM miktarının yarısını (max 32GB) tahsis edin.
- **Refresh Interval:** Verilerin diske yazılma aralığını artırın (Örn: 30s).

## 3. Agent Tuning

- **Events Per Second:** Ajanın Manager'a gönderebileceği saniyelik log limitidir.
- **Buffer:** Bant genişliği kısıtlı ortamlarda `buffer` ayarları ile veri kaybı engellenir.

## 4. Log Rotasyonu (Rotation)

Manager üzerindeki `/var/ossec/logs/` dizinindeki ham logların ve alarmlerin disk dolmaması için periyodik olarak silinmesi veya taşınması sağlanmalıdır.

---

[README'ye Dön](README.md)

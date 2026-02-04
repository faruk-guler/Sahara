# 📊 Ceph Monitoring ve Alerting Rehberi

Bu doküman, Ceph cluster'ının izlenmesi, Prometheus/Grafana entegrasyonu ve alarm yapılandırmasını kapsar.

---

## 1. Ceph Dashboard

### Dashboard Durumunu Kontrol Etme

```bash
# Dashboard modülü aktif mi?
ceph mgr module ls | grep dashboard

# Dashboard URL'sini öğren
ceph mgr services

# Dashboard erişim bilgileri
ceph dashboard get-username
ceph dashboard ac-user-show admin
```

### Şifre Sıfırlama

```bash
# Yeni şifre oluştur
echo "YeniSifre123!" > /tmp/pwd.txt
ceph dashboard ac-user-set-password admin -i /tmp/pwd.txt
rm /tmp/pwd.txt
```

### Dashboard SSL Yapılandırması

```bash
# Self-signed sertifika ile
ceph dashboard set-ssl-certificate -i /etc/ceph/dashboard.crt
ceph dashboard set-ssl-certificate-key -i /etc/ceph/dashboard.key
```

---

## 2. MGR Modülleri

### Aktif Modülleri Görme

```bash
ceph mgr module ls
```

### Kritik Modüller

| Modül | Görev | Varsayılan |
|-------|-------|------------|
| `dashboard` | Web UI | Aktif |
| `prometheus` | Metrik ihracı | Pasif |
| `balancer` | Veri dengeleme | Aktif |
| `pg_autoscaler` | PG otomatik ayarlama | Aktif |
| `telemetry` | Anonim kullanım verisi | Pasif |

### Modül Etkinleştirme

```bash
# Prometheus modülünü aç
ceph mgr module enable prometheus

# Balancer modülünü yapılandır
ceph balancer mode upmap
ceph balancer on
```

---

## 3. Prometheus Entegrasyonu

### Prometheus Modülünü Etkinleştirme

```bash
ceph mgr module enable prometheus
```

### Prometheus Endpoint

Varsayılan olarak MGR sunucusunda `http://<mgr-ip>:9283/metrics` adresinden metriklere erişilir.

```bash
# Endpoint'i test et
curl http://192.168.1.10:9283/metrics | head -50
```

### Prometheus Yapılandırması

`/etc/prometheus/prometheus.yml` dosyasına ekle:

```yaml
scrape_configs:
  - job_name: 'ceph'
    static_configs:
      - targets: ['192.168.1.10:9283']
    honor_labels: true
```

### Node Exporter (Her Ceph Node'da)

```bash
# Node exporter kurulumu
apt install prometheus-node-exporter

# Prometheus'a ekle
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['192.168.1.10:9100', '192.168.1.11:9100', '192.168.1.12:9100']
```

---

## 4. Grafana Entegrasyonu

### Grafana Kurulumu

```bash
apt install grafana
systemctl enable grafana-server
systemctl start grafana-server
```

### Ceph Dashboard'ları İçe Aktarma

Grafana.com'dan hazır Ceph dashboard'ları:

| Dashboard ID | İsim | Açıklama |
|--------------|------|----------|
| 2842 | Ceph Cluster | Genel cluster durumu |
| 5336 | Ceph OSD | OSD detayları |
| 5342 | Ceph Pools | Pool istatistikleri |
| 7056 | Ceph RGW | Object storage metrikleri |

**İçe Aktarma:**

1. Grafana → Dashboards → Import
2. Dashboard ID gir (örn: 2842)
3. Prometheus data source seç

---

## 5. Önemli Metrikler

### Cluster Sağlığı

| Metrik | Açıklama | Alarm Eşiği |
|--------|----------|-------------|
| `ceph_health_status` | 0=OK, 1=WARN, 2=ERR | >0 |
| `ceph_osd_up` | OSD çalışıyor mu | =0 |
| `ceph_osd_in` | OSD cluster'da mı | =0 |
| `ceph_mon_quorum_status` | MON quorum durumu | <1 |

### Kapasite

| Metrik | Açıklama | Alarm Eşiği |
|--------|----------|-------------|
| `ceph_cluster_total_used_bytes` | Kullanılan alan | >%85 |
| `ceph_pool_stored_raw` | Pool ham kullanım | - |
| `ceph_osd_stat_bytes_used` | OSD kullanımı | >%90 |

### Performans

| Metrik | Açıklama |
|--------|----------|
| `ceph_osd_op_r_latency_sum` | Okuma gecikmesi |
| `ceph_osd_op_w_latency_sum` | Yazma gecikmesi |
| `ceph_pool_rd` | Pool okuma sayısı |
| `ceph_pool_wr` | Pool yazma sayısı |

### PG Durumu

| Metrik | Açıklama | Alarm Eşiği |
|--------|----------|-------------|
| `ceph_pg_degraded` | Bozulmuş PG sayısı | >0 |
| `ceph_pg_undersized` | Eksik replikasyon | >0 (uzun süre) |
| `ceph_pg_stale` | Aktif olmayan PG | >0 |

---

## 6. Alertmanager Kuralları

### Prometheus Alert Rules

`/etc/prometheus/rules/ceph.yml`:

```yaml
groups:
  - name: ceph-alerts
    rules:
      - alert: CephHealthWarning
        expr: ceph_health_status == 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Ceph cluster is in WARNING state"
          description: "Ceph cluster health is WARN for more than 5 minutes"

      - alert: CephHealthError
        expr: ceph_health_status == 2
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Ceph cluster is in ERROR state"
          description: "Ceph cluster health is ERR - immediate action required"

      - alert: CephOSDDown
        expr: ceph_osd_up == 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "OSD {{ $labels.ceph_daemon }} is down"
          description: "OSD has been down for more than 5 minutes"

      - alert: CephDiskNearFull
        expr: (ceph_osd_stat_bytes_used / ceph_osd_stat_bytes) * 100 > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "OSD {{ $labels.ceph_daemon }} is filling up"
          description: "OSD disk usage is above 85%"

      - alert: CephDiskCritical
        expr: (ceph_osd_stat_bytes_used / ceph_osd_stat_bytes) * 100 > 95
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "OSD {{ $labels.ceph_daemon }} is almost full"
          description: "OSD disk usage is above 95% - cluster will become read-only!"

      - alert: CephPGDegraded
        expr: ceph_pg_degraded > 0
        for: 30m
        labels:
          severity: warning
        annotations:
          summary: "Ceph has degraded PGs"
          description: "{{ $value }} PGs are in degraded state for >30 minutes"

      - alert: CephMonQuorumLost
        expr: count(ceph_mon_quorum_status == 1) < 2
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Ceph monitor quorum at risk"
          description: "Less than 2 monitors in quorum"

      - alert: CephSlowOps
        expr: ceph_healthcheck_slow_ops > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Ceph has slow operations"
          description: "{{ $value }} slow ops detected"
```

---

## 7. Log Yönetimi

### Ceph Log Konumları

| Bileşen | Log Dosyası |
|---------|-------------|
| MON | `/var/log/ceph/ceph-mon.*.log` |
| OSD | `/var/log/ceph/ceph-osd.*.log` |
| MGR | `/var/log/ceph/ceph-mgr.*.log` |
| MDS | `/var/log/ceph/ceph-mds.*.log` |
| RGW | `/var/log/ceph/ceph-client.rgw.*.log` |

### Cephadm ile Log Görüntüleme

```bash
# Belirli daemon logları
cephadm logs --name osd.2

# Son 100 satır
cephadm logs --name mon.node1 -- --tail 100

# Canlı takip
cephadm logs --name mgr.node1.abc123 -- -f
```

### Log Seviyesi Ayarlama

```bash
# Geçici olarak artır (runtime)
ceph tell osd.* injectargs '--debug-osd 10'

# Kalıcı ayar
ceph config set osd debug_osd 5/10  # günlük/bellek

# Varsayılana dön
ceph config set osd debug_osd 1/5
```

### Log Rotation

Ceph varsayılan olarak logrotate kullanır. `/etc/logrotate.d/ceph` dosyasını kontrol edin.

---

## 8. Cluster Durumu Komutları

### Hızlı Durum Kontrolü

```bash
# Genel durum
ceph -s

# Detaylı sağlık
ceph health detail

# OSD ağacı
ceph osd tree

# PG durumu
ceph pg stat

# IO performansı
ceph osd perf
```

### Canlı İzleme

```bash
# 1 saniyede bir güncelle
watch -n 1 ceph -s

# Ceph watch (gerçek zamanlı olaylar)
ceph -w
```

---

## 9. Performance Monitoring

### Dahili Performans Sayaçları

```bash
# OSD performans istatistikleri
ceph osd perf

# Pool IO istatistikleri
ceph osd pool stats

# RBD performansı
rbd perf image iotop
```

### Slow Ops İzleme

```bash
# Yavaş operasyonları görüntüle
ceph daemon osd.0 dump_ops_in_flight

# Blocked requests
ceph daemon osd.0 dump_blocked_ops
```

---

## 10. Monitoring Best Practices

### ✅ Yapılması Gerekenler

- Prometheus + Grafana kurulumu yapın
- Kritik alarmlar için Alertmanager yapılandırın
- Her OSD için disk kullanım alarmı ekleyin
- MON quorum alarmı mutlaka olsun
- Log rotation'ı kontrol edin

### ❌ Yapılmaması Gerekenler

- Sadece Dashboard'a güvenmeyin (erişilemez olabilir)
- Debug log seviyesini uzun süre yüksek tutmayın (disk dolar)
- Alarm eşiklerini çok düşük tutmayın (alarm yorgunluğu)
- Node exporter'ı unutmayın (OS metrikleri de önemli)

---

## 11. Monitoring Checklist

```
[ ] Prometheus modülü aktif mi?
[ ] Prometheus Ceph endpoint'i scrape ediyor mu?
[ ] Grafana dashboard'ları yüklü mü?
[ ] Health status alarmı tanımlı mı?
[ ] Disk doluluk alarmı tanımlı mı?
[ ] OSD down alarmı tanımlı mı?
[ ] MON quorum alarmı tanımlı mı?
[ ] Log rotation çalışıyor mu?
[ ] Alertmanager e-posta/Slack bildirimi var mı?
```

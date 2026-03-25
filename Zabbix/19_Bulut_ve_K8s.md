# Modül 19: Cloud-Native, Kubernetes (K8s) ve OpenTelemetry İncelemesi

Günümüz IT yapısında sunuculara ajan yüklemek giderek azalıyor. Ephemeral (geçici), saniyeler içinde doğup ölen Pod'lar ve Cloud Database'leri (AWS RDS, DynamoDB) doğrudan Zabbix ile izlenebilmelidir.

## 1. Cloud İzleme Yöntemi (AWS, Azure, GCP API)

Amazon Web Services (AWS) üzerindeki EC2'lara Ajan yükleyebilirsiniz ancak bir RDS (Yönetilen Veritabanı) servisine ajan atamazsınız.

Bunun için Zabbix, **HTTP Agent** mantığıyla (Modül 08) Cloudwar (CloudWatch) API'lerini doğrudan "Pull" (Çeker) mantığıyla sömürür.

- Zabbix sunucusu AWS ortamındaki CloudWatch API'sine kimlik bilgilerini (IAM Access Key / Secret Key) atar (Vault destekli veya Macros).
- LLD (Otomatik Discovery) sayesinde yeni açılan veya silinen EC2, RDS, ELB (Load Balancer) cihazları Zabbix Arayüzünde otomatik yansır ve silinir.
- Cloudwatch'a giden istek maliyet (Cost) yazabileceği için sorgu süresi Zabbix tarafında 5 dakikada 1'e çıkartılmalıdır.

## 2. Kubernetes İzlemenin Master Yolu: Zabbix Operator

Geçmişte Zabbix, K8s için bir "Helm Chart" verirdi. İçinden ufak ajanlar (Daemonset) çıkıp her Node'a yapışırdı. Bu eski yöntemdir.

2026 yılı Kubernetes dünyasında izleme **Kube-State-Metrics**, **Prometheus Endpointleri** ve **Zabbix Custom Resource Definitions (CRDs)** ile yapılır.

Bunun için kurumsal çözüm `Zabbix Operator` kullanmaktır.
Operator; K8s içine bir "Hakem" olarak kurulur ve K8s API'sini doğrudan dinler.
Siz Zabbix tarafında elle hiçbir şey yapmazsınız. K8s tarafında bir Namespace veya Deployment Manifesti (YAML) içine şu küçük bloğu eklersiniz:

```yaml
# K8s Deployment Manifest
metadata:
  annotations:
    zabbix.com/monitor: "true"
    zabbix.com/template: "Template App Nginx HTTP"
```

Bu kelimeyi Zabbix Operator arkaplanda "Hımm, Zabbix'i açayım (API üzerinden), Zabbix'in içine O Pod'un adında bir cihaz ekleyeyim, ona da Nginx Template'i bağlayayım" diyerek 0 insan müdahalesi ile K8s Mimarisi ayağa kaldırılır.

## 3. OpenTelemetry (OTel): Büyük Sıçrama (Sinyal Birleşimi)

2026 Observability standartı olan OTel kavramı; log, metrik ve trace (üçlü sütün) verilerinin Cihaz Bağımsız tek bir evrensel dilde toplanmasını amaçlar. Promtheus Metrikleri, Jaeger traceleri... Hepsi bu OTel sinyal havuzuna dökülür.

Zabbix Server / Proxy (7.0+ sürümü itibarıyla) artık yerleşik bir **OTLP Receiver (Alıcı)** pozisyonundadır.

- `zabbix_server.conf` içine `StartLLDProcessors=5` gibi yeni nesil algılayıcıları açtınız.
- Veri almak için Server TCP Portu: **4317** (gRPC) veya **4318** (HTTP) aktif edilir.
- Artık ne ajan yüklemeye de ihtiyacınız ne de Prometheus kurmaya var. Uygulama geliştiricisi backend kodunun (Golang/Node/Java) içinden veriyi (Trace ve Metrics) direkt TCP 4317 Zabbix sunucusuna fırlatır!
- Zabbix bu dev veri bloğunu `Preprocessing` (Modül 08) ekranındaki **Prometheus pattern** ve **JSONPath** özellikleri eşliğinde ayrıştırarak Zabbix İtem'larına, Alarm (Trigger) nesnelerine dağıtır.

Grafana kullanmadan, Prometheus kurmadan, tek bir Zabbix Merkezi (Native High Availability destekli, TimescaleDB motorlu) tüm modern cloud yığınını (Stack) sindirebilecek güce evrilmiştir.

---
[Önceki Modül](./18_SNMP_Traps.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Güvenlik ve Hardening](./20_Guvenlik_Hardening.md)

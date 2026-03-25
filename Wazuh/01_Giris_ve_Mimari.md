# Modül 01: Wazuh Nedir? Mimari ve Temel Kavramlar

## Wazuh Nedir?

Wazuh; sistemlerin güvenliğini izlemek, tehditleri algılamak ve olaylara yanıt vermek için kullanılan açık kaynaklı bir güvenlik platformudur. Modern SIEM (Güvenlik Bilgisi ve Olay Yönetimi) ve XDR (Genişletilmiş Algılama ve Yanıt) yeteneklerini birleştirir.

## Temel Bileşenler

### 1. Wazuh Indexer

- Verileri depolayan ve arama motoru görevi gören yüksek performanslı bir tam metin arama ve analiz motorudur.
- Veriler JSON formatında saklanır.

### 2. Wazuh Server

- Agent'lardan gelen verileri analiz eden merkezi bileşendir.
- Dekoderlar ve kurallar burada çalışır.
- Tehtit istihbaratı (Threat Intel) verilerini kullanarak olayları zenginleştirir.

### 3. Wazuh Dashboard

- Verilerin görselleştirildiği web arayüzüdür.
- Uyumluluk raporlarını (PCI DSS, HIPAA vb.) buradan takip edebilirsiniz.

### 4. Wazuh Agent

- İzlenen uç noktalara (Linux, Windows, macOS, Solaris vb.) kurulan hafif yazılımdır.
- Log toplama, FIM ve SCA gibi kritik işleri gerçekleştirir.

## Mimari Akış ve Portlar

Wazuh bileşenleri arasındaki iletişim belirli portlar üzerinden gerçekleşir:

- **1514 (TCP/UDP):** Agent'lardan Server'a veri iletimi (Remoted).
- **1515 (TCP):** Agent kayıt (Enrollment) servisi (Authd).
- **55000 (TCP):** Wazuh RESTful API.
- **9200 (TCP):** Indexer API (Server-Indexer iletişimi).

### Arka Plan Servisleri (Daemons)

- **ossec-authd:** Yeni ajanların sisteme kaydedilmesini yönetir.
- **wazuh-modulesd:** Zafiyet tespiti, bulut entegrasyonu ve SCA gibi modülleri çalıştırır.
- **wazuh-analysisd:** Gelen logları analiz eden ana motordur.

---

[README'ye Dön](README.md)

# Modül 17: OSSEC vs Wazuh Farkları

Wazuh, OSSEC'in bir fork'u olarak başlasa da bugün bambaşka bir platform haline gelmiştir.

## Karşılaştırma Tablosu

| Özellik | OSSEC (Standard) | Wazuh |
| :--- | :--- | :--- |
| **Arayüz** | CLI (Web UI kısıtlı) | Wazuh Dashboard (Gelişmiş) |
| **API** | Yok | Gelişmiş RESTful API |
| **FIM** | Klasik Hash Kontrolü | Real-time & Whodata İzleme |
| **Zafiyet Tespiti** | Yok | Entegre Vuln-Detector |
| **Elasticsearch** | Zorunlu değil | Dahili Indexer/Search |
| **SCA** | Kısıtlı | CIS Standartlarında YAML Poliçe |

## Neden Wazuh?

OSSEC hala harika bir HIDS (Host-based IDS) olsa da, modern siber güvenlik ihtiyaçları (Dashboard, API, Bulut entegrasyonu, Zafiyet yönetimi) Wazuh'u vazgeçilmez kılmaktadır.

## Agentless Monitoring

Hem OSSEC hem de Wazuh, ajan kurulamayan (Network devices vb.) sistemleri SSH üzerinden izlemeyi destekler. Wazuh'ta bu işlem `agentless` modülü ile yapılır:

```xml
<agentless>
  <type>ssh_generic_diff</type>
  <frequency>3600</frequency>
  <arguments>ls -la /etc</arguments>
  <host>user@192.168.1.50</host>
</agentless>
```

## Geçiş (Migration)

OSSEC'ten Wazuh'a geçiş yaparken kurallarınız (`local_rules.xml`) büyük oranda uyumludur. Ancak dekoderlarda ufak Regex farkları olabilir.

---

[README'ye Dön](README.md)

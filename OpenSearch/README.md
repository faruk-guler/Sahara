# 🛡️ OpenSearch: Geleceğin AI-Native Arama ve Analiz Platformu

<img src="./img/OpenSearch.jpg" alt="OpenSearch Preview" width="80%">

**OpenSearch**, petabaytlarca veri üzerinde gerçek zamanlı arama, analiz ve izlenebilirlik (observability) sağlayan, Apache 2.0 lisanslı, tamamen açık kaynaklı bir platformdur. 2026 yılı itibarıyla, geleneksel metin araması dünyasını geride bırakarak, tamamen **AI-Native** bir ekosisteme dönüşmüştür.

---

## 🏛️ Mimari Mükemmellik (Black Belt Perspektifi)

Bir OpenSearch uzmanı olarak, sadece API'leri değil, sistemin kalbindeki **dağıtık sistem teorilerini** bilmek sizi diğerlerinden ayırır:

*   **Shard Allocation Deciders:** Kümenizin düğümleri arasında shard'ların nasıl "dans ettiğini" anlamak; disk bazlı, throttling tabanlı ve farkındalık (awareness) bazlı kararların nasıl verildiğini analiz etmek.
*   **Memory Internals:** JVM heap bellek yönetiminin ötesine geçerek; `Fielddata`, `Request Cache` ve `Query Cache` katmanlarının performans darboğazlarını nasıl çözdüğünü bilmek.
*   **High Availability (HA) Design:** 3-Region Active-Active-Passive mimarileri, Quorum hesaplamaları ve Split-Brain senaryolarına karşı bağışıklık kazandırılmış tasarımlar.
*   **Segment Replication (v3+):** Geleneksel belge kopyalamadan, Lucene segment seviyesinde kopyalamaya geçişin getirdiği %40'lık performans devrimini yönetmek.

---

## 🚀 Neden OpenSearch?

Modern veri dünyasında hız, ölçeklenebilirlik ve güvenlik bir seçenek değil, zorunluluktur. OpenSearch, bu üç sütun üzerine inşa edilmiştir:

*   **Sınırsız Ölçeklenebilirlik:** Küçük projelerden devasa veri merkezlerine kadar yatayda sınırsız büyüyebilir.
*   **Tam Güvenlik (Security-First):** Yerleşik TLS/SSL, Rol Tabanlı Erişim Denetimi (RBAC) ve FIPS 140-2 uyumluluğu ile verileriniz her zaman korunur.
*   **Zengin Analitcs:** Sadece arama yapmaz; karmaşık SQL/PPL sorguları ve makine öğrenimi tabanlı anomali tespiti ile verilerinizden anlam çıkarır.

---

## 🧠 2026: AI-Native Devrimi

OpenSearch v3.x serisi, yapay zeka ve agentic iş yüklerini mimarisinin kalbine yerleştirmiştir:

1.  **Vektör Veritabanı Gücü:** Yerleşik **k-NN (k-Nearest Neighbors)** motoru sayesinde, metinlerin anlamca benzerliğini (Semantic Search) milisaniyeler içinde hesaplar.
2.  **RAG (Retrieval-Augmented Generation):** Büyük Dil Modellerine (LLM) en alakalı bağlamı (context) sağlayarak, AI yanıtlarının doğruluğunu zirveye taşır.
3.  **MCP (Model Context Protocol):** AI agent'larının (Claude, Gemini vb.) veritabanınıza güvenli bir şekilde "bağlanıp" araçlarını (tools) kullanmasına olanak tanır.
4.  **Search Pipelines:** Hibrit arama, yeniden sıralama (reranking) ve dil işleme adımlarını sorgu anında otomatikleştirir.

---

## 🏗️ Modern Mimari ve Performans

Arka planda **Apache Lucene 10** koşan OpenSearch, 2026 standartlarına uygun donanım hızlandırma ve mimari teknikleri kullanır:

*   **gRPC & Protobuf:** Düğümler arası iletişimde JSON'un yavaşlığını geride bırakarak ikili (binary) hızına ulaşır.
*   **Segment Replication:** Yazma işlemlerini %40 hızlandıran, replica tarafında CPU maliyetini sıfırlayan modern kopyalama yöntemi.
*   **Reader/Writer Separation:** Okuma ve yazma yüklerini birbirinden ayırarak, sistemin bir tarafı yoğunlaştığında diğer tarafın etkilenmemesini sağlar.

---

## 🛠️ Ekosistem ve Araçlar

OpenSearch sadece bir veritabanı değil, devasa bir araç setidir:

*   **OpenSearch Dashboards:** Verilerinizi görselleştirmek, haritalar oluşturmak ve alarmları yönetmek için zengin web arayüzü.
*   **İzlenebilirlik (Observability):** Loglar, metrikler ve OpenTelemetry tabanlı izler (traces) tek bir çatıda toplanır.
*   **ML Framework:** Kendi modellerinizi (ONNX, TorchScript) doğrudan OpenSearch içine yükleyin veya dış servislerle (OpenAI, SageMaker) bağlayın.

---

## 🏁 Hızlı Başlangıç

Saniyeler içinde denemek için:

```bash
docker run -p 9200:9200 -p 9600:9600 -e "discovery.type=single-node" opensearchproject/opensearch:latest
```

---

---

## 🎓 Uzmanlık Müfredatı ve Öğrenim Yolları

Bu kütüphane, temel bilgilerden uzmanlığa giden 13 ana kategori ve teknik derinlik dosyalarından oluşmaktadır. Kendi öğrenim yolunuzu seçerek derinleşebilirsiniz:

### 🔰 Temel ve Altyapı
*   [01 - Giriş ve Mimari](./docs/01-Giris-ve-Mimari/README.md)
    *   [Mimari Derinlemesine İnceleme](./docs/01-Giris-ve-Mimari/Derinlemesine-Inceleme.md)
*   [02 - Kurulum ve Yapılandırma](./docs/02-Kurulum-ve-Yapi-landirma/README.md)
*   [03 - Düğüm ve Küme Yönetimi](./docs/03-Dugum-ve-Kume-Yonetimi/README.md)
    *   [Küme Yönetimi Derinlemesine İnceleme](./docs/03-Dugum-ve-Kume-Yonetimi/Derinlemesine-Inceleme.md)

### 📊 Veri ve Sorgu Mühendisliği
*   [04 - Veri Girişi ve Dizinleme](./docs/04-Veri-Giris-ve-Dizinleme/README.md)
    *   [Veri Girişi Derinlemesine İnceleme](./docs/04-Veri-Giris-ve-Dizinleme/Derinlemesine-Inceleme.md)
*   [05 - Arama ve Sorgulama](./docs/05-Arama-ve-Sorgulama/README.md)
    *   [Sorgulama Derinlemesine İnceleme](./docs/05-Arama-ve-Sorgulama/Derinlemesine-Inceleme.md)
*   [06 - Vektör ve AI Arama](./docs/06-Vektor-ve-AI-Arama/README.md)
    *   [AI Arama Derinlemesine İnceleme](./docs/06-Vektor-ve-AI-Arama/Derinlemesine-Inceleme.md)

### 🔐 Güvenlik ve Operasyon
*   [07 - Güvenlik ve Erişim Denetimi](./docs/07-Guvenlik-ve-Erisim-Denetimi/README.md)
    *   [Kurumsal Güvenlik Derinlemesine İnceleme](./docs/07-Guvenlik-ve-Erisim-Denetimi/Derinlemesine-Inceleme.md)
*   [08 - İzlenebilirlik ve Analiz](./docs/08-Izlenebilirlik-ve-Analiz/README.md)
    *   [İzlenebilirlik Derinlemesine İnceleme](./docs/08-Izlenebilirlik-ve-Analiz/Derinlemesine-Inceleme.md)
*   [09 - Performans Optimizasyonu](./docs/09-Performans-Optimizasyonu/README.md)
    *   [Bellek ve Performans Derinlemesine İnceleme](./docs/09-Performans-Optimizasyonu/Derinlemesine-Inceleme.md)
*   [10 - Yedekleme ve Kurtarma](./docs/10-Yedekleme-ve-Kurtarma/README.md)
    *   [Yedekleme Derinlemesine İnceleme](./docs/10-Yedekleme-ve-Kurtarma/Derinlemesine-Inceleme.md)

### 🛠️ Dashboards ve Problem Çözme
*   [11 - OpenSearch Dashboards](./docs/11-OpenSearch-Dashboards/README.md)
*   [12 - Sorun Giderme ve SSS](./docs/12-Sorun-Giderme-ve-SSS/README.md)

### 🏆 Black Belt (Mimar Seviyesi)
*   [13 - Uzman Mimar Rehberi (Black Belt Level)](./docs/13-Uzman-Mimar-Rehberi/README.md)

---

> [!NOTE]
> Yukarıdaki tüm detaylar, uzmanlık seviyesinde bir öğrenim için yapılandırılmıştır.

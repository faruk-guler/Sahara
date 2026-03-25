# Wazuh: Kurumsal Düzeyde Açık Kaynaklı Güvenlik Platformu

![Wazuh Logo](./img/logo-wazuh.webp)

**Wazuh**, uç noktalar (endpoints), ağlar ve bulut ortamları için kapsamlı görünürlük sağlayan dünyanın önde gelen açık kaynaklı güvenlik platformudur. Modern siber güvenlik ihtiyaçlarını karşılamak üzere **SIEM (Security Information and Event Management)** ve **XDR (Extended Detection and Response)** yeteneklerini tek bir çatı altında birleştirir.

---

## 🚀 Wazuh Neden Kullanılmalı?

Kurumsal ortamlarda güvenlik, sadece log toplamak değil, bu logları anlamlı bir tehdit istihbaratına dönüştürmektir. Wazuh bu noktada şu temel değerleri sunar:

- **Bütünleşik Güvenlik:** Tehdit algılama, uyumluluk, olay yanıtı ve envanter yönetimini tek bir agent ve dashboard üzerinden yönetin.
- **Maliyet Verimliliği:** Lisans ücreti olmadan kurumsal düzeyde özelliklere sahip olun.
- **Esneklik:** Hibrit bulut, on-premise ve çoklu bulut (Multi-cloud) ortamlarını destekler.
- **Topluluk ve Ekosistem:** Sürekli güncellenen kurallar ve geniş bir kullanıcı desteği.

---

## 🛡️ Temel Yetenekler

### 1. Tehdit Algılama ve Log Analizi

Wazuh, sistem loglarını (syslog, Event Channel, auditd vb.) toplar ve bunları binlerce hazır kural ile analiz eder. Brute force saldırılarından yetkisiz erişim denemelerine kadar her şeyi anlık olarak raporlar.

### 2. Dosya Bütünlüğü İzleme (FIM)

Kritik sistem dosyaları, dizinler ve Windows kayıt defteri (Registry) üzerindeki değişiklikleri izler. Kimin, neyi, ne zaman değiştirdiğini (Who-data) gerçek zamanlı olarak bildirir.

### 3. Zafiyet Tespiti (Vulnerability Detection)

Sistemdeki yüklü yazılımları envanter olarak toplar ve bunları güncel CVE (Common Vulnerabilities and Exposures) veritabanları ile karşılaştırır. Kritik zafiyetleri ve çözüm yollarını dashboard üzerinde gösterir.

### 4. Güvenlik Yapılandırma Analizi (SCA)

Sistemlerin CIS (Center for Internet Security) benchmark'larına ve en iyi uygulama standartlarına göre ne kadar güvenli yapılandırıldığını denetler.

### 5. Aktif Yanıt (Active Response)

Algılanan bir tehdide karşı otomatik tepki verir. Örneğin; saldırgan IP'sini engeller, kullanıcı hesabını askıya alır veya zararlı bir süreci sonlandırır.

### 6. Bulut Güvenliği

AWS, Azure ve GCP gibi platformlarla entegre olarak CloudTrail, GuardDuty ve VPC loglarını analiz eder. Bulut kaynaklarındaki yanlış yapılandırmaları tespit eder.

---

## 🏗️ Mimari Yapı

Wazuh üç ana bileşenden oluşur:

1. **Wazuh Indexer:** Verileri depolayan, arama ve analiz işlemlerini gerçekleştiren merkezi motor.
2. **Wazuh Server:** Agent'lardan gelen verileri analiz eden, kuralları çalıştıran ve tehditleri algılayan beyin.
3. **Wazuh Dashboard:** Tüm verilerin görselleştirildiği, raporların alındığı ve yönetim işlemlerinin yapıldığı arayüz.

---

## 📚 Wazuh Master Series: Eğitim İçeriği

Bu proje, Wazuh platformunu uçtan uca öğrenmeniz için hazırlanmış 22 modülden oluşan bir rehberdir.

### Modül Listesi

- [01: Wazuh Nedir? Mimari ve Temel Kavramlar](01_Giris_ve_Mimari.md)
- [02: Kurulum Senaryoları (All-in-one vs Distributed)](02_Kurulum_Senaryolari.md)
- [03: Wazuh Agent Kurulumu ve Yönetimi](03_Agent_Yonetimi.md)
- [04: Kurallar ve Dekoderlar Mantığı](04_Kurallar_ve_Dekoderlar.md)
- [05: Dosya Bütünlüğü İzleme (FIM)](05_FIM_Dosya_Izleme.md)
- [06: Rootkit ve Zararlı Yazılım Tespiti](06_Rootcheck_ve_Malware.md)
- [07: Güvenlik Yapılandırma Değerlendirmesi (SCA)](07_SCA_Yapilandirma_Denetimi.md)
- [08: Zafiyet Tespiti (Vulnerability Detection)](08_Zafiyet_Tespiti.md)
- [09: Log Verisi Toplama ve Analizi](09_Log_Analizi.md)
- [10: Aktif Yanıt (Active Response)](10_Aktif_Yanit.md)
- [11: Regülasyon Uyumlulugu (PCI DSS, GDPR)](11_Regulasyon_Uyumlulugu.md)
- [12: Bulut Güvenliği (AWS, Azure, GCP)](12_Bulut_Guvenligi.md)
- [13: Container ve Kubernetes Güvenliği](13_Container_ve_K8s.md)
- [14: Wazuh API Kullanımı ve Otomasyon](14_API_ve_Otomasyon.md)
- [15: Dashboard Özelleştirme ve Raporlama](15_Dashboard_ve_Raporlama.md)
- [16: Syscheck ve Syscollector Detayları](16_Syscheck_ve_Syscollector.md)
- [17: OSSEC vs Wazuh Farkları](17_OSSEC_vs_Wazuh.md)
- [18: Entegrasyonlar (Slack, Email, Jira)](18_Entegrasyonlar.md)
- [19: Yedekleme ve Cluster Yönetimi](19_Yedekleme_ve_Cluster.md)
- [20: Performans İyileştirme ve Tuning](20_Performans_Tuning.md)
- [21: Gerçek Hayat Senaryoları](21_Gercek_Hayat_Senaryolari.md)
- [22: Hızlı Referans (Cheat Sheet)](CheatSheet.md)

---
*Bu Master Series, Siber Güvenlik Uzmanları için rehber niteliğindedir.*

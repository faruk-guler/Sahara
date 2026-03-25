# Modül 17: Kurumsal Bakım Yönetimi (Maintenance Mode)

On binlerce cihaz barındıran altyapılarda "Sistemi yamalayacağız sunucuyu kapat" demek lükstür ve planlı bir çalışmanın Zabbix'i "Kırmızı alarm" denizine boğup felaket çanları çaldırması önlenmelidir. **Maintenance (Bakım Yeri)**, sistemde ne olup bittiğini zekice susturan mekanizmadır.

## 1. Mimaride İki Çeşit Bakım Türü (Data Collection)

Bakım oluştururken en çok karıştırılan menü `Maintenance type` seçimidir. Bu seçim SLA raporlarınızı sonsuza kadar bozabilir.

### 1.1 "With Data Collection" (Veri Toplanarak Devam Eden Bakım)

Sunucuyu kapatıp Ram yükseltmesi değil, üzerine yeni bir yazılım yükleme işlemi (Örn: Web uygulamasını derleme) yapıyorsunuz diyelim.

- Zabbix sunucudan CPU, Ping, RAM verilerini sanki hiçbir şey yokmuş gibi saniye saniye okumaya, kaydetmeye ve arşivlemeye (DB'ye yazmaya) aynen devam eder.
- Grafiklerde (Çizgilerde) asla boşluk görülmez.
- Tek fark şudur: Eğer bu işlem esnasında CPU %100 olursa, Zabbix Trigger üretse dahi "Send Message" (SMS atmayı) iptal eder. Kimseyi rahatsız etmez.
- *Aylık Uptime/SLA hesabında (Modül 11) o süre tamamen hesaba katılır!*

### 1.2 "Without Data Collection" (Ağzı Kilitli, Kör Bakım)

Sunucunun komple fişinin çekilmesi, fiziksel sunucularda anakart değişimi, lokasyon/Ağ anahtarı (Switch) değişimi durumunda kullanılır.

- Zabbix o cihazla irtibatı 1 saniye bile olsa anında keser.
- Veritabanına hiçbir bilgi yazılmaz. Grafiklerde `Gece 02:00 ile 05:00` arası tamamen boş (Eksik) bir çizgi doğar!
- Avantajı: Hem alarm gitmez hem de Zabbix Poller (veri okuyucu işçiler) kapalı makineye boştan yere 3 saat ping atıp Timeout'a düşerek "Time_Wait" yaratmaz (Performans için altın kuraldır).

## 2. Akıllı Tagging (Etikete Göre Bakım Almak)

Zabbix'in eski versiyonlarında Ankara'da yapılacak bir çalışma için önce `Ankara Cihazları` isminde geçici Host Group (Host Grubu) kurulur; cihazlar oraya sürüklenirdi.
2026 RBAC ve modern ITIL düzeninde makineye değil Tag (Etiket)'e bakım yazılır.

**Senaryo:** "Ankara'daki SADECE PostgreSQL Veritabanı Kümesinde 30dk Bakım var."

Çözüm:
Zabbix menüsünden `Data Collection` -> `Maintenance` alanına girildiğinde;

- `Host group`: Ankara Location seçilir.
- `Tags (Or/And matrisi)` altına: *Tag=Service, Value=PostgreSQL* girilir.

Bu kural oluşturulduğunda Zabbix, Ankara'daki Linux Web, Windows, Switch cihazlarının hiçbirini bakım moduna (Turuncu ikon) almaz. Yalnızca üzerinde *Service=PostgreSQL* etiğine sahip olanlara Mute (Susturucu) uygular. Tag mimarisinin gücü budur.

## 3. API ve CI/CD Üzerinden Otomatik Maintenance (Kurumsal Mimari)

DevOps mühendisleri CI/CD pipeline (Jenkins/GitLab) üzerinden koda push atar atmaz sunucuyu otomatik deploy ediyorlarsa Zabbix arayüzünden bakım almaları (manuel işlem) felsefeye aykırıdır.

Master yaklaşımda: GitLab CI'da yazılım dağıtımı (Deployment) başladığı 1. saniye, Jenkins PypLine'ı Zabbix API (Modül 15) adresine bir API Token ile istek atar.
Zabbix'te makineyi 10 dakikalığına uçtan uca anında `Maintenance` moduna çeker. (Müdahalesiz Gözlemlenebilirlik).

---
[Önceki Modül](./16_Database_Optimizasyon.md) | [README'ye Dön](./README.md) | [Sonraki Modül: SNMP Traps](./18_SNMP_Traps.md)

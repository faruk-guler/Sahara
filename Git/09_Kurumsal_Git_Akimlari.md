# Modül 09: Kurumsal Git Akımları (Workflows)

Ekip büyüdükçe, "Herkes canı ne zaman isterse main'e commit atsın" mantığı çalışmaz. Kodun test aşamaları, canlı sunucuya (Production) aktarımı ve hata (hotfix) yönetimini standartlaştıran Git Çalışma Modelleri (Workflows) devreye girer.

Sektörde en yaygın kullanılan modeller şunlardır:

## 1. Git Flow 

Vincent Driessen tarafından yaratılan çok katı ve kurumsal bir modeldir. Genellikle canlı yayın (release) takvimleri önceden belirli, büyük projelerde kullanılır.

**Temel Dalları (Branches):**
- **`main` (veya `master`):** Her zaman tam stabil, Production ortamındaki koddur. Direkt commit ASLA yapılmaz. Her commitin bir versiyon Tag'i (v1.0.0) vardır.
- **`develop`:** Ana geliştirme dalıdır. Sonraki versiyon için hazır olan kodlar burada toplanır.

**Geçici Dallar:**
- **`feature/xxx`:** Yeni özellik ekleneceği zaman `develop`'tan ayrılır. İş bitince tekrar `develop`'a merge edilir.
- **`release/v1.1`:** Yeni sürüm yayınlanmaya karar verildiğinde `develop`'tan ayrılır. Sadece son bugfixler test edilir "Özellik eklenmez". Bitince hem `main`'e hem `develop`'a merge edilir.
- **`hotfix/xxx`:** Canlıda (Production) acil bir hata bulunduğunda doğrudan `main`'den ayrılır. Hata düzeltilir, hem `main`'e hem `develop`'a anında merge edilir.

*Git Flow karmaşıktır, sürekli ürün çıkılan SaaS (Software as a Service) projeleri için çok ağır (burokratik) kalabilir.*

## 2. GitHub Flow

GitHub tarafından tercih edilen, oldukça hafif, Çevik (Agile) ve Sürekli Dağıtım (Continuous Deployment) odaklı bir modeldir.

**Kurallar:**
1. Tek bir ana dal vardır: **`main`**. `main` her an Production'a deploy edilebilir durumdadır.
2. Yeni çalışılacak her iş için `main`'den kısa ömürlü ve isimlendirilmiş *(descriptive)* branch açılır (`feature/login-page` vb.).
3. İş bitince, sunucuda (GitHub arayüzünde) bir **Pull Request (PR)** açılır. Kod takım tarafından review edilir, varsa eksikleri dalda düzeltilir.
4. Onaylanan PR doğrudan `main` branch'e merge edilir.
5. Merge işlemi biter bitmez otomatik CI/CD süreçleri ile "Canlı"ya çıkarılır.

## 3. GitLab Flow

GitHub Flow ile Git Flow arasında bir köprüdür. GitHub Flow'un basitliğini alır ancak "Environment (Ortam)" kavramlarını ekler.

Tıpkı GitHub Flow gibi branch + PR mantığıyla ana dala (`main`) geliştirme yapılır. Ama şirket politikasında "Staging" (Test Ortamı) ve "Production" (Canlı Ortam) varsa, bu dallar ortamlara bağlanır:

- `main` branch -> **Staging (Test)** sunucusuna deploy edilir.
- `main`den kodu alıp `production` branch'ine merge ederseniz -> **Production (Canlı)** sunucusuna deploy edilir.

Böylece hangi kodun hangi ortamda olduğunu Git History'den açıkça okuyabilirsiniz.

## 4. Trunk-Based Development

DevOps kültürünün geldiği son noktadır. Sadece çok gelişmiş "Automated Testing" (Birim ve Entegrasyon testleri) altyapısına sahip elit ekiplerde başarılı olur.

- **Kural:** Geliştiriciler çok kısa ömürlü dallar (max 1 günlük) açarak sürekli ve günlük olarak ana dala (Trunk / Main) kodu entegre ederler.
- Bitmemiş özellikler kodda "Feature Flags / Toggles" (Özellik anahtarları) ile canlıdan gizlenecek şekilde main'e gömülür.
- Asla devasa "Merge Conflict" savaşları yaşanmaz. Herkes günde 3-4 kere kod birleştirir.

> [!TIP]
> **Hangi Workflow'u Seçmeliyim?**
> Eğer günde birden fazla kez canlıya (Production) çıkış yapabilen web tabanlı SaaS bir ürününüz varsa **GitHub Flow** veya **Trunk-Based**; Ayda veya haftada paket (Masaüstü, Mobil Uygulama vb.) sürümü çıkartıyorsanız **Git Flow** daha uygundur.

---

[← Geri: Modül 08](08_Alt_Moduller_ve_Buyuk_Dosyalar.md) | [İleri: Modül 10 - İpuçları, Kısayollar ve Performans →](10_Ipuclari_Kisayollar_ve_Performans.md)

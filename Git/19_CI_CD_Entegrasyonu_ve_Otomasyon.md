# Modül 19: CI/CD Entegrasyonu ve Otomasyon

Git, modern yazılım geliştirme süreçlerinin (DevOps) merkezidir. Bu modülde, Git'in otomasyon sistemleri ve sunucu taraflı yeteneklerini inceleyeceğiz.

---

## 9.1 Server-side Hooks (Sunucu Taraflı Kancalar)

Önceki modüllerde (7) istemci taraflı kancaları görmüştük. Sunucu tarafında (GitLab, GitHub Enterprise vb.) en önemli kancalar şunlardır:

* **pre-receive:** Kural ihlali (örn. büyük dosya yükleme, yasaklı kelime) varsa tüm push işlemini reddeder.
* **update:** Reddedilecek veya kabul edilecek dalları (branch) tek tek seçer.
* **post-receive:** Push bittikten sonra çalışır. Genellikle CI sunucusuna (Jenkins/GitLab CI) "Yeni kod geldi, build et!" sinyali gönderir.

---

## 9.2 Feature Toggles vs Git Branching

Bir özelliği geliştirdiniz ama henüz canlıya (Production) vermeye hazır değilsiniz. Eskiden bu özellikler ayrı bir dalda aylarca beklerdi (`merge hell` yaşanırdı). **Feature Toggles (Özellik Anahtarları)** ile kodu `main` dalına merge edebilir ama canlıda kapalı tutabilirsiniz.

**Faydası:** `main` dalı her zaman güncel kalır ve büyük çatışmalar önlenir.

---

## 9.3 Git-based Deployment (Git ile Dağıtım)

Bazı küçük/orta ölçekli projelerde, sunucuya SSH ile girip `git pull` yapmak yerine, sunucuyu bir Git remote olarak ekleyebilirsiniz.

```bash
# Sunucuda (Server side):
git init --bare repo.git

# Kendi bilgisayarınızda (Client side):
git remote add production ssh://user@server:/yol/repo.git
git push production main
```

*Sunucudaki `post-receive` hook'u otomatik olarak dosyaları web klasörüne kopyalar ve servisi restart eder.*

---

## 9.4 GitHub Actions ve GitLab CI Konfigürasyonu

Artık projelerin çoğunda `.github/workflows/` veya `.gitlab-ci.yml` dosyaları bulunur. Git, bu dosyaları gördüğü anda otomatik testleri başlatır.

### Örnek (Lint ve Test)

```yaml
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm test
```

---

## 9.5 Protected Branches (Korunan Dallar)

Merkezi sistemlerde (GitHub/Lab), belirli dalları (main/develop) koruma altına alabilirsiniz:

* Doğrudan push yapılamaz (Sadece Pull Request / Merge Request).
* Kod ancak en az 2 kişi tarafından onaylanınca merge edilebilir.
* CI testleri (status checks) geçmeden merge butonuna basılamaz.

---

[← Geri: Modül 18](18_Dev_Projelerde_Calisma_Teknikleri.md) | [İleri: Modül 20 - Bakım, Performans ve Plumbing →](20_Bakim_Performans_ve_Plumbing.md)

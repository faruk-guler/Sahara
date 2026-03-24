# Modül 07: Git Hook'ları ve Otomasyon

Git Hook'ları, Git komutları (commit, push, merge vs.) çalıştırılmadan hemen önce veya çalıştırıldıktan hemen sonra otomatik olarak tetiklenen özel kod/script parçacıklarıdır. Kurumsal projelerde kod kalitesini garanti altına almak için vazgeçilmezdir.

## 1. Git Hooks Nerede Bulunur?

Her Git deposunda, gizli `.git` klasörünün altında `hooks` adında bir klasör vardır (`.git/hooks/`). Git ilk kurulduğunda bu klasörde `.sample` uzantılı örnek scriptler bulunur.

```bash
cd .git/hooks
ls -l
```

## 2. İstemci Taraflı (Client-Side) Hook'lar

Geliştiricinin kendi bilgisayarında tetiklenen hook'lardır.

### `pre-commit` (En Çok Kullanılan)
`git commit` yazıp enter'a bastığınız anda, commit mesajı sorulmadan *önce* çalışır. Çoğunlukla **Code Linting**, **Birim Testleri (Unit Tests)** veya **Güvenlik Taramaları** için kullanılır. Eğer bu script "0" dışında bir hata kodu dönerse, commit işlemi iptal edilir (abort).

**Örnek Senaryo:** Kötü formatlanmış bir Python/JS kodunun depoya girmesini engellemek için `pre-commit` scripti içine linter aracını yerleştirebilirsiniz.

### `commit-msg`
Commit mesajı girildikten sonra çalışır. Mesajın şirket standartlarına (örneğin kurumsal Jira/Trello issue numarası içermesi gibi) uygun olup olmadığını denetler.

### `pre-push`
`git push` komutundan hemen önce çalıştırılır. Ağır entegrasyon testlerini uzaktaki CI/CD pipeline'ını meşgul etmeden önce yerel makinede son kez doğrulamak için harikadır.

> [!TIP]
> Bir hook'u (örneğin `pre-commit`) istisnai bir durumda es geçmek isterseniz `--no-verify` flag'i kullanabilirsiniz:
> `git commit -m "Acil fix" --no-verify`

## 3. Sunucu Taraflı (Server-Side) Hook'lar

Kodu alan merkezi sunucuda (örneğin GitHub, GitLab sunucusunda) çalışır.

### `pre-receive`
İstemcilerden gelen `push` isteklerini sunucu tarafında doğrular. Örneğin "Sadece takım lideri main branch'e push yapabilir" kuralı burada implemente edilebilir.

### `post-receive`
Push işlemi başarıyla bittikten sonra çalışır. Genellikle **CI/CD tetiklemelerinde** (Jenkins, Travis vb.) veya kodu canlı sunucuya otomatik deploy (dağıtma) işlemlerinde kullanılır.

## 4. Hook'ları Takım ile Paylaşmak

`.git/hooks` dizini varsayılan olarak Git repo tarihçesine dâhil edilmez (untracked'dir). Bu yüzden yazdığınız harika `pre-commit` hook'unu takım arkadaşlarınız doğrudan alamaz.

**Çözüm:** Hook'ları projenin ana dizininde `.githooks` adlı açık bir klasörde tutun ve herkesin Git ayarlarını bu klasörü okuyacak şekilde yapılandırın:

```bash
git config core.hooksPath .githooks
```
*(Böylelikle `.githooks` klasörü Git üzerinden paylaşımlı hale gelir.)*

## 5. Modern Alternatifler: Husky ve Pre-Commit Framework

Shell scriptleri yazmak zor geldiyse:
- **Node.js/JS Projeleri İçin:** `Husky` paketi endüstri standartıdır. Sadece bir `package.json` ayarıyla tüm hookları yönetirsiniz.
- **Python ve Genel Projeler İçin:** `pre-commit` (framework) aracı yaml dosyası ile hook konfigürasyonu sağlar.

---

[← Geri: Modül 06](06_Hata_Ayiklama_ve_Geri_Alma.md) | [İleri: Modül 08 - Alt Modüller ve Büyük Dosyalar →](08_Alt_Moduller_ve_Buyuk_Dosyalar.md)

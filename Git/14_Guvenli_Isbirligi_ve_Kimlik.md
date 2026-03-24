# Modül 14: Güvenli İşbirliği ve Kimlik

Kurumsal projelerde, sadece commit atmak yetmez; bu commit'lerin kime ait olduğunu kanıtlamak ve uzak sunucularla (GitHub, GitLab vb.) güvenli iletişim kurmak gerekir.

---

## 4.1 SSH Key Yönetimi

HTTPS yerine SSH kullanmak, her seferinde şifre girme zahmetini ortadan kaldırır ve daha güvenlidir.

```bash
# Yeni bir Ed25519 (modern ve güvenli) anahtar üretmek:
ssh-keygen -t ed25519 -C "emailiniz@sirket.com"

# Üretilen public key'i (.pub) kopyalayıp GitHub ayarlarına ekleyin:
cat ~/.ssh/id_ed25519.pub
```

### Çoklu Hesap Yönetimi (~/.ssh/config)

Eğer hem kişisel hem şirket GitHub hesabınız varsa, SSH config dosyası ile bunları ayırabilirsiniz:

```text
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_kisisel

Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_sirket
```

---

## 4.2 GPG ile Commit İmzalama

Git, aslında herhangi birinin `user.name` ve `user.email` bilgisini taklit etmesine izin verir. Ancak **GPG (GNU Privacy Guard)** imzalama ile commit'lerin gerçekten sizden geldiğini kanıtlayabilirsiniz (GitHub'da "Verified" etiketi görünür).

```bash
# Anahtar listesini gör:
gpg --list-secret-keys --keyid-format LONG

# Git'e anahtarınızı tanıtın:
git config --global user.signingkey [ANAH_ID]
git config --global commit.gpgsign true
```

---

## 4.3 Upstream ve Multi-Remote Senkronizasyonu

Bir projeyi "fork" ettiğinizde (veya ortak bir repo üzerinde çalışırken), hem kendi "origin" adresiniz hem de ana projenin "upstream" adresi bulunur.

```bash
# Ana proje adresini ekle:
git remote add upstream https://github.com/ana-proje/app.git

# Ana projeden güncellemeleri çekip kendi dalına merge et:
git fetch upstream
git merge upstream/main
```

---

## 4.4 Personal Access Tokens (PAT)

Geleneksel şifrelerin yerine geçen, kapsamı (scope) sınırlı anahtarlardır. Özellikle CI/CD sistemlerinde veya 2FA (İki Faktörlü Doğrulama) aktif hesaplarda zorunludur.

> [!WARNING]
> PAT anahtarlarını asla kod paketlerine veya GitHub'daki dosyalar içine commit etmeyin! Bunun için `.env` dosyalarını veya GitHub Secrets gibi ortam değişkenlerini kullanın.

---

## 4.5 Güvenlik Taramaları: git-secrets

Repo içine yanlışlıkla AWS Key, API Key veya şifre sızmasını önlemek için AWS tarafından geliştirilen `git-secrets` gibi araçlar `pre-commit` hook'larında her saniye tetikte beklemelidir.

---

[← Geri: Modül 13](13_Stratejik_Dallanma_ve_Rerere.md) | [İleri: Modül 15 - Tarihçeyi Ustaca Yeniden Yazma →](15_Tarihceyi_Ustaca_Yeniden_Yazma.md)

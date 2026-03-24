# Git Professional Cheat Sheet

Git komutlarını hızlıca hatırlamak için hazırlanan, kategorize edilmiş ve optimize edilmiş referans dökümanı.

## İçindekiler

1. [Yapılandırma (Setup & Config)](#1-yapılandırma-setup--config)
2. [Depo Oluşturma (Starting a Repo)](#2-depo-oluşturma-starting-a-repo)
3. [Temel İş Akışı (Basic Workflow)](#3-temel-iş-akışı-basic-workflow)
4. [Dallanma ve Etiketleme (Branching & Tags)](#4-dallanma-ve-etiketleme-branching--tags)
5. [Uzak Depolar (Remote Repos)](#5-uzak-depolar-remote-repos)
6. [Birleştirme ve Yeniden Temellendirme (Merge & Rebase)](#6-birleştirme-ve-yeniden-temellendirme-merge--rebase)
7. [Geri Alma ve Sıfırlama (Undo & Reset)](#7-geri-alma-ve-sıfırlama-undo--reset)
8. [Geçici Kaydetme (Stash)](#8-geçici-kaydetme-stash)
9. [Sorgulama ve Arama (Search & Inspect)](#9-sorgulama-ve-arama-search--inspect)
10. [Gelişmiş Özellikler (LFS & Submodules)](#10-gelişmiş-özellikler-lfs--submodules)

---

## 1. Yapılandırma (Setup & Config)

### Kimlik Bilgileri

```bash
# Global kullanıcı adı ve email ayarla
git config --global user.name "Adınız Soyadınız"
git config --global user.email "email@örnek.com"

# Yapılandırmayı listele
git config --list
git config --global --list
```

### Araçlar ve Renkler

```bash
# Otomatik renklendirmeyi aç
git config --global color.ui auto

# Varsayılan editörü ayarla
git config --global core.editor "code --wait" # VS Code için
```

---

## 2. Depo Oluşturma (Starting a Repo)

```bash
# Mevcut dizinde yeni bir repo başlat
git init

# Belirli bir dizinde repo başlat
git init <dizin_adı>

# Uzak bir repoyu kopyala
git clone <url>
```

---

## 3. Temel İş Akışı (Basic Workflow)

### Durum ve Değişiklikler

```bash
# Dosya durumlarını gör (staging, untracked)
git status

# Değişiklikleri incele
git diff
git diff <dosya_adı>
```

### Ekleme ve Commit

```bash
# Dosyayı staging'e ekle
git add <dosya_adı>

# Tüm değişiklikleri ekle
git add .

# Mesajla birlikte commit at
git commit -m "Commit mesajı"

# Staging'i atlayarak tüm değişiklikleri commit et
git commit -am "Hızlı commit mesajı"
```

---

## 4. Dallanma ve Etiketleme (Branching & Tags)

### Dallar (Branches)

```bash
# Dalları listele
git branch      # Yerel
git branch -a   # Hepsi (yerel + uzak)

# Yeni dal oluştur
git branch <dal_adı>

# Bir dala geçiş yap
git checkout <dal_adı>

# Yeni dal oluştur ve ona geç
git checkout -b <dal_adı>

# Dalı sil (dikkat: birleştirilmemişse hata verir)
git branch -d <dal_adı>
git branch -D <dal_adı> # Zorla sil
```

### Etiketler (Tags)

```bash
# Etiketleri listele
git tag

# Yeni etiket oluştur
git tag v1.0

# Mesajlı etiket (Annotated)
git tag -a v1.0 -m "Versiyon 1 serbest bırakıldı"
```

---

## 5. Uzak Depolar (Remote Repos)

```bash
# Yapılandırılmış uzak depoları gör
git remote -v

# Yeni uzak depo ekle
git remote add origin <url>

# Uzak depodaki değişiklikleri çek (merge etmez)
git fetch <remote>

# Uzak depodaki değişiklikleri çek ve merge et
git pull origin <dal_adı>

# Değişiklikleri uzak depoya gönder
git push origin <dal_adı>

# Etiketleri gönder
git push --tags
```

---

## 6. Birleştirme ve Yeniden Temellendirme (Merge & Rebase)

```bash
# Belirli bir dalı mevcut dala birleştir
git merge <dal_adı>

# Mevcut dalı başka bir dalın üzerine taşı (Rebase)
git rebase <dal_adı>

# Rebase işlemini iptal et
git rebase --abort

# Rebase çakışma sonrası devam et
git rebase --continue
```

---

## 7. Geri Alma ve Sıfırlama (Undo & Reset)

```bash
# Dosyayı son commit haline geri döndür
git restore <dosya_adı>
git checkout HEAD -- <dosya_adı> # Eski yöntem

# Dosyayı staging alanından çıkar (unstage)
git restore --staged <dosya_adı>
git reset HEAD <dosya_adı> # Eski yöntem

# Son commit mesajını düzelt
git commit --amend -m "Yeni mesaj"

# Sert sıfırlama (DİKKAT: Değişiklikleri silebilir!)
git reset --hard HEAD
git reset --hard <commit_sha>

# Geri alma (yeni bir commit oluşturur)
git revert <commit_sha>
```

---

## 8. Geçici Kaydetme (Stash)

```bash
# Değişiklikleri geçici olarak sakla
git stash
git stash push -m "Mesaj"

# Stash listesini gör
git stash list

# Son stash'i geri getir ve sil
git stash pop

# Stash'i geri getir ama listede tut
git stash apply stash@{0}

# Stash'i temizle/sil
git stash drop stash@{0}
git stash clear # Tüm stashleri siler
```

---

## 9. Sorgulama ve Arama (Search & Inspect)

```bash
# Commit geçmişini gör
git log
git log --oneline --graph --decorate
git log -p <dosya_adı> # Dosyadaki değişiklik geçmişi

# Kim, neyi, ne zaman değiştirdi?
git blame <dosya_adı>

# Kod içinde arama yap (Git entegrasyonlu)
git grep "aranan_kelime"

# Reflog (HEAD'in tüm hareketleri)
git reflog
```

---

## 10. Gelişmiş Özellikler (LFS & Submodules)

### Git LFS (Büyük Dosyalar)

```bash
# LFS kurulumu
git lfs install

# Belirli dosyaları LFS ile takip et
git lfs track "*.psd"

# LFS durumunu gör
git lfs status
```

### Submodules (Alt Modüller)

```bash
# Alt modül ekle
git submodule add <url> <dizin>

# Alt modülleri güncelle
git submodule update --init --recursive

# Alt modülleri ile birlikte clone et
git clone --recurse-submodules <url>
```

---

[← Ana Sayfaya Dön](README.md)

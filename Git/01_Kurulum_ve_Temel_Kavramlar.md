# Modül 01: Kurulum ve Temel Kavramlar

Bu modülde Git versiyon kontrol sisteminin ne olduğu, temel mimarisi, kurulum adımları ve ilk kullanım için gerekli ayarlar detaylandırılmıştır.

## 1. Git Mimarisi ve Temel Kavramlar

Git, bir **Dağıtık Versiyon Kontrol Sistemi (DVCS)**'dir. Her geliştiricinin makinesinde, projenin tarihçesini içeren tam bir depo (repository) kopyası bulunur. 

Git mimarisinde en çok bilinmesi gereken üç ana bölge vardır:

1. **Working Directory (Çalışma Klasörü):** Proje dosyalarınızın bilgisayarınızda (diskinizde) bulunduğu ve üzerinde değişiklik yaptığınız alandır.
2. **Staging Area / Index (Hazırlık Alanı):** Bir sonraki `commit`'e dâhil edilecek dosyaların paketlendiği/işaretlendiği ara bölgedir.
3. **Repository / .git klasörü (Depo):** Tüm `commit`'lerin (versiyonların) tarihçesiyle beraber güvenle saklandığı, Git'in kalbidir.

> [!NOTE]
> Git dosya bazlı değil, **"Snapshot" (Anlık Görüntü)** bazlı bir sistemdir. Bir commit alındığında, sadece değişen dosyalar değil, tüm projenin o anki haritası (snapshot) kaydedilir, ancak sadece değişen kısımlar diskte yeni bir obje olarak saklanarak performans optimize edilir.

## 2. Git Kurulumu

### Windows İçin Kurulum
Windows'ta Git kurmanın en pratik yolu resmi yükleyiciyi indirmektir.

1. [git-scm.com/download/win](https://git-scm.com/download/win) adresinden yükleyiciyi indirin.
2. Kurulum sihirbazında default (varsayılan) ayarları kullanabilirsiniz ancak ilerleyen adımlarda:
   - *Default Editor* olarak **VS Code**, **Notepad++** ya da **Vim** seçebilirsiniz.
   - *Path Environment* adımında, komut satırı araçları için "Git from the command line and also from 3rd-party software" seçeneğinde kalması tavsiye edilir.

### Linux (Debian/Ubuntu) İçin Kurulum
```bash
sudo apt update
sudo apt install git
```

### macOS İçin Kurulum
Eğer sisteminizde Homebrew kurluysa:
```bash
brew install git
```
*(Ya da terminale sadece `git` yazarak Apple'ın geliştirici araçlarını yükleme penceresini tetikleyebilirsiniz.)*

## 3. İlk Konfigürasyon Ayarları (`git config`)

Git'i kurduktan sonra atmanız gereken ilk adım, yaptığınız commit'lerin kime ait olduğunu belirtecek yazar (user) bilgilerini ayarlamaktır. Aksi halde Git commit yapmanıza izin vermez.

Git ayarları 3 seviyede tutulur:
- **`--system`**: Tüm kullanıcılar ve tüm repolar için.
- **`--global`**: Sadece mevcut işletim sistemi kullanıcısı için tüm repolarda (En çok kullanılan).
- **`--local`**: Sadece içinde bulunulan repository için.

### Kullanıcı Bilgilerini Tanımlama (Global)

```bash
git config --global user.name "Adınız Soyadınız"
git config --global user.email "sirketveya@kisisel-email.com"
```

> [!WARNING]
> GitHub, GitLab veya Bitbucket'a kod gönderecekseniz, yukarıda belirttiğiniz e-posta adresi uzak sunucudaki (örneğin GitHub hesabı) e-posta adresiniz ile aynı (veya onaylanmış) olmalıdır. Aksi halde commit'ler sizin hesabınıza linklenmez.

### Varsayılan Editörü Ayarlama
Commit mesajları veya conflict çözümleri için Git'in çalıştıracağı editörü seçmek işlemlerinizi hızlandırır. Örnek olarak VS Code seçimi:

```bash
git config --global core.editor "code --wait"
```

### Git Çıktılarını Renklendirme
Komut çıktılarının (özellikle diff ve status) okunabilirliğini artırmak için otomatik renklendirme iyi bir pratiktir:

```bash
git config --global color.ui auto
```

### Varsayılan Branch Adını "main" Yapmak
Eski Git versiyonlarında varsayılan dal adı `master` olarak gelir. Sektör standardı gereği bunu `main` yapmak faydalıdır:

```bash
git config --global init.defaultBranch main
```

### Tüm Ayarları Görüntüleme
Mevcut konfigürasyonu görmek için:

```bash
git config --list
# veya
git config --global --list
```
> [!TIP]
> Bir ayarın hangi seviyeden geldiğini (`global`, `local`, `system`) görmek istiyorsanız `--show-origin` flag'ini kullanabilirsiniz:
> `git config --list --show-origin`

---

[← Ana Sayfaya Dön](README.md) | [İleri: Modül 02 - Temel Komutlar ve Çalışma Akışı →](02_Temel_Komutlar_ve_Calisma_Akisi.md)

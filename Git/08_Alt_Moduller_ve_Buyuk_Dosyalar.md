# Modül 08: Alt Modüller ve Büyük Dosyalar

Projeniz çok büyüdüğünde, başkalarının projelerini kendi projenizin içinde barındırmak (Submodules) veya devasa medya/oyun dosyalarını versiyonlamak (Git LFS) ihtiyacı doğar.

## 1. Alt Modüller (`git submodule`)

Bir projeyi başka bir projenin alt klasörü olarak kullanmak ama aynı zamanda o alt klasörün bağımsız bir Git reposu olarak kalmasını istediğimiz durumlar için `submodule` kullanılır. 

Örnek kullanım senaryosu: Kendi geliştirdiğiniz ortak bir kütüphaneyi, 5 farklı projeniz içinde güncel tutabilmek.

### Submodule Eklemek
```bash
git submodule add https://github.com/kullanici/ortak-kutuphane.git lib/ortak-kutuphane
```
Bu işlem projenize özel bir `.gitmodules` dosyası oluşturur ve o kütüphaneyi çekip kitler.

### Başkasının Submodule İçeren Projesini Clone'lamak
Normal `git clone` komutu alt modüllerin klasörlerini yaratır ama içlerini boş bırakır. Submodule'leri de doldurarak clone'lamak için:

```bash
git clone --recurse-submodules https://github.com/kullanici/ana-proje.git
```

Eğer düz clone yaptıysanız, sonradan içlerini doldurmak için:
```bash
git submodule init
git submodule update
```

## 2. Büyük Dosyalar ve Git LFS (Large File Storage)

Git metin (text/code) tabanlı dosyalar için harikadır ama 500 MB boyutundaki PSD, MP4, veya 3D `.obj` dosyalarını sevmez. Git her commit'te o devasa dosyanın değişimini (farkı bulamadığı için kopyasını) depolar, bu da repository'nin boyutunu hızla GB'lara ulaştırır ve sistemi kilitler.

Bunun çözümü açık kaynaklı eklenti **Git LFS (Large File Storage)** kullanmaktır.

### Git LFS Kurulumu ve Kullanımı

Önce bilgisayarınıza Git LFS kurun:
```bash
git lfs install
```

Sonra repository içinde hangi uzantıların LFS tarafından ele alınacağını (track) söyleyin:
```bash
git lfs track "*.psd"
git lfs track "*.mp4"
```

Bunu yaptığınızda dizinde `.gitattributes` isimli bir dosya oluşur.
Bu dosyayı ve büyük dosyalarınızı normal bir şekilde add/commit yaparsınız.

```bash
git add .gitattributes
git add video.mp4
git commit -m "feat: tanıtım videosu eklendi"
git push origin main
```

**Arka Planda Ne Olur?**
Git LFS, o .mp4 dosyasını uzak sunucuda (LFS Server) depolar. Sizin `.git` repositorynizin içine ise o dosyanın minik bir TXT "işaretçisini" (pointer) kaydeder. Böylece deponun boyutu KB boyutlarında kalır, hız ve performans korunur.

> [!WARNING]
> Eğer projenizi GitHub/GitLab'a yüklüyorsanız, Git LFS kotalarının (Genelde bedava kullanım için 1GB) olduğunu unutmayın. Kotayı aşmamak için gereksiz büyük dosyaları `git lfs untrack` ile silmeyi veya dış depolama (S3 vb.) linkleri kullanmayı düşünebilirsiniz.

---

[← Geri: Modül 07](07_Git_Hooklari_ve_Otomasyon.md) | [İleri: Modül 09 - Kurumsal Git Akımları →](09_Kurumsal_Git_Akimlari.md)

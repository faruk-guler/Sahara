# Modül 10: İpuçları, Kısayollar ve Performans

Git'i günlük iş hayatınızda kullanırken bazı tekrarlanan işleri kısa yollara (Alias) bağlamak ve depo büyüdüğünde yaşanacak performans (yavaşlık) sorunlarını gidermek için kullanılan bazı gizli araçlar vardır.

## 1. Git Alias (Kısayol) Tạo Etmek

"git terminalinde çok uzun komutları yazmaktan yoruldunuz mu?" sorusunun cevabı Git Alias'larıdır.

```bash
# Sık kullanılan statüs kontrolünü "st" yapmak
git config --global alias.st status

# "git checkout" kelimesini "co" yapmak
git config --global alias.co checkout

# "git commit -m" komutunu pratikleştirmek
git config --global alias.cm "commit -m"
```

Artık sadece `git st` yazmanız `git status` çıkartması için yeterlidir.

### Gelişmiş Log Formatı Kısayolu

`git log` komutunun standart çıktısı çok detaylıdır. Bunu takım dinamiklerinde grafiğe dökmek (tree view) adına efsanevi `git graph` partial log kısayolu yaratabilirsiniz:

```bash
git config --global alias.graph "log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

Daha sonra terminale `git graph` yazdığınızda inanılmaz renkli ve okunaklı, branch ayrılıklarını oklarla gösteren bir ağaç göreceksiniz.

## 2. Git Performans Yönetimi (`git gc`)

Siz Git üzerinde rebase, delete, reset yaptıkça arka planda (çöplükte) ulaşılamaz / kopuk objeler çoğalır. Git bir süre sonra log ve branch listeleme işlemlerinde yavaşlamaya başlayabilir.

Bu durumlarda repository'yi sıkıştırmak ve temizlemek için **Garbage Collection (GC)** çalıştırılır:

```bash
# Repo çöplüğünü temizle ve boyutu küçült
git gc --prune=now
```

Bu komut repoyu tarar, hiçbir commit veya dal listesine bağlı olmayan atık nesneleri kalıcı olarak siler ve dosyaları sıkıştırıp paketler.

> [!NOTE]
> `git gc` normal projelerde çok gerekmez çünkü Git komutları (örn. `git pull`) arka planda zaman zaman kendi mini Gc'sini çalıştırır.

## 3. Faydalı Küçük Komutlar

- **Kodu kim yazdı? (`git blame`):** Takım arkadaşlarınızdan birinin bozduğu "satırı" bulmak için harika bir adli tıp aracıdır.

  ```bash
  git blame main.js
  ```

  Her kod satırının yanında tarihi ve kimin yazdığı görünür.

- **Dosya ismini ve yerini kurallı değiştirmek (`git mv`):** Dosyanın adını Windows'ta sağ tıkla değiştirirseniz Git bunu "Silindi" ve "Yeni Dosya Yaratıldı" satırları ile karşılar. Git'e uygun klasör/isim değişimi için:

  ```bash
  git mv eski-isim.txt yeni-isim.txt
  ```

- **Sadece kimlerin çalıştığı bilgisini görmek (`git shortlog`):** Bir GitHub PR'ında kimler kaç commit katkı sağladığını görmek isterseniz:

  ```bash
  git shortlog -sn
  ```

## Tebrikler! 🚀

Bu konu ile **Git Master Dokümantasyon Seti'nin ilk aşamasını** (Level 1) tamamlamış bulunuyorsunuz. Git korkulacak kara bir terminal aracı değil, mantıksal çalışma katmanları olan profesyonel bir kurtarıcıdır.

*Hataları geri almaktan, branchleri kaybetmekten korkmayın; Git logları reflog sayesinde sizin korumanız altındadır!*

---

[← Geri: Modül 09](09_Kurumsal_Git_Akimlari.md) | [İleri: Modül 11 - Git Mimarisi ve İç Yapı →](11_Git_Mimarisi_ve_Ic_Yapi.md)

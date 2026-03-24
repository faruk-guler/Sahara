# Modül 04: Uzak Depolar ve İşbirliği

Geliştiriciler Git'i genelde kendi bilgisayarlarında kullandıktan sonra takım arkadaşlarıyla paylaşmak veya kodlarını buluta yedeklemek için "Uzak Depolara" (Remote Repository) ihtiyaç duyarlar. GitHub, GitLab, Bitbucket gibi platformlar remote repository hizmeti sunar.

## 1. Remote (Uzak Depo) Eklemek ve Görüntülemek

Yerel bilgisayarınızda bir depo oluşturduğunuzda bu deponun henüz dış dünya ile bir bağlantısı yoktur. Bir uzak depo adresi (URL) eklemeniz gerekir. Git jargonu gereği ana uzak depoya varsayılan olarak **`origin`** ismi verilir.

```bash
# Uzak depo eklemek
git remote add origin https://github.com/kullanici/proje.git

# Bağlı olan uzak depoları listelemek
git remote -v
```

Çıktı şu şekilde olacaktır:
```text
origin  https://github.com/kullanici/proje.git (fetch)
origin  https://github.com/kullanici/proje.git (push)
```

## 2. Push: Değişiklikleri İnternete Göndermek

Commit'lerinizi uzak sunucuya yüklemek için `push` komutu kullanılır. Ancak ilk defa yeni bir branch pushluyorsanız, uzak sunucuya o branch'in bağlanması için `-u` (veya `--set-upstream`) flag'ini kullanmalısınız.

```bash
# Main dalını origin'e ilk kez gönderirken
git push -u origin main

# Daha sonraki push işlemlerinde sadece:
git push
```

## 3. Uzaktan Veri Çekmek: Fetch ve Pull

Takım arkadaşlarınız uzak sunucuya kod göndermiş olabilir. Bu kodları kendi bilgisayarınıza almanız gerekir. Bunu yapmanın iki temel yolu vardır: `git fetch` ve `git pull`.

### `git fetch` (Güvenli Yol)
Uzak depodaki en güncel commitleri ve dalları yerel bilgisayarınıza indirir ancak **sizin dosyalarınızı değiştirmez.** Değişiklikleri sadece inceler, hazır hissettiğinizde manuel olarak `merge` yaparsınız.

```bash
git fetch origin
git branch -r  # Sadece remote branchleri listele
```

### `git pull` (Hızlı Yol)
`git pull` aslında arkasında iki komut çalıştırır: `git fetch` + `git merge`. Yani uzak depodaki değişiklikleri bilgisayarınıza indirir ve bir çakışma (conflict) yoksa anında çalışma klasörünüze (working tree) yedirir.

```bash
git pull origin main
```

> [!WARNING]
> Eğer projenizde uzun süre `pull` yapmadıysanız ve yerel commit'lerinizle takımın commit'leri farklı yönlere gittiyse, `pull` sırasında çakışma (merge conflict) yaşarsınız. Kodu `commit`lemeden veya `stash`'e almadan `pull` işlemi yapmamaya özen gösterin.

## 4. Takip Edilen Branchler (Tracking Branches)

Bir `push -u` işlemi yapıldığında, yereldeki branch ile remote branch eşleşir (tracking olur). Eşleşme durumunu ve ne kadar "önde/geride" olduğunuzu ayrıntılı şekilde görmek için:

```bash
git branch -vv
```

Çıktı örneği:
```text
* main 3f1a2 [origin/main: ahead 2, behind 1] feat: yeni css
```
*Bu çıktı, sizin 2 commit önde (push beklemede) ve 1 commit geride (pull beklemede) olduğunuzu söyler.*

## 5. Başkasının Projesini İndirmek: `git clone`

Sıfırdan var olan bir projeyi bilgisayarınıza indirmek için kullanılan komuttur. Bu komut, `git init`, `git remote add` ve `git pull` sarmalını tek kelimede bilgisayarınıza kopyalar.

```bash
git clone https://github.com/torvalds/linux.git
```

> [!TIP]
> Clone işlemi sırasında sonuna bir boşluk bırakıp klasör adı yazarak repo adından farklı bir dizin ismiyle indirme yapabilirsiniz.
> `git clone https://github.com/.../app.git BenimUygulamam`

---

[← Geri: Modül 03](03_Branch_ve_Merge_Stratejileri.md) | [İleri: Modül 05 - Gelişmiş Git Komutları →](05_Gelismis_Git_Komutlari.md)

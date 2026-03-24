# Modül 12: Gelişmiş Log ve Veri Arama

Büyük projelerde, binlerce commit arasından aradığınız bir değişikliği veya hatayı bulmak için sadece `git log` yeterli değildir. Bu modülde, Git'in güçlü "query" (sorgu) yeteneklerini inceleyeceğiz.

---

## 2.1 Git Pickaxe (-S): Belirli Bir Metnin Eklenişini/Silinişini Bulma

Bir değişken adının veya hata mesajının ilk ne zaman koda eklendiğini veya silindiğini merak ediyorsanız `-S` flag'i hayat kurtarır.

```bash
# "api_key_secret" string'inin geçtiği tüm commit'leri ara:
git log -S "api_key_secret"

# Hangi dosyada ve hangi satırda değiştiğini görmek için `-p` ile birleştirin:
git log -S "api_key_secret" -p
```

> [!TIP]
> **Regex ile Arama (-G):** Eğer tam kelime değil de, belirli bir kalıp (pattern) arıyorsanız `-G` kullanabilirsiniz.
> `git log -G "password|secret" --oneline`

---

## 2.2 Custom Pretty Formats (Kendi Log Formatınızı Tasarlayın)

Terminalde sıkıştığınızda, kendinize özel bir log çıktısı oluşturabilirsiniz.

```bash
# Sadece Commit Hash (|), YazarAdı (|), Tarih (|) ve Mesaj görünümü:
git log --pretty=format:"%h | %an | %ad | %s" --date=short
```

### Önemli Format Belirteçleri

* **%h:** Kısa Hash.
* **%an:** Yazar Adı (Author Name).
* **%ad:** Yazar Tarihi (Author Date).
* **%s:** Commit Konusu (Subject).
* **%d:** Ref adları (Branch/Tag işaretleri).

---

## 2.3 rev-list: Programatik Log Analizi

`git rev-list`, Git loglarının ham halidir ve genellikle scriptlerde kullanılır. Sadece commit SHA listesini döner.

```bash
# Belirli iki tarih arasındaki commit sayısını öğrenmek:
git rev-list --count --since="2023-01-01" --until="2023-12-31" main
```

---

## 2.4 Dosya Bazlı Geçmiş Sorgulama

Belirli bir dosyanın veya dizinin tarihçesini, o dosya silinmiş olsa bile görebilirsiniz.

```bash
# Dosya ismi değişse bile (rename) tarihçeyi takip et:
git log --follow -- path/to/file.txt

# Bir dosyadaki belirli satır aralığının (L10-L20) tarihçesini incelemek:
git log -L 10,20:path/to/file.txt
```

---

## 2.5 Reflog (Görünmez Tarihçe)

Hangi commit'e, ne zaman ve hangi işlemle gittiğinizi (HEAD'in hareketlerini) gösteren sistem logudur.

```bash
git reflog
```

*Reset veya Rebase ile "silinen" commitlerinizi kurtarmak için buradaki SHA'ları kullanabilirsiniz.*

---

[← Geri: Modül 11](11_Git_Mimarisi_ve_Ic_Yapi.md) | [İleri: Modül 13 - Stratejik Dallanma ve Rerere →](13_Stratejik_Dallanma_ve_Rerere.md)

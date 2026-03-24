# Modül 05: Gelişmiş Git Komutları

Git'i usta seviyesinde kullanmak, tarihçeyi gerektiğinde değiştirmeyi (rewrite history) ve büyük repolarda ince ayar komutlarını ustalıkla kullanmayı gerektirir. 

## 1. Rebase: Temiz Bir Geçmiş Oluşturmak

`Merge`, iki branch'i bir araya getirirken yeni bir birleştirme commit'i atar. `Rebase` ise sizin yazdığınız commit'leri alır, hedef branch'in (örneğin main) en uç noktasına "sanki daha yeni yazılmışsınız gibi" baştan yazar.

**Faydası:** Karmaşık takım projelerinde doğrusal (linear) ve çok temiz bir Git geçmişi yaratır.

```bash
# feature branch'indeyken main'deki yenilikleri kodunun altına/temeline almak:
git rebase main
```
> [!WARNING]
> Kural: **Asla public (herkese açık) branch'ler üzerinde Rebase yapmayın!** Rebase commit'lerin Hash (SHA) id'lerini baştan yaratır. Remote deponuza başkalarının çektiği commit'leri rebase yaparsanız tarihçe inanılmaz bir kaosa sürüklenir. Rebase daima kendi private/lokal branch'iniz üzerinde pushlamadan önce yapılmalıdır.

### Interactive Rebase (Commit Düzenleme)
Son attığınız "N" adet commit'i birleştirmek (squash), mesajını değiştirmek veya silmek için Interactive Rebase harikadır.

```bash
git rebase -i HEAD~3
```
Bu komut seçtiğiniz editörü açarak size seçenekler sunar (`pick`, `reword`, `edit`, `squash`, `drop`). Çok sayıda gereksiz "wip", "test", "fix typo" commit'lerini kurumsal projeye göndermeden önce `squash` (ezmek/sıkıştırmak) çok profesyonel bir davranıştır.

## 2. Cherry-pick: Cımbızla Commit Çekmek

Bazen yan dalda 20 commit atılmıştır fakat acil olarak size sadece 4. commit (örneğin bir hotfix) main dalda lazımdır. Dalı merge etmek istemiyorsanız sadece o tekil commit'in hash (SHA) değerini alıp kendi dalınıza uygulayabilirsiniz.

```bash
# İlgili commit'in sadece SHA başını (örneğin 5f3a1d) alıp uygulamak:
git cherry-pick 5f3a1d
```

Eğer konflikt çıkarsa çözüp `git cherry-pick --continue` derseniz işlemi bitirmiş olursunuz.

## 3. Bisect: Hatayı (Bug) Otomatik Bulmak

Kodunuz sorunsuz çalışan sürüm v1.0'dan bug dolu mevcut sürüme (v1.5) geçmiştir ve arada 200 commit atılmıştır. Hataya hangi commit'te sebep olunduğunu "İkili Arama" (Binary Search) algoritması ile şıp diye bulabilirsiniz.

```bash
git bisect start          # Otomasyonu başlatır
git bisect bad            # Mevcut commit'in kötü olduğunu Git'e söyler
git bisect good v1.0      # Çalıştığını bildiğiniz eski komutu veya etiketi işaretler
```

Git, aradaki 200 commiti ikiye böler, 100. commite atlar ve "Şu an test et ve bana söyle" der. 
Eğer uygulama çalışıyorsa `git bisect good`, hala bug varsa `git bisect bad` yazarsınız. Git bu şekilde max 7-8 denemede o kodu kıran tekil commit'i yüzünüze vurur.

```bash
git bisect reset  # İşlemin onarımı bitince, ilk başladığınız noktaya geri döner
```

## 4. Amend: Son Commit'i Sessizce Değiştirmek

Bir commit attıktan hemen sonra "Ah, şu dosyayı da unutmuşum" veya "Yazım hatası yaptım" derseniz yeni bir commit (örn: "forgotten file") atmak amatörcedir.

Dosyayı stash/staging alanına alın ve:
```bash
git commit --amend --no-edit
# veya mesajı değiştirmek isterseniz:
git commit --amend -m "Yeni Daha Güzel Bir Commit Mesajı"
```
*(Bu komut da tarihçeyi değiştirir, sunucuya çoktan yolladığınız commitlere `--amend` yaparken dikkatli olmalısınız.)*

---

[← Geri: Modül 04](04_Uzak_Depolar_ve_Isbirligi.md) | [İleri: Modül 06 - Hata Ayıklama ve Geri Alma →](06_Hata_Ayiklama_ve_Geri_Alma.md)

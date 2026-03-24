# Modül 16: İleri Seviye Geri Alma ve Kurtarma

Hata yaptığınızda paniğe kapılmayın. Git'in "Data Integrity" (Veri Bütünlüğü) yapısı sayesinden, objeler veritabanından hemen silinmez.

---

## 6.1 Reflog ile Derinlemesine Kurtarma

`git log` sadece aktif dalın commit tarihçesini gösterirken, `git reflog` o bilgisayardaki tüm başlık (HEAD) hareketlerini tutar.

**Senaryo:** `git reset --hard` yaptınız ve 3 saatlik işiniz uçtu.

1. `git reflog` yazın.
2. İşlerin uçmadan önceki halini (`HEAD@{1}` gibi) bulun.
3. `git reset --hard HEAD@{1}` ile her şeyi geri getirin.

> [!TIP]
> Reflog'da zaman temelli arama yapabilirsiniz:
>
> `git log -g --since="1 hour ago"`

---

## 6.2 Partial Restore: Parça Parça Geri Alma (-p)

Bir dosyada 5 farklı değişiklik yaptınız ama sadece 2 tanesini geri almak, 3 tanesini tutmak istiyorsunuz.

```bash
git restore -p index.html
```

Git size her bir "hunk" (kod parçası) için soracaktır: "Geri alayım mı? (y/n)".

---

## 6.3 Revert --no-commit

Bazen bir dizi hatalı commit'i geri almanız gerekir ama her biri için ayrı ayrı "Revert" mesajı oluşturmak istemezsiniz. Tüm geri almaları staging'e toplayıp tek bir commit atmak için:

```bash
git revert -n [HASH1..HASH3]
```

Her şey staging'e gelir, şimdi tek bir mesajla bitirin:

```bash
git commit -m "fix: Hatalı modül geri sarıldı"
```

---

## 6.4 Öksüzleri Kurtarma: fsck

Eğer bir commit referansı (`branch/tag`) kalmadıysa ve reflog süresi dolduysa, o commit "Dangling" (Sallantıda/Öksüz) hale gelir.

```bash
# Sahipsiz (unreachable) objeleri bul:
git fsck --full --unreachable

# Bu objeleri bir klasöre çıkartıp incelemek isterseniz:
git fsck --lost-found
```

*(Bulunan dosyalar `.git/lost-found/commit/` altına çıkarılır.)*

---

## 6.5 Stash: İsimlendirilmiş ve Belirli Dosyalı

Sadece `git stash` yazmak yerine, onları isimlendirip yönetmek daha profesyoneldir.

```bash
# Mesajlı stash:
git stash push -m "yarım kalan login işi"

# Sadece belirli dosyaları stash'le:
git stash push path/to/file.js

# Stash'ten bir parçayı (patch) geri getir:
git stash apply --index stash@{0}
```

---

[← Geri: Modül 15](15_Tarihceyi_Ustaca_Yeniden_Yazma.md) | [İleri: Modül 17 - Profesyonel Konfigürasyon →](17_Profesyonel_Konfigurasyon.md)

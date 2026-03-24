# Modül 15: Tarihçeyi Ustaca Yeniden Yazma

Git'te geçmişi değiştirmek tehlikelidir ama doğru yapıldığında projenin okunabilirliğini ve temizliğini mucizevi şekilde artırır. Bu modülde, standart `rebase`'in ötesine geçeceğiz.

---

## 5.1 Rebase --onto (Cerrahi Müdahale)

Bazen bir özelliği (feature) başka bir özellik dalının üzerinden geliştirirsiniz ama sonra o dalı atlayıp doğrudan `main`'e bağlamanız gerekir.

```text
A---B---C (main)
     \
      D---E (feature-1)
           \
            F---G (feature-2)
```

Burada `feature-2`'yi (F, G), `feature-1`'den koparıp doğrudan `main` (C) üzerine taşımak için:

```bash
git rebase --onto main feature-1 feature-2
```

*Sonuç: F ve G artık B'den değil, C'den türemiş olur.*

---

## 5.2 Interactive Rebase: Projeyi Temizleme

Push atmadan önce yaptığınız "gereksiz" commitleri birleştirmek, silmek veya sıralamasını değiştirmek profesyonelliğin gereğidir.

```bash
git rebase -i HEAD~5
```

### Komutlar

* **pick:** Commiti olduğu gibi tut.
* **reword:** Sadece mesajını değiştir.
* **edit:** Commitin içeriğini değiştir (amend).
* **squash:** Commiti bir öncekine yedir (mesajları birleştir).
* **fixup:** Commiti bir öncekine yedir (mesajı çöpe at).

> [!WARNING]
> **Altın Kural:** Shared/Public (başkalarının çektiği) branch'lerde ASLA rebase yapmayın. Bu her bir commit'in SHA'sını değiştirir ve takım arkadaşlarınızın kod tabanını bozar.

---

## 5.3 Range-diff: Commitler Arasındaki Farkı Görmek

Bir `rebase` yaptıktan sonra, yeni commitlerin eskileriyle aynı "içeriğe" sahip olup olmadığını nasıl doğrularsınız? `git diff` iki nokta arasındaki farkı gösterir, `range-diff` ise iki commit serisi/kümesi arasındaki farkı gösterir.

```bash
git range-diff feature-v1...feature-v2
```

---

## 5.4 filter-branch Artık Eskidi: git-filter-repo

Eskiden tüm tarihçeden bir dosyayı (örn. yanlışlıkla commitlenen 1GB'*) silmek için `filter-branch` kullanılırdı. Artık çok daha hızlı ve güvenli olan **`git-filter-repo`** aracı tavsiye edilmektedir.

---

## 5.5 Cherry-pick ile Çakışma Yönetimi

Bir seriyi cherry-pick yaparken çakışma olursa:

```bash
git cherry-pick --continue # Çözdükten sonra devam et
git cherry-pick --skip     # Bu commit'i es geç
git cherry-pick --abort    # Her şeyi iptal et, başa dön
```

---

[← Geri: Modül 14](14_Guvenli_Isbirligi_ve_Kimlik.md) | [İleri: Modül 16 - İleri Seviye Geri Alma ve Kurtarma →](16_Ileri_Seviye_Geri_Alma_ve_Kurtarma.md)

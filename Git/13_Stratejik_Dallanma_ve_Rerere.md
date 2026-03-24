# Modül 13: Stratejik Dallanma ve Rerere

Git'te dal birleştirmek (merge) her zaman sorunsuz geçmez. Bu modülde, karmaşık birleştirme stratejilerini ve Git'in "çakışmaları hatırlama" (Rerere) özelliğini keşfedeceğiz.

---

## 3.1 Birleştirme Stratejileri (Merge Strategies)

`git merge` komutu arka planda farklı algoritmalar kullanabilir. Varsayılan olarak `recursive` (veya yeni Git sürümlerinde `ort`) kullanılır.

### Manuel Strateji Seçimi

* **Ours (Bizimki):** Eğer bir çakışma olursa, tereddütsüz "bizim" daldaki değişikliği kabul et, karşı tarafı reddet.

  ```bash
  git merge -s ours feature-branch
  ```

* **Theirs (Onlarınki):** Tam tersi; karşı daldaki değişikliği tek geçerli kabul et.

  ```bash
  git merge -X theirs feature-branch
  ```

* **Squash:** Karşı daldaki 50 tane commit'i tek bir tertemiz commit'e indirgeyerek ana dala ekle. Geçmişin kalabalıklaşmasını önler.

  ```bash
  git merge --squash feature-branch
  ```

---

## 3.2 Rerere: Reuse Recorded Resolution (Kayıtlı Çözümü Yeniden Kullan)

Uzun soluklu bir `feature` dalında çalışıyorsunuz ve sürekli `main` dalından güncellemeler (merge/rebase) alıyorsunuz. Her seferinde aynı dosyada aynı çakışmayı (conflict) manuel çözmekten sıkıldınız mı?

**Rerere'yi Aktifleştirin:**

```bash
git config --global rerere.enabled true
```

### Rerere Nasıl Çalışır?

1. Bir çakışmayı çözdüğünüzde Git bu çözümü `.git/rr-cache` içine kaydeder.
2. Gelecekte aynı iki dosya parçası arasında bir çakışma olursa, Git sizin eski çözümünüzü hatırlar ve otomatik olarak uygular.
3. Size sadece `git add` yapmak kalır.

---

## 3.3 Octopus Merge (Ahtapot Birleştirme)

İkiden fazla dalı aynı anda birleştirmek için kullanılır. Genellikle 4-5 farklı özellik dalını birleştirip bir `release` oluştururken tercih edilir.

```bash
git merge feature-a feature-b feature-c
```

*Eğer karmaşık çakışmalar varsa Octopus Merge başarısız olur; dalları tek tek birleştirmeniz gerekir.*

---

## 3.4 Çakışma Halinde Dosya Karşılaştırma (--ours, --theirs)

Bir çakışma anında, dosyanın her iki halini de ayrı ayrı dışarı çıkartıp incelemek isteyebilirsiniz:

```bash
# Çakışan dosyanın "bizim" halini göster:
git show :2:index.html

# Çakışan dosyanın "onların" (karşı taraf) halini göster:
git show :3:index.html
```

*(Burada `index` numaraları: 1=Ortak Ata, 2=Bizim, 3=Onlar)*

---

[← Geri: Modül 12](12_Gelismis_Log_ve_Veri_Arama.md) | [İleri: Modül 14 - Güvenli İşbirliği ve Kimlik →](14_Guvenli_Isbirligi_ve_Kimlik.md)

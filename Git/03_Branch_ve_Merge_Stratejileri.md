# Modül 03: Branch ve Merge Stratejileri

Dallanma (Branching), Git'in en güçlü özelliklerinden biridir. Main projeyi bozmadan yan dallar açmayı, yeni özellikler denemeyi ve bağımsız bağlamlarda çalışmayı olanaklı kılar.

## 1. Branch Nedir ve Neden Kullanılır?

Bir `branch` (dal), Git repository'nizdeki commit'lerin ilerleme yolunu işaretleyen hareketli bir etikettir (pointer). Varsayılan branch adınız `main` (eski adıyla `master`)'dır. Takım çalışmalarında yeni bir özellik eklerken asla ortak main dala direkt commit atılmaz; yeni bir özellik dalı (feature branch) açılır.

### Dal Oluşturma ve Listeleme

Mevcut dalları listelemek (aktif dal yeşil renkli ve asteriks (*) ile başlar):
```bash
git branch
```

Yeni bir dal oluşturmak:
```bash
git branch feature-login
```

## 2. Dallar Arasında Geçiş Yapmak

Oluşturduğunuz dala geçiş yapmak için `checkout` veya yeni Git versiyonlarında sunulan (daha okunaklı) `switch` komutu kullanılır.

> [!TIP]
> Eski komut `checkout` hem dal değiştirmek hem de dosyaları geri almak için kullanıldığından karışıklık yaratıyordu. Git >= 2.23 sonrası `git switch` (dal değiştirmek) ve `git restore` (dosya düzeltmek) hayatımıza girdi.

```bash
# Klasik Yöntem:
git checkout feature-login

# Modern Yöntem:
git switch feature-login
```

**Hem Dala Geçip Hem de Yenisini Oluşturmak:**
Tek satırda hem branch yaratıp hem de o branch'e atlamak kodlarken sık kullanılır.

```bash
git switch -c feature-shopping-cart
# veya eski yöntemle:
git checkout -b feature-shopping-cart
```

> [!IMPORTANT]
> Yeni bir dal açtığınızda, o an üzerinde bulunduğunuz commit'in/dalın kopyası üzerinden ayrılırsınız. Main daldan ayrılmak istiyorsanız, branch oluşturmadan önce `git switch main` komutu ile önce main'e geçtiğinizden emin olun.

## 3. Değişiklikleri Birleştirmek (Merge)

Kendi dalınızda (örneğin `feature-login`) işinizi bitirdikten sonra bunu ana koda (`main`) aktarmanız gerekir. Bunun işlemine "Merge" (Birleştirme) denir.

Merge işleminde temel kural şudur: **"Değişikliklerin geleceği dala (hedefi) geçip pull veya merge yapılmalıdır."**

```bash
# 1. Ana dala geçin:
git switch main

# 2. Main daldan feature-login'i kendinize çekin:
git merge feature-login
```

### Fast-forward Merge

Eğer siz `feature-login` dalında çalışırken `main` dalında hiçbir yeni commit yapılmadıysa, Git sadece "pointer" (işaretçi) ı ileri sarar. Buna *Fast-forward Merge* denir. Komplike bir birleştirme durumu oluşmaz.

### 3-Way Merge

Ortak bir atadan (A) iki dal ayrılmış olsun (`main` B, C'ye; `feature` ise D, E'ye gitmiş). Bu iki yolu birleştirirken Git 3 farklı referans noktasına bakar: Ortak ata (A), main sonu (C) ve feature sonu (E). Bu durumda Git otomatik bir **Merge Commit** ("Merge branch 'feature-login'") oluşturarak dalları bir araya getirir.

## 4. Conflict (Çakışma) Çözümü

Aynı dosyanın aynı satırı iki farklı branch'te üzerinde düzenlendiyse, `merge` sırasında Git otomatik karar veremez. Bu duruma "Conflict" denir. Terminalde `CONFLICT (content): Merge conflict in index.html` veya `Automatic merge failed` ifadesini görürsünüz.

### Conflict Nasıl Çözülür?
1. **Dosyayı Açın:** Çakışan dosyayı (örn: `index.html`) açtığınızda Git, spesifik işaretler bırakmıştır.
   ```html
   <<<<<<< HEAD
   <h1>Ana sayfaya hoş geldiniz</h1>
   =======
   <h1>Kullanıcı paneli</h1>
   >>>>>>> feature-login
   ```
2. **Düzenleyin:** `=======` çizgisi iki seçeneği ayırır. Birine, diğerine, veya ikisinin karışımına karar verin.
   *İşaretleri (`<<<`, `===`, `>>>`) mutlaka silin.*
3. **Commit ile Bitirin:**
   Çözümleri yaptıktan sonra Staging'e ekleyin ve işlemi sonlandırın.
   ```bash
   git add index.html
   git commit -m "fix: Merge conflict resolved for header text"
   ```

> [!TIP]
> Çakışmalardan kaçınmanın en iyi yolu: Başkalarının çalıştığı aynı dosyalar üzerinde eş-zamanlı geliştirmeler yapmaktan kaçınmak ve ana repository'den sık sık güncellemeleri kendi branch'inize aktarmaktır (küçük çaplı proaktif senkronlar).

## 5. Dal Silmek

İşi biten ve başarıyla merge edilmiş bir özellik dalını temiz tutmak adına silmeniz tavsiye edilir.

```bash
# Güvenli silme (Eğer branch merge edildiyse siler)
git branch -d feature-login

# Zorla silme (Merge edilmemiş değişiklikleri kaybetmeyi göze alarak silmek)
git branch -D feature-login
```

---

[← Geri: Modül 02](02_Temel_Komutlar_ve_Calisma_Akisi.md) | [İleri: Modül 04 - Uzak Depolar ve İşbirliği →](04_Uzak_Depolar_ve_Isbirligi.md)

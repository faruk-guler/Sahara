# Modül 18: Dev Projelerde Çalışma Teknikleri

Projeniz büyüdükçe (milyonlarca satır kod, binlerce dosya), standart Git işlemleri hantallaşmaya başlar. Bu modülde, dev ölçekli projelerde (monorepolarda) hayatta kalma tekniklerini öğreneceğiz.

---

## 8.1 Git Worktree (Aynı Anda Birden Fazla Dal)

Aynı projenin `main` dalında kod yazarken aniden gelen bir "hotfix" isteği için `stash` yapmakla vakit kaybetmeyin. `Worktree` ile aynı depoya ait farklı bir klasörde farklı bir dalı açabilirsiniz.

```bash
# feature-x dalını "v2-testing" klasöründe aç:
git worktree add ../v2-testing feature-x
```

*Artık yan dizinde bir `v2-testing` klasörünüz var. Terminalde oraya geçip testlerinizi yaparken, orijinal klasörünüzde `main` dalında çalışmaya devam edebilirsiniz.*

### Listeleme ve Temizlik

```bash
git worktree list
git worktree remove ../v2-testing
```

---

## 8.2 Sparse-checkout (Sadece İhtiyacın Olanı İndir)

Devasa bir mono-repoda çalışıyorsunuz ama size sadece `frontend/` klasörü lazım. Diğer 20GB'lık `backend/` ve `docs/` klasörlerini diskinize indirmek zorunda değilsiniz.

```bash
# Mevcut repoda sparse-checkout aktifleştir:
git sparse-checkout init --cone

# Sadece frontend klasörünü çek:
git sparse-checkout set frontend/src
```

*Dizinde sadece istediğiniz klasörler görünecektir; ancak tüm tarihçeniz hala güvendedir.*

---

## 8.3 Git LFS: Büyük Dosya Saklama

Oyun motoru modelleri, 4K videolar veya ağır grafik dosyaları için kullanılır (Bkz: Modül 08'in detaylı versiyonu).

```bash
git lfs install
git lfs track "*.psd"
```

---

## 8.4 Shallow Clone (--depth)

Eğer projenin 10 yıllık tarihçesiyle değil, sadece "en son haliyle" ilgileniyorsanız (örneğin bir CI sunucusunda sadece build alacaksanız):

```bash
# Sadece son 1 commit'i ve güncel dosyaları çek:
git clone --depth 1 https://github.com/torvalds/linux.git
```

*Bu, Linux çekirdeği gibi dev projelerde saatlerce sürecek clone işlemini saniyelere indirir.*

---

## 8.5 Scalar (Microsoft'un Git Hızlandırıcısı)

Microsoft'un 300GB'lık Windows repo'sunu yönetmek için geliştirdiği, Git'in üzerine eklenen bir katmandır. `background maintenance` ve `virtual filesystem` gibi özellikleri otomatik yapılandırır.

---

[← Geri: Modül 17](17_Profesyonel_Konfigurasyon.md) | [İleri: Modül 19 - CI/CD Entegrasyonu ve Otomasyon →](19_CI_CD_Entegrasyonu_ve_Otomasyon.md)

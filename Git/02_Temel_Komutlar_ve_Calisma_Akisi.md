# Modül 02: Temel Komutlar ve Çalışma Akışı

Git deposu oluşturma, dosyaları takip etme ve anlık görüntüleri (commit) kaydetme süreçleri, Git kullanmanın temel ve günlük iş akışını oluşturur.

## 1. Depo (Repository) Oluşturma

Bir klasörü Git kontrolüne almak için klasörün içine terminal üzerinden girip `init` komutu çalıştırılır.

```bash
mkdir proje_klasoru
cd proje_klasoru
git init
```

Bu komut proje ana dizininin içerisine gizli bir `.git` klasörü yaratır. **Tüm Git tarihçesi bu klasördedir.**

> [!IMPORTANT]
> Proje klasörünü silmeden, sadece projeyi Git'ten çıkartmak (tarihçeyi silmek) isterseniz gizli `.git` klasörünü silmeniz yeterlidir. (`rm -rf .git`)

## 2. Durum Kontrolü: `git status`

Git ile çalışırken "Hangi dosya değişti?", "Hangi dosya Staging Area'da?" gibi soruların cevabı daima `git status` komutundadır. Ciddi bir işlem (`commit` veya `checkout` vb.) yapmadan önce status komutunu çalıştırmayı refleks haline getirmek gerekir.

```bash
git status
```

Git'teki dosyalar iki temel durumda bulunur:
- **Untracked (Takip edilmeyen):** Git'in henüz haberdar olmadığı, yeni oluşturulmuş dosyalar.
- **Tracked (Takip edilen):** Git'in bildiği, önceki commit'lerde var olan dosyalar. Bunlar da kendi içinde *Unmodified* (Değişmemiş), *Modified* (Değiştirilmiş) veya *Staged* (Sıraya alınmış) durumlarda olabilir.

## 3. Dosyaları Hazırlık Evresine Almak: `git add`

Değişikliklerinizi veya yeni dosyalarınızı Staging Area'ya (hazırlık evresi) taşımak için `git add` kullanılır.

**Sadece belirli bir dosyayı eklemek:**
```bash
git add index.html
```

**Değişen ve yeni eklenen tüm dosyaları eklemek:**
```bash
git add .
# veya
git add -A
```

> [!TIP]
> Sadece belli dosyaları değil, bir dosyanın içindeki belirli satır değişikliklerini parça parça eklemek (interactive staging) için `git add -p` komutu kurumsal projelerde kod kalitesini artırmak için sıkça kullanılır.

## 4. Değişiklikleri Kalıcı Hale Getirmek: `git commit`

Staging Area'ya alınan tüm dosyaları bir anlık görüntü (snapshot) olarak tarihe kaydetmek için `commit` işlemi yapılır. İyi bir commit mesajı, kodu inceleyecek diğer takım arkadaşları veya ilerideki kendiniz için referanstır.

```bash
git commit -m "feat: login ekranı eklendi ve buton renkleri ayarlandı."
```

**Git Add ve Commit'i Birlikte Kullanmak:**
Sadece daha önce takip edilen (*tracked*) dosyaların değişikliklerini staging alanına çekip commit'i tek satırda atmak için `-a` flag'i çok kullanışlıdır:

```bash
git commit -am "fix: veritabanı bağlantısındaki timeout süresi güncellendi"
```
*(Yeni yaratılan dosyalar için `-a` flag'i bir işe yaramaz, onlar mutlaka önce `git add .` ile tanıtılmalıdır.)*

## 5. Değişiklikleri Gitmek (Diff Görüntüleme)

Hangi satırlarda değişiklik yaptığınızı commit öncesi incelemek için `diff` aracı harikadır.

- **Staging'e atılmamış (Sadece Working Directory'deki) değişiklikleri görmek:**
  ```bash
  git diff
  ```

- **Staging'e atılmış (commit'i bekleyen) değişiklikleri görmek:**
  ```bash
  git diff --staged
  ```

> [!NOTE]
> `git status` size hangi dosyanın değiştiğini, `git diff` ise dosyanın içindeki spesifik kod satırlarına kadar (`+` ve `-` ile) değişimi gösterir.

## 6. Proje Geçmişine Bakmak: `git log`

Atılan commit'lerin listesini ve ayrıntılarını görmek için `git log` kullanılır.

```bash
git log
```

Logları daha kompakt görmek (her commit bir satır):
```bash
git log --oneline
```

Hangi dosyalarda değişiklik yapıldığının minik bir özetini görmek:
```bash
git log --stat
```

Son n commit loguna bakmak (örn: son 3 commit):
```bash
git log -3
```

---

[← Geri: Modül 01](01_Kurulum_ve_Temel_Kavramlar.md) | [İleri: Modül 03 - Branch ve Merge Stratejileri →](03_Branch_ve_Merge_Stratejileri.md)

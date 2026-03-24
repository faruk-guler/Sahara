# Modül 06: Hata Ayıklama ve Geri Alma

Hata yapmak yazılım geliştirmenin doğasında vardır. Git, paniklememeniz için size zaman makinesi işlevi gören sayısız araç sunar. Yeter ki ne istediğinizi bilin.

## 1. Çalışma Ağacındaki (Working Directory) Hataları Silmek

Kodunuzda değişiklikler yaptınız ama her şeyi bozdunuz, commit falan da atmadınız. En son commitlenen, o "tertemiz" hale dönmek istiyorsunuz.

**Tek bir dosyayı geri sarmak:**
```bash
git restore index.html
# Veya daha eski (Git 2.23 öncesi) kullanım:
git checkout -- index.html
```

**Tüm dizini (Workspace) tamamen sıfırlamak:**
```bash
git restore .
```
> [!CAUTION]
> Git takip etmediği (Untracked) dosyaları silemeyebilir. Eğer proje ana dizininde bilmediğiniz klasör veya yeni eklenmiş .tmp dosyalarını da yok etmek isterseniz (Staging'e alınmamışsa) `git clean -fd` komutu ile zorla silebilirsiniz. Ancak unutmayın, Git'e hiç eklenmemiş dosya bu işlemle silinirse sonsuza dek kaybolur!

## 2. Staging Area (Hazırlık Evresi) Hatalarını Geri Almak

Yanlışlıkla `git add .` komutunu kullandınız ve her şeyi commit etmeye hazırladınız. Ama bazı dosyaların bu commit'te olmaması gerekiyor. Commit atmadan önce indeks'ten (Staging Area) çıkartmak için:

```bash
git restore --staged sirlar_dosyasi.txt
# Veya
git reset HEAD sirlar_dosyasi.txt
```
*(Bu komut kodunuzu diskten silmez, sadece staging klasöründen çıkarıp "modified" statüsüne geri indirir.)*

## 3. Commit'leri Geri Almak: Reset vs Revert

Git'te commit edilmiş değişiklikleri geri almanın temel olarak 2 felsefesi vardır. Biri yıkıcı ve kendi lokaliniz için (Reset), diğeri yapıcı ve herkese açık dallar için (Revert).

### `git reset` (Zamanı Geçmişte Yeniden Yazmak)
Reset, tarihçeyi bir önceki commit'e doğrudan atlatır ve siler.

- **`--soft`**: Commit kaybolur, ancak o commit'teki kodlarınız **Staging Area'ya** düşer. Sadece commit mesajını toparlamak için idealdır.
- **`--mixed` (Varsayılan)**: Commit kaybolur, Staging boşalır ama kodlarınız diskinizde hala durur. Kodu baştan add ve commit edeceksiniz.
- **`--hard`**: Commit Kaybolur. Kodlar ve değişiklikler diskten tamamen uçar (sanki o kodlar hiç yazılmamış gibi).

```bash
git reset --hard HEAD~1   # Son 1 commiti kalıcı olarak yok et
```

### `git revert` (Zıt Commit Atmak)
Hatalı kodu uzak sunucuya (`origin`) çoktan pushladınız. Artık `reset` kullanamazsınız çünkü takım arkadaşlarınız o bozuk commit'i indirdi. Bunun yerine hatalı commit'in eksiğini/tam tersini yapan yeni bir commit yaratmanız gerekir. Buna **Revert** denir.

```bash
git revert d8f23b
```
Git otomatik olarak `- Revert: Orijinal Commit Mesajı` isminde, o satırları tekrar eski haline çeviren bir "Anti-Commit" çıkarıp tarihçeye asar. Bu takım ile çalışırken tek güvenli yoldur.

## 4. Hayat Kurtaran: Reflog (`git reflog`)

Sihirli komut budur.
Panikle `git reset --hard` attınız ve binlerce satır kodu yok ettiniz. Git, varsayılan logların (`git log`) silindiği yerde "Arka Planda Ne Olduğunun Gizli Silinmez Günlüğünü" tutar!

```bash
git reflog
```
Çıktı size son 30-90 gündeki "işaretçi"nin (HEAD) tüm hareketlerini (her türlü pull, rebase, reset hamlelerinizi) gösterir. Kaybolan commit'in SHA'sını oradan bulursanız hemen o kimlikle diriltebilirsiniz:

```bash
# Sildiğimi sandığım 84dc2k'yi geri kurtar
git reset --hard 84dc2k
```

## 5. Fikri Şimdilik Kenara Koymak: Stash

Bir feature dalında yarım yamalak bir iş yapıyorsunuz. Aniden boss aradı, "Müşteri de acil hotfix lazım, main branch'e geç onu çöz bana" dedi. Dal değiştireceksiniz ama Git diyor ki "Değişikliği commit'le yoksa sileceğim". Yarım yamalak iş commitlenmez!

Çözüm: `git stash`. Tüm modifiye işlerinizi alır, koda "gizli bir rafa" kaldırıp çalışma masasını tertemiz yapar. 

```bash
git stash             # Kodları rafa kaldırır ve temizler. (Dala geçip hotfix halledilir)
git stash pop         # Geri döndüğünüzde raftaki kodu geri çalışma ekranına döker
```

Daha fazla seçenek:
```bash
git stash list        # Rafta bekleyen tüm işleri gösterir
git stash apply       # Raftan indirir ama rafta yedeği kalmaya devam eder (pop ise indirir ve siler)
git stash clear       # Raftaki her şeyi çöpe atar
```

---

[← Geri: Modül 05](05_Gelismis_Git_Komutlari.md) | [İleri: Modül 07 - Git Hook'ları ve Otomasyon →](07_Git_Hooklari_ve_Otomasyon.md)

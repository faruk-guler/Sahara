# Modül 11: Git Mimarisi ve İç Yapı

Git, basit bir dosya takipçisi değil, içerik-adresli (content-addressable) bir veri deposudur. Bu modülde Git'in kaputunun altındaki nesne tabanlı yapıyı ve hashing mekanizmasını inceleyeceğiz.

---

## 1.1 Git Object Database (Nesne Veritabanı)

Her şey `.git/objects` dizininde saklanır. Git, her dosyayı ve her değişikliği bir **SHA-1 Hash** değeri (40 karakterlik HEX dizesi) ile tanımlar.

### 4 Ana Nesne Türü

1. **Blob (Binary Large Object):** Dosyanın içeriğini saklar. Dosya adını veya izinlerini tutmaz; sadece saf veri!
2. **Tree (Ağaç):** Dizin yapısını temsil eder. İçinde `blob`'lar (dosyalar) veya başka `tree`'ler (alt dizinler) barındırır. Dosya adları ve izinleri burada tutulur.
3. **Commit:** Bir `tree` nesnesine işaret eder. Ayrıca yazar (author), tarih ve varsa parent commit (ata) bilgisini içerir.
4. **Tag:** Belirli bir commit için insan tarafından okunabilir bir etiket (v1.0 gibi) sağlar.

> [!NOTE]
> **Hashing Çakışması:** Aynı içeriğe sahip iki dosya, projenin farklı yerlerinde de olsa tek bir `blob` olarak saklanır. Git, mükemmel bir veri sıkıştırma (deduplication) yeteneğine sahiptir.

---

## 1.2 Plumbing (Tesisat) vs Porcelain (Porselen) Komutları

Git komutları ikiye ayrılır:

* **Porcelain:** Kullanıcı dostu komutlar (`git add`, `git commit`, `git log`).
* **Plumbing:** Arka planda iş yapan, düşük seviyeli teknik komutlar (`git cat-file`, `git hash-object`, `git update-index`).

### Nesneleri Röntgenleme: git cat-file

Bir commit veya blob'un içinde ne olduğunu görmek için:

```bash
# Bir nesnenin tipini öğrenmek:
git cat-file -t [hash]

# Bir nesnenin içeriğini olduğu gibi okumak:
git cat-file -p [hash]
```

---

## 1.3 DAG (Directed Acyclic Graph) Yapısı

Git tarihçesi bir ağaç değil, bir **Yönlendirilmiş Döngüsüz Graf**'tır.

* **Dallar (Branches):** Aslında sadece belirli bir commit SHA'sına işaret eden küçük metin dosyalarıdır (`.git/refs/heads/`).
* **HEAD:** O an hangi commit veya dal üzerinde olduğunuzu gösteren bir göstergedir (`.git/HEAD`).

> [!TIP]
> **Detached HEAD Nedir?**
> Eğer bir branch adı yerine doğrudan bir commit hash'ine `checkout` yaparsanız, HEAD artık bir dalı değil, sabit bir noktayı gösterir. Bu durumda yaptığınız yeni commit'lerin dalı olmayacağı için, dal değiştirince bu commitler "öksüz" kalır ve ileride GC tarafından silinebilir.

---

## 1.4 Dosya Değişim Takip Mekanizması

Git, dosyalar arasındaki farkları (diff) değil, dosyanın her halinin tam bir "Snapshot" (Anlık Görüntü) bilgisini saklar.

* Eğer dosya değişmemişse, yeni commit sadece eski dosyanın (blob) hash'ine bir pointer (işaretçi) atar.
* Bu sayede `git checkout` yapmak, dosyaları kopyalamak yerine sadece linkleri değiştirmek kadar hızlıdır.

---

[← Modül 10: İpuçları ve Performans](10_Ipuclari_Kisayollar_ve_Performans.md) | [İleri: Modül 12 - Gelişmiş Log ve Veri Arama →](12_Gelismis_Log_ve_Veri_Arama.md)

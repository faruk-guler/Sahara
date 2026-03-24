# Modül 20: Bakım, Performans ve Plumbing

Git Master eğitimimizin son ayağında, sistem seviyesindeki optimizasyonları ve düşük seviyeli (Plumbing) incelemeleri göreceğiz.

---

## 10.1 Modern Git Maintenance (Bakım)

Eski `git gc` komutunun yerini artık daha akıllı bir yapı olan `maintenance` aldı. Git, arka planda periyodik olarak veri paketlerini sıkıştırabilir ve commit grafiğini (commit-graph) önbelleğe alabilir.

```bash
# Bakım servisini bu repo için aktifleştir:
git maintenance start

# Manuel olarak bakım görevlerini tetiklemek:
git maintenance run
```

---

## 10.2 Commit-Graph Hızlandırması

Çok karmaşık dallanma yapısına sahip repolarda `git log --graph` komutu yavaş çalışır. Commit-graph dosyasını oluşturmak, bu aramayı ciddi oranda (10 kata kadar) hızlandırır.

```bash
git commit-graph write --reachable
```

---

## 10.3 Packfiles: Git Veriyi Nasıl Paketler?

Normalde Git her nesneyi ayrı bir dosya (loose object) olarak saklar. Ancak zamanla bu dosyaları birleştirip sıkıştırarak `.git/objects/pack` dizinine `.pack` ve `.idx` formatında kaydeder.

```bash
# Mevcut objeleri paketle:
git repack -a -d
```

---

## 10.4 Plumbing: git rev-parse ve git cat-file

Bazen bir branch adını veya sembolik referansı (HEAD, master~2) SHA'ya dönüştürmeniz gerekir:

```bash
# HEAD'in SHA-1 hash'ini al:
git rev-parse HEAD

# Bir commit'in yazar tarihini doğrudan terminale dök:
git cat-file -p [HASH] | grep author
```

---

## 10.5 Git Database Röntgenciliği: git verify-pack

Hangi dosyaların en çok yer kapladığını (tarihçede nelerin depoyu şişirdiğini) görmek için:

```bash
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -n | tail -10
```

*Bu komut, repo içindeki en büyük 10 nesneyi hash'leri ile birlikte listeler.*

---

## 🎖️ Tebrikler: Artık Bir Git Master'ınız

10 modül boyunca Git'in alt katmanlarından, tehlikeli tarihçe değişikliklerine, kurumsal iş akışlarından performans optimizasyonlarına kadar her şeyi inceledik.

### Hatırlamanız Gereken Üç Altın Kural

1. **Commit Erken, Commit Sık:** Ama asla bozuk kod commit etmeyin.
2. **Pull Öncesi Stash/Commit:** Çalışma alanınız her zaman temiz olsun.
3. **Reflog Sizin Dostunuzdur:** Git'te gerçek anlamda bir şeyi "kaybetmek" neredeyse imkansızdır.

---

[← Geri: Modül 19](19_CI_CD_Entegrasyonu_ve_Otomasyon.md) | [Başa Dön: Ana Sayfa](README.md)

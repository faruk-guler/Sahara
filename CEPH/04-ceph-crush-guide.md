# 🗺️ Ceph CRUSH Map Yönetimi Rehberi

CRUSH (Controlled Replication Under Scalable Hashing), Ceph'in verinin nerede depolanacağını belirleyen algoritmasıdır. Bu rehber, CRUSH haritasının anlaşılması ve özelleştirilmesini kapsar.

---

## 1. CRUSH Temelleri

### CRUSH Nedir?

CRUSH, merkezi bir metadata sunucusuna ihtiyaç duymadan verinin hangi OSD'lerde saklanacağını hesaplayan deterministik bir algoritmadır.

**Avantajları:**

- Merkezi darboğaz yok
- Sonsuz ölçeklenebilirlik
- Deterministik (aynı girdi = aynı çıktı)
- Failure domain kontrolü

### CRUSH Hiyerarşisi (Bucket Türleri)

```text
root (kök)
 └── datacenter (veri merkezi)
      └── room (oda)
           └── rack (dolap)
                └── host (sunucu)
                     └── osd (disk)
```

---

## 2. CRUSH Map Görüntüleme

### Mevcut Haritayı Görme

```bash
# CRUSH ağacını görüntüle
ceph osd tree

# CRUSH haritasını dışa aktar (binary)
ceph osd getcrushmap -o crushmap.bin

# Binary'yi okunabilir formata çevir
crushtool -d crushmap.bin -o crushmap.txt

# Haritayı görüntüle
cat crushmap.txt
```

### OSD Konumlarını Görme

```bash
ceph osd crush dump
```

---

## 3. CRUSH Map Düzenleme

### Yöntem 1: CLI ile Anlık Değişiklik

```bash
# OSD'yi farklı bir host'a taşı
ceph osd crush set osd.5 1.0 host=node3

# Yeni bucket (host) oluştur
ceph osd crush add-bucket newhost host

# Host'u rack'e bağla
ceph osd crush move newhost rack=rack1

# Rack oluştur ve root'a bağla
ceph osd crush add-bucket rack2 rack
ceph osd crush move rack2 root=default
```

### Yöntem 2: Map Dosyasını Düzenleyerek

```bash
# 1. Mevcut haritayı al
ceph osd getcrushmap -o crushmap.bin
crushtool -d crushmap.bin -o crushmap.txt

# 2. Düzenle
nano crushmap.txt

# 3. Tekrar derle ve uygula
crushtool -c crushmap.txt -o newcrushmap.bin
ceph osd setcrushmap -i newcrushmap.bin
```

---

## 4. Failure Domain (Hata Alanı)

Failure domain, Ceph'in replikaları nasıl dağıttığını belirler.

### Varsayılan Davranış

```text
Varsayılan: failure-domain = host
→ Her replika farklı bir sunucuda tutulur
→ Bir sunucu bozulursa veri kaybolmaz
```

### Failure Domain Seviyeleri

| Seviye | Koruma | Gereksinim |
| :--- | :--- | :--- |
| `osd` | Disk arızası | En az 3 OSD |
| `host` | Sunucu arızası | En az 3 sunucu |
| `rack` | Dolap arızası | En az 3 rack |
| `datacenter` | DC arızası | En az 3 veri merkezi |

### CRUSH Rule ile Failure Domain Değiştirme

```bash
# Host bazlı kural (varsayılan)
ceph osd crush rule create-replicated host-rule default host

# Rack bazlı kural
ceph osd crush rule create-replicated rack-rule default rack

# Pool'a uygula
ceph osd pool set mypool crush_rule rack-rule
```

---

## 5. OSD Sınıfları (Device Classes)

Ceph, disk türlerini otomatik algılar ve sınıflandırır.

### Sınıfları Görme

```bash
ceph osd crush class ls
# Çıktı: [hdd, ssd, nvme]

# OSD'lerin sınıflarını gör
ceph osd crush tree --show-shadow
```

### Sınıf Bazlı CRUSH Kuralı

```bash
# Sadece SSD'leri kullanan kural
ceph osd crush rule create-replicated ssd-only default host ssd

# Sadece HDD'leri kullanan kural
ceph osd crush rule create-replicated hdd-only default host hdd

# Pool'lara uygula
ceph osd pool set fast-pool crush_rule ssd-only
ceph osd pool set archive-pool crush_rule hdd-only
```

### Manuel Sınıf Atama

```bash
# OSD'nin sınıfını kaldır
ceph osd crush rm-device-class osd.5

# Yeni sınıf ata
ceph osd crush set-device-class nvme osd.5
```

---

## 6. Custom CRUSH Rules

### Kural Yapısı

```text
rule <rule-name> {
    id <unique-id>
    type replicated
    min_size <minimum-replicas>
    max_size <maximum-replicas>
    step take <bucket>
    step chooseleaf firstn <count> type <failure-domain>
    step emit
}
```

### Örnek: Rack Bazlı Kural

```bash
# CRUSH haritasını düzenle
crushtool -d crushmap.bin -o crushmap.txt
```

Dosyaya ekle:

```
rule rack-isolation {
    id 10
    type replicated
    min_size 1
    max_size 10
    step take default
    step chooseleaf firstn 0 type rack
    step emit
}
```

```bash
# Derle ve uygula
crushtool -c crushmap.txt -o newcrushmap.bin
ceph osd setcrushmap -i newcrushmap.bin
```

---

## 7. Stretched Cluster (Çoklu Veri Merkezi)

### Yapı

```text
root
├── datacenter-a
│   └── host-a1, host-a2
├── datacenter-b
│   └── host-b1, host-b2
└── datacenter-arbiter
    └── host-arbiter (sadece MON için)
```

### Stretched Mode Aktifleştirme

```bash
# Stretch mode için MON'ları yapılandır
ceph mon enable_stretch_mode arbiter-host dc datacenter-a datacenter-b
```

---

## 8. CRUSH Tunables

CRUSH versiyonu ve davranışını kontrol eder.

```bash
# Mevcut tunables'ı gör
ceph osd crush show-tunables

# Optimal profile'a güncelle (dikkatli ol!)
ceph osd crush tunables optimal
```

### Tunable Profilleri

| Profil | Ceph Versiyonu | Özellik |
| :--- | :--- | :--- |
| `legacy` | Bobtail öncesi | Eski, kötü dağılım |
| `bobtail` | Bobtail | Temel iyileştirmeler |
| `hammer` | Hammer | Daha iyi dengeleme |
| `optimal` | Luminous+ | En iyi dağılım |

---

## 9. OSD Ağırlıkları (Weights)

### CRUSH Weight

Disk kapasitesini temsil eder (TB cinsinden).

```bash
# Weight'i görüntüle
ceph osd crush tree

# Weight değiştir
ceph osd crush reweight osd.5 2.0  # 2 TB disk için
```

### OSD Reweight (Geçici Ağırlık)

OSD doluluk dengesizliğini düzeltmek için kullanılır.

```bash
# 0.0 ile 1.0 arası değer (1.0 = tam kapasite)
ceph osd reweight osd.5 0.8

# Otomatik dengeleme
ceph osd reweight-by-utilization
```

---

## 10. CRUSH Simülasyonu ve Test

### Veri Dağılımını Simüle Et

```bash
# Bir pool için veri dağılımını test et
crushtool -i crushmap.bin --test --show-mappings --rule 0 --num-rep 3
```

### Kural Değişikliğinin Etkisini Önceden Gör

```bash
# Kaç PG hareket edecek?
ceph osd getmap -o osdmap.bin
osdmaptool osdmap.bin --test-map-pgs --pool mypool
```

---

## 11. CRUSH Best Practices

### ✅ Yapılması Gerekenler

- Failure domain'i iş yüküne göre seçin
- SSD ve HDD'leri ayrı device class'larda tutun
- CRUSH değişikliklerini lab ortamında test edin
- Büyük değişiklikler öncesi `ceph osd set norebalance` kullanın

### ❌ Yapılmaması Gerekenler

- Production'da test edilmemiş CRUSH map uygulamayın
- Tüm OSD'leri aynı host'a atamayın (failure domain ihlali)
- Tunables'ı gereksiz yere değiştirmeyin
- Manual weight değişikliklerini aşırı kullanmayın

---

## 12. Yaygın CRUSH Sorunları ve Çözümleri

### Sorun: Uneven Data Distribution

```bash
# Kullanım oranlarını kontrol et
ceph osd df tree

# Otomatik reweight
ceph osd reweight-by-utilization

# Balancer modülünü etkinleştir
ceph balancer on
ceph balancer mode upmap
```

### Sorun: PG Stuck in Active+Remapped

```bash
# CRUSH kuralını kontrol et
ceph osd pool get mypool crush_rule

# Yeterli OSD var mı?
ceph osd tree
```

### Sorun: HEALTH_WARN - Failed to choose

```bash
# CRUSH rule ve failure domain uyumsuzluğu
# Host sayısı < replica sayısı olabilir
ceph osd pool set mypool size 2  # veya daha fazla host ekle
```

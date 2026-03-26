# Modül 06: Low-Level Discovery (LLD) Master Eğitimi

Zabbix'in standart metriklerinden (CPU, RAM) ayrılan en büyük gücü **Low-Level Discovery (Düşük Seviyeli Keşif - LLD)**'dir. LLD, bir sunucunun içindeki "değişken" veya "çeşitli" bileşenleri bulup, onlara otomatik olarak item, trigger ve grafik oluşturma işlemidir.

## 1. LLD Neden Hayat Kurtarır?

Bir veri tabanı sunucusunda `/`, `/var`, `/home`, ve `/data` disk bölümleri (Partition) olabilir. Başka bir sunucuda ise sadece `/` ve `/app` olabilir. Siz her sunucu için tek tek `vfs.fs.size[/data,pfree]` gibi itemlar açamazsınız.

LLD kuralı (`vfs.fs.discovery`) Zabbix Ajanı'ndan sistemdeki tüm disklerin listesini çeker, bu liste üzerinden bir döngü kurar ve dinamik makrolar çıkarır.

## 2. Prensip: JSON Data Model

Bütün LLD kurallarının altında, Zabbix'in anlayabileceği bir JSON Array (Dizi) döndürme zorunluluğu yatar:

```json
[
  {"{#FSNAME}": "/", "{#FSTYPE}": "ext4"},
  {"{#FSNAME}": "/boot", "{#FSTYPE}": "xfs"},
  {"{#FSNAME}": "/var/log", "{#FSTYPE}": "ext4"}
]
```
Zabbix bu JSON dizisini işler ve oluşturduğunuz Prototiplere uygular.

### 2.1 İtem Prototipleri (Item Prototypes)
Zabbix, bu JSON'u okur ve prototipi her bir makro için klonlar:
- **Prototip:** `vfs.fs.size[{#FSNAME},pfree]`
- **Oluşan İtem 1:** `vfs.fs.size[/,pfree]`
- **Oluşan İtem 2:** `vfs.fs.size[/boot,pfree]`

Aynı mantıkla **Trigger Prototypes** ve **Graph Prototypes** de otomatik klonlanır.

## 3. Kendi Custom LLD Betiğinizi Yazmak

Bir klasördeki Docker containerları LLD ile bulmak istediğinizi varsayalım (`/opt/custom_lld.sh`):

```bash
#!/bin/bash
echo -n '{"data":['
first=1
for i in $(docker ps -q); do
    name=$(docker inspect --format="{{.Name}}" $i | sed 's/^\///')
    if [ $first -eq 0 ]; then echo -n ","; fi
    echo -n "{\"{#CONTAINERNAME}\":\"$name\"}"
    first=0
done
echo -n ']}'
```

Bunu `zabbix_agent2.conf` içinde UserParameter olarak tanımladığınızda:
```ini
UserParameter=docker.discovery,/opt/custom_lld.sh
```
Artık Zabbix Server'da `docker.discovery` key'ine sahip bir LLD kuralı oluşturup tüm containerlara otomatik izleme yapılandırabilirsiniz.

## 4. 2026 Gelişmiş LLD Özellikleri

### 4.1 Filter (Filtreler)
"Ben sadece `ext4` dosya sistemlerini izlemek istiyorum, `tmpfs` istemiyorum."
- **Macro:** `{#FSTYPE}`
- **Regular Expression (Regex):** `^ext4$`

### 4.2 Overrides (Geçersiz Kılmalar)
"Arama yaptık, tüm diskleri bulduk. Diskin kapasitesi %90 dolduğunda Alarm (Warning) verecek genel kuralım var. Ancak `{#FSNAME}`=/backup isimli diskimde kapasite çok büyük olduğu için %95 dolduğunda (High) Alarm versin."

Zabbix 7.0 ile LLD kuralı altındaki **Overrides** bölümü bu esnekliği sunarak alarm çöplüğünün önüne geçer.

### 4.3 Preprocessing'in LLD'ye Etkisi
Zabbix 2026'da (JSON Path ile) harici bir API'den (Örn: ServiceNow API) dönen devasa bir JSON sonucunu `Preprocessing` kısmında LLD nesnelerine dönüştürebilirsiniz. Agent kullanmadan, doğrudan HTTP(s) üzerinden otomatik bir ağaç yapısı çıkartmak mümkündür.

---
[Önceki Modül](./05_Otomatik_Kesif.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Template Yönetimi](./07_Template_Yonetimi.md)

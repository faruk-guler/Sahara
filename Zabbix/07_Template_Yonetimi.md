# Modül 07: Kapsamlı Template Yönetimi ve Makrolar

Kurumsal ortamlarda 5000 sunucuya tek tek ayar yapmak imkansızdır. Zabbix'te her izleme yapılandırması (Configuration Item), **Template (Şablon)** adı verilen konteynerlerin içine yazılır ve Hostlara (Cihazlara) uygulanır (Linking).

## 1. 2026 Şablon Mimarisi: Linked Templates (Bağlantılı Şablonlar)

Zabbix 7.0 standartlarında tasarımlar monolitik (tek parça) değil, modüler yapılmalıdır. Bütün ayarları "Template Sirket Sunuculari" isimli dev bir template altına yığmak yerine, küçük parçalar oluşturulur:

- `Template OS Linux by Zabbix agent 2`
- `Template DB PostgreSQL`
- `Template App Nginx`

Bu üçünü birleştirerek "Template Web-PostgreSQL" isimli bir **Kapsayıcı (Wrapper) Template** yaratılır. Sadece bu wrapper'ı sunucuya atadığınızda tüm bağımlı template'ler de aktarılır. Ayrı ayrı her şeyi tek bir template'e koymadığınız için yönetimi çok kolaylaşır.

## 2. Makrolar (Zabbix Değişkenleri - `{$MACRO}`)

Template'lerin her sunucuya uygulanabilir olması için statik değerler (sabit rakamlar) barındırmaması gerekir. Makrolar bu esnekliği sağlar.

### Neden Makro Kullanılır?

Bir Nginx sunucunuz `8080` portunda, diğer Nginx sunucunuz `8888` portunda çalışıyorsa, tek bir Nginx Template'i yazmak için "portu" değişken yapmanız gerekir:
`net.tcp.service[http,,{$NGINX.PORT}]`

### Makro Hiyerarşisi (Sıralama Önemi)

Zabbix bir makronun değerine bakarken şu sırayı takip eder (En spesifik olan her zaman önceliklidir):

1. **Host Makrosu:** Doğrudan sunucu içerisindeki ayar (En yüksek öncelik). Örn: `{$NGINX.PORT} = 8888`
2. **Template Makrosu:** Bağlı olan template'in varsayılan değeri. Örn: `{$NGINX.PORT} = 8080`
3. **Global Makro:** Tüm Zabbix Server için geçerli olan (`Administration` -> `General` -> `Macros`). Örn: `{$SNMP_COMMUNITY} = public`

Eğer Host'a girip `{$NGINX.PORT}` tanımlamazsanız, Zabbix Template içindeki `8080` değerini alır. Tanımlarsanız o sunucu için `8888` geçerli olur ve diğer sunucular etkilenmez.

## 3. Secret ve Vault Makrolar (Gizli Bilgiler)

Veritabanı izleme için `{$POSTGRES.PASSWORD}` oluşturduğunuzda, arayüze giren "User" yetkili kişiler bu şifreyi görebilir. Zabbix güvenlik standartları bunu iki yöntemle çözer:

### 3.1. Secret Macro (Yerleşik Çözüm)

Makro tipi `Text` yerine `Secret text` olarak seçilir. Makronun değeri girilip kaydedildikten sonra bir daha **hiçkimse** (Super Admin dahil) arayüzden o değeri göremez, sadece `******` şeklinde görünür. Zabbix Server kendi arka planında bunu hash'li değerlendirir.

### 3.2. Vault Macro (CyberArk & HashiCorp) - Mimarisi Modül 20'dedir

Makro tipi `Vault secret` olarak ayarlanır. Değer şu şekilde girilir:
`secret:zabbix/database:password`
Zabbix şifreyi kendi DB'sinde asla tutmaz. İhtiyaç duyduğu saniye (Polling anında) Vault API'sine gidip şifreyi alır, komutu dener ve bellekten (RAM) anında siler. **(Master Practice)**

## 4. Kullanıcı Makroları Context (Bağlamsal Kullanım)

"Bütün disklerde %90 doluluk alarmı var, ancak /logs diskim çok büyük, onunki %95 olsun." (LLD Overrides dışında bir diğer yöntem).

`{$DISK_CRIT_THRESHOLD}` = 90
`{$DISK_CRIT_THRESHOLD:"/logs"}` = 95

Gördüğünüz gibi, tırnak işareti içerisinde (Context - Bağlam) belirterek, tek bir makronun sadece bir parametreye özel farklı davranış göstermesini ayarlayabilirsiniz. Trigger formülü ise bunu otomatik çözer: `vfs.fs.size[{#FSNAME},pused] > {$DISK_CRIT_THRESHOLD:"{#FSNAME}"}`

---
[Önceki Modül](./06_LLD.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Veri Toplama](./08_Veri_Toplama.md)

# Modül 04: Kurallar (Rules) ve Dekoderlar (Decoders)

Wazuh'un "beyni" kurallar ve dekoderlardır. Gelen karmaşık logları anlamlandırır ve alarm üretir.

## Dekoder Nedir?

Ham log metninden anlamlı alanlar (IP adresi, kullanıcı adı, hata kodu vb.) çıkaran düzenli ifadelerdir (Regex).

**Örnek Dekoder:**

```xml
<decoder name="example_app">
  <prematch>AppLogs:</prematch>
</decoder>

<decoder name="example_app_fields">
  <parent>example_app</parent>
  <regex>User (\S+) failed from (\d+.\d+.\d+.\d+)</regex>
  <order>user, srcip</order>
</decoder>
```

## Kural Nedir?

Dekoderın çıkardığı alanları inceleyen ve belirli bir risk seviyesine göre alarm üreten mantıksal yapılardır.

**Örnek Kural:**

```xml
<group name="example_group">
  <rule id="100001" level="5">
    <decoded_as>example_app</decoded_as>
    <description>Uygulama üzerinden giriş hatası algılandı.</description>
  </rule>

  <rule id="100002" level="10">
    <if_sid>100001</if_sid>
    <match>admin</match>
    <description>Kritik: Admin kullanıcısı için giriş hatası!</description>
  </rule>
  
  <!-- Frekans Tabanlı Kural: 1 dakika içinde 5 hata -->
  <rule id="100003" level="10" frequency="5" timeframe="60">
    <if_matched_sid>100001</if_matched_sid>
    <description>Aynı kullanıcıdan çok sayıda giriş hatası (Brute Force?).</description>
  </rule>
</group>
```

## Kurallar Nerede Saklanır?

- **Default Ruleset:** `/var/ossec/ruleset/rules/` (Asla burada değişiklik yapmayın).
- **Custom Rules:** `/var/ossec/etc/rules/` (Kendi kurallarınızı buraya yazın, güncellemede silinmez).

## Kural Seviyeleri (Levels)

- **0 - 2:** Bilgi amaçlı, alarm üretilmez.
- **3 - 7:** Düşük/Orta öncelikli uyarılar.
- **8 - 12:** Yüksek öncelikli, dikkat gerektiren olaylar.
- **13 - 15:** Kritik alarmler (Active Response tetikleyebilir).

## Test Aracı: `wazuh-logtest`

Yazdığınız kural veya dekoderın çalışıp çalışmadığını test etmek için kullanılır.

```bash
/var/ossec/bin/wazuh-logtest
```

(Logunuzu yapıştırın ve hangi kuralın tetiklendiğini görün.)

---

[README'ye Dön](README.md)

# Modül 08: Gelişmiş Veri Toplama Yöntemleri ve Preprocessing

Zabbix yalnızca sunuculara ajan yükleyip CPU okuyan bir sistem değildir. 2026 yılı altyapılarında "Veri her yerdedir (API, Text File, Database)" ve bu veriyi toplamak zenginleştirilmiş yöntemler gerektirir.

## 1. HTTP Agent (API'lerden Veri Çekme)

Bazen sistemlerinizi sadece sundukları HTTP Rest API'ler üzerinden izleyebilirsiniz. **HTTP agent** doğrudan Zabbix arayüzünden yapılandırılan ve proxy üzerinden de çalışabilen native bir özelliktir.

- NTLM, Kerberos, Digest veya Basic Authentication destekler.
- JSON/XML veya Plain Text çeker.
- `https://api.sirket.com/v1/health` adresine bir GET veya POST (Payload ekleyerek) isteği göndererek uygulamanızın canlılığını/verilerini okuyabilirsiniz.

## 2. JavaScript ve Script Items (Yenilik!)

Önceden API'ye gidip data çeken veya bir matematiksel fonksiyon uygulayan komut dosyalarını (Python/Bash) Zabbix Sunucusuna (`/usr/lib/zabbix/externalscripts`) komut olarak yüklemeniz ve `External Check` yapmanız gerekirdi. Bu performans problemlerine sebep oluyordu.

Artık doğrudan Zabbix arayüzündeki **Script item** yeteneği sayesinde `JavaScript` kullanarak:

```javascript
// Arayüzden girilen basit bir Zabbix JS kodu
var req = new HttpRequest();
var response = req.get('https://api.github.com/users/zabbix');
var data = JSON.parse(response);
return data.public_repos; // Sadece public repository sayısını Zabbix'e item datası olarak verir.
```

Bu yöntem C düzeyinde izole (Sandboxed) çalışır, dış (External) Python scriptlerden 10 kat daha hızlıdır ve Sunucunun CPU'sunu yormaz.

## 3. Dependent Items (Bağımlı Nesneler - Master Kuralı)

NVPS'yi (saniyedeki sorgu yükünü) %90 düşüren teknik.
Diyelim ki bir API'den gelen veride (veya bir Database sorgusunda) aynı anda 10 farklı veri var: Toplam Kullanıcı Sayısı, Aktif Siparişler, İptal Siparişler, Biten Siparişler.

**Yanlış Yöntem:** Zabbix üzerinden bu 4 veriyi almak için her item için API'ye ayrı ayrı (toplam 4 kere) istek atmak.
**Doğru Yöntem (Dependent Items):**

1. **Master Item (Ana Nesne):** Tüm veriyi içeren devasa JSON/XML çıktısını çeker ve saklar (Tipi genelde `Text`). Sadece 1 defa ağ isteği yapar.
2. **Dependent Item 1 (Kullanıcılar):** Veriyi ağdan çekmek yerine **Master Item** içinden alır. Üzerine bir `Preprocessing` kuralı (Örn: JSON Path `$.users.total`) yazar.
3. **Dependent Item 2 (Siparişler):** Yine Master Item'dan okur, `$.orders.active` diyerek kendi değerini cımbızlar.

Böylece Zabbix, sunucuyla yalnızca 1 kere iletişime girip arkaplanda 50 farklı veriyi sorunsuz işler. Ağır Database Loglarını ve REST API çıktılarını performans dostu bir şekilde işleyebilmenin tek yöntemidir.

## 4. Preprocessing (Ön İşleme)

Veri, Zabbix veritabanına kaydedilmeden **hemen önce** üzerinden geçiş (transformasyon) yapılan kritik konudur.

- **JSON Path / XML XPath:** Gelen büyük kütleden tek bir objeyi bulmak.
- **Regex:** `CPU Temperature is 45C` cümlesinden sadece `45` rakamını almak.
- **Multiplier:** Gelen byte verisi 1024 olarak geldiyse bunu 8 ile çarpıp (Bits per second - `bps`) elde etmek. (`Custom multiplier: 8`)
- **Change per second:** Ağ adaptörü (eth0) toplam gönderilen paket miktarını tutar (Sürekli artan büyük bir rakam). Eğer `Change per second` kullanırsanız, önceki ölçümle aradaki farkı hesaplar ve saniyeye böler; bu size anlık trafiği (Bant genişliğini) verir.
- **Discard unchanged with heartbeat:** (Master Teknik). "Ping durumu: Up(1)". Bu veri sürekli "1" olarak gelir. Değer değişmiyorsa saniyede bir bunu DB'ye sormanın ve yazmanın alemi yoktur. Bu seçeneği eklerseniz, Zabbix sadece değer "0" olduğunda (veri değiştiğinde) ve ara sıra (heartbeat: Örn her 1 saat) canlılık belirtsin diye DB'ye yazar. Dev boyutlarda veritabanı tasarrufu sağlar.

## 5. Browser Items (Sentetik İzleme - Zabbix 7.0+)

Modern web uygulamaları (React/Angular) API'den gelen veriyi tarayıcıda işler. Klasik Web Monitoring (cURL bazlı) bu sitelerin yüklenip yüklenmediğini anlayamaz. Zabbix 7.0 ile gelen **Browser Items**, arka planda bir Chromium tarayıcısını **Playwright** motoru ile ayağa kaldırır:

- **Ekransız Tarayıcı (Headless Browser):** Javascript kodlarını çalıştırır, siteye giriş yapar (Login), bir butona tıklar ve sepetin durumunu okuyabilir.
- Ağ sekmesini izleyerek sitenin tam yüklenme süresini (DOM Interactive time) saniye saniye grafiğe döker.
- Bu özellik, uçtan uca (End-to-End) kullanıcı deneyimini test etmek için 2026 Observability vizyonunun en değerli araçlarındandır.

---
[Önceki Modül](./07_Template_Yonetimi.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Triggerlar](./09_Triggerlar.md)

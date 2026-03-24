# 3. Gelişmiş Tehdit Analizi (Gürültü Karşıtı Şablonlar)

Ağ üzerinde yakalanan paket sayılarının yüz binleri bulduğu devasa .pcapng dosyalarında sadece tehlike potansiyeli taşıyan anormallikleri veya yeni kurulan bağlantıları elemek için bu şablonları ekranınızdaki "Display Filter" kutusuna yapıştırarak kullanabilirsiniz.

---

## 3.1 Genel ve İzole Paket Keşfi (Standart Analiz)

Telsiz/Kablosuz ağ gürültüsünü engeller; DNS, TLS el sıkışmaları, HTTP talepleri, QUIC ve büyük boyutlardaki veri yüküne (payload) odaklanır:

```text
(tcp.flags.syn == 1 && tcp.flags.ack == 0) || (dns.flags.response == 0 && dns.qry.name) || tls.handshake.extensions_server_name || http.request || quic || (udp && !(udp.port == 53 || udp.port == 123 || udp.port == 443 || udp.port == 5353 || udp.port == 1900)) || frame.len > 2000
```

## 3.2 Tek Bir Cihazı / IP'yi Mercek Altına Alan Tehdit Analizi

Verilen örnekteki (`192.168.1.43`) bilgisayarının veya telefonunun arkada ne tür bağlantılar oluşturduğunu listeler:

```text
(ip.addr == 192.168.1.43) && (tcp.flags.syn == 1 || dns.flags.response == 0 || tls.handshake.extensions_server_name || http.host || http.request || quic || frame.len > 1500)
```

## 3.3 DNS / TLS Hızlı Güvenlik Kontrolü

Ağda en sıkı haberleşmenin olduğu şifreli tokalaşmaları (SNI) ve domain çözümlemelerini (DNS) süzer:

```text
dns || http || tls.handshake.extensions_server_name || tcp.flags.syn == 1
```

---

## 3.4 Uzman Seviye Filtre Operatörleri

Gerçek "Master" seviyesindeki kullanıcılar, statik filtreler yerine dinamik operatörler kullanır.

* **Regex (Düzenli İfade) ile Arama (`matches`):**
  Belirli bir kelime kalıbını (büyük/küçük harf duyarsız) aramak için kullanılır.
  `http.host matches "(?i)google|facebook|amazon"`

* **Küme Üyeliği Üyeliği (`in`):**
  Birden fazla değeri tek seferde taramak için pratik bir yöntemdir.
  `tcp.port in {80, 443, 8080}`

* **Slicing (Dilimleme - `[n:m]`):**
  Bir paketin içeriğindeki belirli bayt (byte) aralığını kontrol etmenizi sağlar.
  `eth.addr[0:3] == 00:00:5e` (Sadece ilk 3 baytı -yani üreticiyi- kontrol eder).

[< Kablosuz Ağ Analizi >](04_Kablosuz_Ag_Analizi.md)

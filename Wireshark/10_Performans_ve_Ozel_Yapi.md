# 10. Performans Optimizasyonu ve Özel Yapılandırmalar

Ağ trafiğini kesintisiz analiz etmek, küresel seviyeden verileri haritalamak ve bilinmeyen protokolleri çözümlemek; bir Wireshark operatörünü "Kullanıcı"dan "Master" seviyesine çıkaran özelliklerdir.

---

## 10.1 Ring Buffer (Kesintisiz Yakalama Modu)

Uzun soluklu analizlerde (Örneğin; "Sistem 3 gün boyunca rastgele anlarda çöküyor" gibi sorunlarda) Wireshark'ı açık bırakmak bilgisayarın RAM'ini (belleğini) patlatacaktır. Bunun yerine, disk alanını belirli bir sınırda tutan ve eski dosyaları üstüne yazarak ilerleyen **Ring Buffer (Halka Tampon)** yöntemi kullanılır.

**TShark ile Komut Satırından Kesintisiz Yakalama:**
Aşağıdaki komut; 50 MB boyutunda en fazla 10 dosya (`-b files:10`) oluşturarak kayda girer. 11. dosya yazılmaya başlandığında otomatik olarak en eski (1.) dosyayı siler. Toplamda sadece max 500 MB disk kapasitesi harcar:

```bash
tshark -i 1 -b filesize:50000 -b files:10 -w surekli_kayit.pcap
```

**Wireshark Arayüzünden Ring Buffer:**

1. `Capture -> Options...` menüsünden uygun ağı seçin.
2. `Output` (Çıktı) sekmesine geçin.
3. Çıktı formatını pcapng seçip bir dosya adı belirleyin.
4. **Create a new file automatically** (Otomatik yeni dosya oluştur) seçeneğini işaretleyip, "After 50 Megabytes" ve "Ring buffer with 10 files" ayarlarını yapın.

---

## 10.2 MaxMind GeoIP Entegrasyonu (IP Konumlandırma)

Analiz ettiğiniz IP adreslerinin dünyadaki harita konumlarını (Hangi ülke, şehir ya da ASN numarasında olduğunu) numaralarla görmek için ücretsiz GeoIP veritabanları olan MaxMind kullanılabilir.

1. MaxMind (GeoLite2 Free) sitesinden `City` ve `ASN` veritabanı dosyalarını (.mmdb) indirin ve bir klasöre koyun (Örn: `C:\GeoIP\`).
2. Wireshark'ta `Edit -> Preferences` menüsüne gidin.
3. `Name Resolution` başlığını seçin.
4. En alttaki **MaxMind database directories** satırına `<Edit>` diyerek oluşturduğunuz `C:\GeoIP\` klasörünü gösterin.
5. Wireshark'ı yeniden başlatın. Artık IP katmanı analizlerinde hedefin bağlandığı ülkeyi, hatta şehri paket detaylarında `[Source GeoIP: Turkey]` formatında görebilirsiniz.
6. `Statistics -> Endpoints` sekmesinde `Map` (Harita) özelliğini aktive edebilir ve verilerin dünyadaki konum yoğunluğunu tarayıcınızda açılacak harita üzerinde inceleyebilirsiniz.

---

## 10.3 LUA Scriptleri ile Özel Protokol (Dissector) Yazımı

Wireshark, binlerce protokolü tanır ancak bir şirkete ait "Özel" (Custom Proprietary) bir protokolü, eğer internete açık bir projesi yoksa çözemez. Payload kısmında anlamsız Hex verileri görürsünüz. Bu durumu çözmek için **LUA betik dili** ile kendi çeviricinizi (Dissector) Wireshark'a tanıtabilirsiniz.

LUA kodlarıyla "Bu paketin ilk 4 byte'ı komut ID'si, sonraki 8 byte timestamp'tir" gibi eşleştirmeler yapılarak, o porttan geçen veriler Wireshark arayüzünde anlamlı bir ağaca dönüşür.

> [!TIP]
> **Not:** Özel Dissector geliştirme, ayrı ve detaylı bir yazılım geliştirme sürecidir. LUA eklentilerinizi `C:\Program Files\Wireshark\plugins\` klasörüne atarak devreye alabilirsiniz. Bu işlem kurum içindeki AR-GE departmanlarınca sık kullanılmaktadır.

[< Başa Dön (Ana README Menüsü) >](README.md)

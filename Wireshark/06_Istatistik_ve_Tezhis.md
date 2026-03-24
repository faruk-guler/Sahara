# 6. Uzman İstatistikleri ve Teşhis Araçları

Wireshark, sadece paket okumakla kalmaz ağdaki verilerin dökümünü de sayısal istatistiklere döker. Karmaşık sorunların kök nedenlerini bulurken filtreler tek başına yetersiz kalabilir.

---

## 6.1 İstatistik Menüsü (Statistics)

Bu özellikler yatay menü çubuğundaki **Statistics** sekmesinde yer alır:

* **Endpoints & Conversations:** Ağda en çok veri gönderen/alan portları, IP adreslerini listeler. Aşırı yüksek trafiğe yol açan IP'leri tespit etmek burada saniyeler alır.
* **Protocol Hierarchy:** Paketlerin ne kadarının UDP'den, ne kadarının TCP veya TLS üzerinden geldiğini bir piramit (yüzde ağacı) olarak yansıtır.
* **TCP Stream Graphs:** Uzun süreli paket kayıplarını, Round-trip-time (Gidiş geliş gecikmesi) süresini görsel grafik ekranda sergiler.

---

## 6.2 Expert Info (Uzman Sistem Bilgisi)

Wireshark ekranının sol alt köşesindeki minik yuvarlak duruma tıklayarak "Expert Info" ekranını açabilirsiniz. Ağ sorunlarını derecelerine göre şöyle listeler:

* **Chat (Mavi):** Standart, sorunsuz iş akışları (İletişim başlangıçları vs.)
* **Note (Turkuaz):** HTTP Header sorunları, TCP pencerelerindeki durumlar ya da ufak uyarı sekansları.
* **Warn (Sarı):** Paketin yolda düşüp yeniden istenmesi (TCP Retransmission), sıradan dışı ağ kısıtlamaları (Zero Window).
* **Error (Kırmızı):** Bozulmuş, oynanmış formatlı (Checksum Bad vs.) veya büyük seviye protokol iletişim hataları.

---

## 6.3 Pratik Kısayollar ve İpuçları

* **Birebir Kelime Araması:** Klavyeden `Ctrl + F` yaparak beliren çubukta en soldaki kriteri `String` veya `Regular Expression` olarak değiştirip sadece metinsel bazda hedef aratabilirsiniz.
* **Tüm Akışı İzleme (Follow Stream):** Karışık paket trafiği yerine iki IP arasındaki iletişimin şifresiz tamamını okumak için; HTTP veya TCP bir pakete sağ tıklayın ve `Follow -> TCP Stream` deyin. Tüm süreç karşılıklı metin formatına uyarlanır. Açıldığında payload'lar direkt belirir.
* **Diske Dosya Çıkartma (Export Objects):** Wireshark ile yakalanan ağ verilerindeki FTP, SMB, veya HTTP içerisindeki fiziksel dosyaları bulup gerçek bir resim/script/exe formatında bilgisayarınıza indirebilirsiniz: `File -> Export Objects -> HTTP`.

---

## 6.4 VoIP ve Telefon Analizi (SIP & RTP)

İnternet üzerinden yapılan sesli aramaları (Voice over IP) analiz etmek için özel yöntemler kullanılır.

1. Menüden `Telephony -> VoIP Calls` yolunu izleyin.
2. Yakalanan arama trafiğini listede göreceksiniz. İlgili aramayı seçip `Flow Sequence` derseniz, telefonun çaldığı andan kapandığı ana kadarki tüm paket diyagramını görebilirsiniz.
3. **Sesi Dinleme:** Eğer trafik şifreli değilse (RTP), `Play Streams` butonuna basarak ağ üzerinden geçen ses verisini doğrudan bilgisayarınızda dinleyebilir veya `.wav` olarak kaydedebilirsiniz.

[< Başa Dön (Kurulum) >](01_Kurulum_ve_Arayuz.md)

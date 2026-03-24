<!-- markdownlint-disable MD033 -->
# Wireshark Master Rehberi (Türkçe)

<p align="center">
  <img src="img/ws.png" alt="Wireshark Preview" width="20%">
</p>

Hoş geldiniz! Bu repo, dünyaca ünlü ağ analiz aracı **Wireshark**'ı ilk kurulumdan ileri seviye siber güvenlik analizlerine kadar her yönüyle ele alan modüler bir eğitim setidir.

---

## 📚 Dokümantasyon Modülleri

Analiz yapmak istediğiniz konuyu aşağıdan seçerek başlayabilirsiniz:

1. **[1. Kurulum ve Arayüz Düzeni](01_Kurulum_ve_Arayuz.md)**
   * Farklı platformlara kurulum, Npcap/USBPcap ve kolon özelleştirme.
2. **[2. Temel Filtreleme Teknikleri](02_Temel_Filtreleme.md)**
   * IP, MAC, Port filtreleri ve TCP Bayrak (Flags) analizi.
3. **[3. Gelişmiş Tehdit Analizi](03_Gelismis_Tehdit_Analizi.md)**
   * Gürültü engelleyici filtreler ve özel siber anafor temizlik şablonları.
4. **[4. Kablosuz Ağ (WLAN) Analizi](04_Kablosuz_Ag_Analizi.md)**
   * Monitor mod, Beacon paketleri ve WPA şifre çözme.
5. **[5. Şifreli Trafik (TLS) ve Uygulama Katmanı](05_Sifreli_Trafik_TLS.md)**
   * SNI yakalama, HTTPS trafiğini log dosyasıyla çözme.
6. **[6. Uzman İstatistikleri ve Teşhis Araçları](06_Istatistik_ve_Tezhis.md)**
   * Expert Info, VoIP analizi, akış takibi ve dışa aktarma.
7. **[7. TShark ve Komut Satırı Kullanımı](07_TShark_ve_Komut_Satiri.md)**
   * Grafik arayüzsüz sunucularda CLI üzerinden paket yakalama.
8. **[8. Renklendirme ve Pcap İşlemleri](08_Renklendirme_ve_Pcap_Islemleri.md)**
   * Trafiği boyama kuralları, Editcap/Mergecap ile dosya yönetimi.
9. **[9. Kurumsal Ağ ve Protokol Analizi](09_Kurumsal_ve_Protokol_Analizi.md)**
   * DNS tespiti, SMB analizi ve HTTP/2, HTTP/3 (QUIC) trafik detayları.
10. **[10. Performans Optimizasyonu ve Özel Yapılandırmalar](10_Performans_ve_Ozel_Yapi.md)**
    * Ring Buffer dizin kaydı, MaxMind GeoIP haritalama ve LUA Dissector'ları.

---

## ⚡ Hızlı Başlangıç

Eğer acilen bir tıkanıklığı çözmeniz gerekiyorsa, her modülün sonundaki "Bir Sonraki Sayfa" linklerini takip ederek sırasıyla ilerlemeniz tavsiye edilir.

> [!TIP]
> **Pro Tip:** Wireshark'ta filtreleme yaparken `ssl` anahtar kelimesi yerine her zaman `tls` kullanmayı unutmayın!

---

## 🤝 Katkıda Bulunma

Hata görürseniz veya yeni bir filtre şablonu eklemek isterseniz, lütfen bir Issue açın veya Pull Request gönderin.

**Mutlu Paket Yakalamalar!** 🦈

```bash
git clone git@github.com:ismailtasdelen/wireshark-cheatsheet.git
```

> 💡 Projeye destekte bulunmak veya yeni filtre setleri / modüller eklemek isterseniz her zaman "Issue" oluşturabilir veya "Pull Request" atabilirsiniz!

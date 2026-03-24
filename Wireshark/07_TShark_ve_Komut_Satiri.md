# 7. TShark ve Komut Satırı Kullanımı

Wireshark'ın görsel grafik arabirimi (GUI) her zaman kullanılamayabilir. Özellikle Linux tabanlı CLI sunucularına sızma testleri veya "Headless" (ekransız) ortamlarda ağ analizi yapmak için Wireshark'ın komut satırı aracı olan **TShark** kullanılır.

---

## 7.1 TShark Kurulumu

Genellikle Wireshark ile birlikte gelir. Şayet sunucuda sadece CLI versiyonu arıyorsanız:

```bash
sudo apt install tshark
```

---

## 7.2 Arayüz (Interface) Seçimi ve Listeleme

Paketleri dinlemeye başlamadan önce sistemdeki ağ yüzlerinin ID (numaralarını) öğrenmelisiniz.

* **Tüm ağ arabirimlerini listele:**
  `tshark -D`

---

## 7.3 Temel TShark Paket Yakalama Komutları

* **Belirli bir interfacete (Arayüzde) dinleme yapmak (örneğin eth0 veya 1 numaralı arayüz):**
  `tshark -i eth0` veya `tshark -i 1`
* **Görünümü Wireshark UI gibi satır bazlı okumak:**
  `tshark -i 1 -V` (Tüm paket detaylarını (Verbose) uzun metin olarak basar).
* **Sadece belirli sayıda paket yakalayıp durmak (Örn. 100 paket):**
  `tshark -i 1 -c 100`

---

## 7.4 TShark ile Trafik Filtreleme ve .pcap Kaydı

> [!NOTE]
> TShark'ta Capture (Yakalama) filtreleri Wireshark ile aynıdır. `-f` yakalama (BPF tabanlı) filtresini, `-Y` ise görüntüleme (Display) filtresini belirtir.

**Örnek Kayıt:**
`tshark -i 1 -f "tcp port 80" -w kayit.pcap` (80 portunu dinler ve dosyaya yazar).

---

[< İstatistikler (Bir Önceki Modül) >](06_Istatistik_ve_Tezhis.md) | [< Pcap İşlemleri ve Renklendirme (Sonraki) >](08_Renklendirme_ve_Pcap_Islemleri.md)

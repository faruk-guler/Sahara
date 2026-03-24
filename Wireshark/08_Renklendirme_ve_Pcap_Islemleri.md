# 8. Renklendirme Kuralları ve Pcap İşlemleri

Siber güvenlik analistleri çoğu zaman gigabaytlarca büyüklükteki `.pcap` veya `.pcapng` dosyalarıyla baş başa kalırlar. Bu dosyaları verimli yönetmek ve görsel olarak trafiği boyamak (Coloring Rules) çok kritik bir yetenektir.

---

## 8.1 Renklendirme Kuralları (Coloring Rules)

Wireshark, oluşturduğunuz filtrelere karşılık paketleri **otomatik** renklendirmenize imkan tanır. Bu sayede bir pcap dosyasını aşağı doğru kaydırırken, "Tehlikeli" bir paketi kırmızı arka planla anında fark edebilirsiniz.

1. Menüden `View -> Coloring Rules` seçeneğine tıklayın.
2. Ekrandaki **+** simgesine basarak yeni bir kural satırı oluşturun.
3. **Name (İsim):** `SQLi / DoS Denemesi`
4. **Filter (Filtre):** `http.request.uri contains "SELECT" || frame.len > 2500`
5. Hemen yanındaki **Background** / **Foreground** (Arkaplan/Yazı Rengi) kısımlarından "Arkaplan: Acı Kırmızı", "Yazı rengi: Beyaz" gibi kontrast renkler seçin ve kuralı kaydedin.

> [!TIP]
> Kuralların önceliği vardır. Liste sıralamasında üstte olan kural, alttakileri ezer. Kritik uyarıları her zaman listenin en üstüne sürükleyin.

---

## 8.2 Terminal Pcap Dosya Yönetimi (Editcap & Mergecap)

Büyük `.pcap` dosyaları sistem RAM'ini tüketebilir. Wireshark kurduğunuzda arka planda gelen komut satırı araçlarıyla (Windows CMD üzerinden veya Linux Terminalinden) pcap dosyalarını manipüle edebilirsiniz.

---

### Editcap (PCAP Dosyalarını Bölmek ve Düzeltmek)

Editcap, devasa bir trafik kaydını tarih, paket sayısı veya dosya boyutuna göre küçük parçalara `(split)` ayıran araçtır.

* **Çok büyük bir PCAP dosyasını 100.000 paketlik parçalara bölmek:**
  `editcap -c 100000 devasa_trafik.pcap parcalanmis_trafik.pcap`
  *(Sonuç: parcalanmis_trafik_00000.pcap, _00001.pcap... gibi numaralandırılır)*
* **Zamansal Olarak Bölmek (Sadece belli bir saat aralığını çekmek):**
  `editcap -A "2023-11-20 10:00:00" -B "2023-11-20 10:05:00" tam_gun.pcap olay_ani.pcap`

---

### Mergecap (PCAP Dosyalarını Birleştirmek)

Farklı ağ cihazlarından (Örn: Firewall ve Switch portlarından) aynı ana ait çekilmiş iki pcap kaydını, zaman damgalarına göre **kronolojik** birleştirir.

* **İki veya daha fazla PCAP'i tek bir büyük dosyada zaman sırasıyla kaynaştırmak:**
  `mergecap -w butunlesik_sonuc.pcap router_ici.pcap firewall_disi.pcap`

[< TShark (Önceki Modül) >](07_TShark_ve_Komut_Satiri.md) | [< Kurumsal Ağ Analizi (Sonraki) >](09_Kurumsal_ve_Protokol_Analizi.md)

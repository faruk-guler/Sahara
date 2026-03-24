# 4. Kablosuz Ağ (WLAN) Detayları

Sızma testleri veya ev ağlarındaki kablosuz iletişim Frame tiplerine göre ayrılarak analiz edilir. Wireshark, Monitor modundayken sadece veri akışını değil Yönetim (Management) şemalarını da sergiler.

---

## 4.1 Frame Türüne Göre Temel Ayrımlar

* **Management Frame (`wlan.fc.type == 0`):** Ağ cihazı ve istemci arasındaki tüm kimlik ve bağlantı iletişimi (Authentication/Probe vs.) paketlerini tutar.
* **Control Frame (`wlan.fc.type == 1`):** Veri bütünlüğü ve akış kontrol trafiği verilerini barındırır.
* **Data Frame (`wlan.fc.type == 2`):** Çözümlendiğinde içinden asıl bilgi akışının (payload) çıktığı paket türüdür.

---

## 4.2 Kablosuz İşlemler ve Alt Tür Filtreleri (Subtypes)

| WLAN Aksiyonu | İlgili Filtre | Açıklama |
| :--- | :--- | :--- |
| **Association Request** | `wlan.fc.type_subtype == 0` | Bağlantı (Ağa dâhil olma) isteği |
| **Association Response** | `wlan.fc.type_subtype == 1` | Modemin istemciye bağlantı cevabı |
| **Probe Request** | `wlan.fc.type_subtype == 4` | İstemcilerin etraftaki ağları "yoklama(arama)" isteği |
| **Probe Response** | `wlan.fc.type_subtype == 5` | Modemin SSID yayın ve ağ tanıtım cevabı |
| **Beacon** | `wlan.fc.type_subtype == 8` | Modemin sürekli yaptığı aktif sinyal yayını |
| **Authentication** | `wlan.fc.type_subtype == 11` | Kimlik (şifre vs.) doğrulama aşaması |
| **Deauthentication** | `wlan.fc.type_subtype == 12` | Bağlantının kesilmesi veya zorla koparılması durumu |

---

## 4.3 WLAN'da MAC Bazında Daraltma

* **Hedef (Destination) MAC Taraması:** `wlan.da == 00:11:22:33:44:55`
* **Kaynak (Source) MAC Taraması:** `wlan.sa == 00:11:22:33:44:55`
* **Herhangi Bir Yerdeki MAC:** `wlan.addr == 00:11:22:33:44:55`

---

## 4.4 Şifreli WLAN Trafiğini Çözme (WPA/WPA2 Decryption)

Eğer monitör modda havadan yakaladığınız `[Data]` paketlerinin sadece `wlan` katmanlarında gezinmekten bıktıysanız ve ağın **şifresini (PSK) biliyorsanız**, bu verileri okuyabilirsiniz. (Paketi yakalama işlemine başlamadan *önce*, veya en azından hedeflerin "4-Way Handshake" aşamasını yakalamış olmalısınız).

**Nasıl Yapılır?**

1. Wireshark menüsünden `Edit -> Preferences...` tıklayın.
2. Sol tarafta `Protocols` (Protokoller) sekmesini genişletip **IEEE 802.11** satırını seçin.
3. Çıkan menüde `Enable decryption` (Şifre Çözmeyi Aktif Et) kutucuğunu işaretleyin.
4. `Decryption Keys -> Edit` butonuna basın.
5. Key Type kısmından **wpa-pwd**'yi seçin, Key alanına şu formatı yazın: `WifiSifresi:SSID_Yani_Ağ_Adi`.
   Örn: `SüperŞifre123:Ev_Internetim`
6. `OK` menüleriyle kapatıp kaydedin. Artık eski gizli [Data] verilerinde TCP/HTTP okumaları yapabilirsiniz!

[< Şifreli Trafik (TLS) >](05_Sifreli_Trafik_TLS.md)

# Modül 18: SNMP Traps ve Anlık Ağ Bildirimleri

Birçok Ağ yöneticisi Zabbix'e Switchleri ve Routerları Modül 08'deki "Polling" mantığıyla SNMP (v2c/v3) üzerinden ekler. Zabbix her 5 dakikada bir cihaza "Bana port durumlarını ver" der. Bu geleneksel yöntem ITIL felsefesinde devasa bir kör nokta (Blind spot) yaratır.

## 1. Polling ve Trap Arasındaki Mimari Fark

- **SNMP Polling (Yoklama):** Zabbix 5. dakikada "Cihaz iyi mi?" dedi, cihaz "İyi" dedi. Oysa 6. dakikada cihazda port düştü (Down), 8. dakikada port tekrar kalktı (Up). Zabbix 10. dakikada tekrar sorduğunda cihaz "Ben İyiyim" der. Zabbix o 4 dakikalık kritik felaketi (Flapping port) **ASLA** Ruhuna bile duyuramaz.
- **SNMP Traps (Tuzak/Tetikleme):** Zabbix asla cihazı yormaz ve soru sormaz. Cihaz kendi CPU'suyla izleme yapar; eğer portu 6. dakikada düşerse, anında (Salisesinde) Zabbix'e UDP 162 portundan dev bir XML/Text paketi fırlatır: "Port 24 Çöktü!". Zabbix sorunu anında kavrar.

> Master Kuralları: CPU, RAM, Isı (Sıcaklık) yavaş değişen (Analog) verilerdir, bunlar **Polling** ile çekilir. Ancak BGP kopması, Fan yanması, Port çökmesi saniyelik dijital evrelerdir; bunlar **SNMP Traps** ile dinlenmelidir.

## 2. Zabbix Trap Mimarisi (2026 Kurulumu)

Zabbix doğrudan 162 UDP portunu okuyamaz. Araya bir Linux çevirmeni (`snmptrapd`) girer.

1. Switch, UDP Port 162'ye bir Trap (Hata mesajı fırlatır).
2. `snmptrapd` servisi bu UDP paketini karşılar.
3. Aradaki bir Perl çevirmeni (`zabbix_trap_receiver.pl`) bu kargaşık yazıyı, Zabbix'in okuduğu süzülmüş bir Log text (Metin) dosyasına basar: `/var/log/zabbix/zabbix_traps.tmp`.
4. `zabbix_server` (Ya da Zabbix Proxy) **Trapper Processleri (İşçileri)** saniye başı bu dosyayı açar, okur, Host'u (Makineyi) eşlemeye çalışır ve Zabbix ekranına Alarm olarak düşürür.

### 2.1 Konfigürasyon Dosyası

`snmptrapd.conf` içine şu satır eklenir:

```bash
authCommunity   log,execute,net public # Sadece yetki.
perl do "/usr/bin/zabbix_trap_receiver.pl"; # Çevirmene yetki.
```

Böylelikle cihazlardan akan Traplar Zabbix'in `snmptrap.fallback` isimli genel sepet İtem'ına düşer.

## 3. Syslog-NG vs Zabbix Traps

Trapların tek bir kusuru vardır: Formatları değişkendir (OID numaraları).
Örneğin Cisco cihaz `SNMPv2-SMI::enterprises.9.9.43` gönderirken, Juniper başka bir MIB/OID kodu fırlatır.

Bunu çözmek için `snmptrap["OID"]` kurmak (Yani Regex ile OID süzmek) gerekir. Büyük telco ortamlarında Trapler doğrudan Zabbix'e atılmaz. Traplar önce **Wazuh (SIEM)** veya Syslog sunucusunda çözülür (Parse edilir) ve Alert'a çevrilip son analiz Zabbix API (Modül 15) tetiklemesiyle sonlandırılır.

---
[Önceki Modül](./17_Bakim_Modu.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Cloud ve K8s](./19_Bulut_ve_K8s.md)

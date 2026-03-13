# 📄 Sayfa 15: Extreme Debugging ve Hot Key Yönetimi (Expert Edition)

Bu final bölümünde, normal komutların yetmediği, sistemin sınırlarını zorlayan sorunları nasıl teşhis edeceğinizi öğreneceğiz.

## 1. Hot Key ve Big Key Teşhisi
- **Hot Key:** Bir saniye içinde milyonlarca kez okunan tek bir anahtar (Örn: `son_dakika_haberi`). Bir node'un CPU'sunu kilitler.
    - **Teşhis:** `redis-cli --hotkeys`. (Not: LFU policy açık olmalıdır).
- **Big Key:** RAM'de devasa yer kaplayan tek bir anahtar (Örn: 1GB'lık bir List). Replikasyon ve silme anında sistemi dondurur.
    - **Teşhis:** `redis-cli --bigkeys`.

## 2. GDB (GNU Debugger) ile Redis Çekirdek Analizi
Redis donduğunda veya açıklanamayan bir yavaşlık olduğunda, C düzeyinde ne olduğunu görmek için GDB kullanılabilir.
```bash
# Çalışan Redis sürecine bağlan
gdb -p <redis_pid>

# O an çalışan fonksiyonları listele (Stack Trace)
(gdb) bt full

# Redis'in o anki durumunu çöz (Detach)
(gdb) detach
```
> [!WARNING]
> GDB bir sürece bağlandığı an o süreci DURDURUR. Canlı sistemde (Production) bu işlemi yapmak tüm sistemi o an için dondurur.

## 3. Core Dump Analizi
Redis çöktüğünde (`crash`), bellek durumunu bir dosyaya yazar (**Core Dump**).
- `redis.conf` içinde `dump-core-on-crash yes` ayarı yapılmalıdır.
- Çökme sonrası `gdb /usr/local/bin/redis-server /path/to/core` komutu ile çökme anında hangi C satırında hata oluştuğu incelenebilir.

## 4. Stack Trace ve `SIGUSR1`
Redis kilitlendiğinde (`hang`), ona Linux üzerinden `SIGUSR1` sinyali göndererek loglara o anki işlem yığınını (Stack Trace) yazmasını sağlayabilirsiniz:
```bash
kill -SIGUSR1 <redis_pid>
```
Bu sayede Redis'i durdurmadan loglardan durumu takip edebilirsiniz.

## 5. Network Paket Analizi (Redis-Sniffer)
Eğer sorun ağda mı yoksa Redis'te mi emin değilseniz, ağ paketlerini dinleyebilirsiniz:
```bash
tcpdump -i eth0 port 6379 -w redis_traffic.pcap
```
Ardından bu trafiği Wireshark veya özel araçlarla analiz ederek gecikmenin (RTT) kaynağını bulabilirsiniz.

---


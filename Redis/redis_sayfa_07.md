# 📄 Sayfa 7: İleri Programlanabilirlik ve Keyspace Notifications (Ultra-Detay)

Redis'i sadece veri saklayan değil, olaylara tepki veren akıllı bir sisteme dönüştürün.

## 1. Keyspace Notifications (Pub/Sub üzerinden Olay İzleme)
Redis'te bir anahtar silindiğinde veya süresi dolduğunda uygulamanızın anında haberdar olmasını sağlayabilirsiniz.
- **Konfigürasyon:** `notify-keyspace-events "Ex"` (E: Keyexpired olayları, x: Expired olayları).
- **Kullanım:** Uygulamanız `__keyevent@0__:expired` kanalını dinler. Bir anahtarın süresi bittiği an Redis bu kanala anahtarın ismini mesaj atar.
- **Kullanım Senaryosu:** Süresi dolan bir oturum için veritabanında temizlik yapmak veya bir "timeout" görevini tetiklemek.

## 2. Lua LDB Debugger (Betiği Hata Ayıklama)
Karmaşık Lua betiklerinde hata bulmak zordur. Redis, yerleşik bir debugger sunar:
```bash
redis-cli --ldb --eval script.lua key1 , arg1
```
Bu komutla betiği satır satır çalıştırabilir (`step`), değişkenleri izleyebilir (`print`) ve durak noktaları (`breakpoint`) koyabilirsiniz.

## 3. Redis Streams: PEL (Pending Entries List) Analizi
Streams kullanırken bir tüketici (consumer) mesajı okur ama onay (ACK) vermezse ne olur?
- **PEL:** Redis, her tüketici grubu için "okunmuş ama onaylanmamış" mesajları bu listede tutar.
- **`XPENDING`**: Hangi mesajların takılı kaldığını görmenizi sağlar.
- **`XCLAIM`**: Ölen bir tüketicinin PEL listesindeki mesajlarını başka bir canlı tüketiciye devretmek (mesaj kaybını önlemek) için kullanılır.

## 4. Function as a Code (Redis 7+)
Redis 7 ile betikleri tek tek göndermek yerine **Functions** kütüphanesi geldi.
- Betikler sunucuda kalıcı olarak saklanır, `EVAL` yerine `FCALL` ile çağrılır. Bu, uygulama tarafında kod karmaşasını azaltır ve performansı artırır.

> [!TIP]
> Bir Lua betiği çalışırken Redis "atomic" durumdadır. Bu yüzden çok uzun süren veya sonsuz döngüye giren betiklerden kaçınmalısınız (`lua-time-limit`).


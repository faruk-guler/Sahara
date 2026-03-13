# 📄 Sayfa 5: İleri Güvenlik ve ACL Kategorileri (Ultra-Detay)

Redis ACL sistemi sadece kullanıcı şifrelemek değil, komut bazlı bir güvenlik duvarı oluşturmaktır.

## 1. ACL Komut Kategorileri
Komutları tek tek yazmak yerine kategorileri kullanabilirsiniz:
- **`+@read`**: Sadece okuma komutlarına (GET, HGET vb.) izin verir.
- **`+@write`**: Yazma komutlarına izin verir.
- **`-@dangerous`**: `FLUSHALL`, `CONFIG`, `KEYS` gibi sistemi tehlikeye atacak komutları yasaklar.

Örnek: `ACL SETUSER monitor-user on >sifre +@read -@dangerous`

## 2. Anahtar Desenleri (Key Patterns) ile Kısıtlama
Bir kullanıcının sadece kendi departmanına ait verilere erişmesini sağlayabilirsiniz:
- **`~sales:*`**: Sadece "sales:" ile başlayan anahtarlara izin ver.
- **`&*`**: Sadece belirli Pub/Sub kanallarına izin ver.

## 3. Komut Parametresi Bazlı Kısıtlama (Redis 7+)
Redis 7 ile artık bir komutun hangi parametrelerle çalışacağını bile kısıtlayabilirsiniz:
- **`+GETSET|key_prefix:*`**: Sadece belirli isimdeki anahtarlar üzerinde işlem yapılmasına izin ver.

## 4. Güvenlik Hardening: `rename-command`
Eşsiz bir güvenlik önlemi olarak, tehlikeli komutların isimlerini tahmin edilemez hale getirebilirsiniz:
```text
# redis.conf içinde
rename-command CONFIG "GIZLI_AYAR_KOMUTU_99"
rename-command FLUSHALL "" # Komutu tamamen kapatır
```
> [!WARNING]
> `rename-command` kullanırken ACL sistemi ile çakışmamasına dikkat edin. ACL v2 (Redis 7) sonrası ACL kullanımı daha çok tavsiye edilmektedir.

## 5. Denetim (Auditing): `ACL LOG`
Sisteminizde yetkisiz bir erişim denemesi olduğunda Redis bunu kaydeder.
- **`ACL LOG`**: Hangi kullanıcının, hangi IP'den, hangi şifreyle ve neden reddedildiğini (yetkisiz komut vb.) detaylıca listeler.


# Modül 10: Aktif Yanıt (Active Response)

Active Response, belirli bir alarm tetiklendiğinde sistemin otomatik olarak tepki vermesini sağlayan bir özelliktir.

## Nasıl Çalışır?

1. Wazuh Server bir alarm algılar (Örn: Brute Force saldırısı).
2. Manager, ilgili Agent'a bir komut gönderir.
3. Agent, önceden tanımlanmış bir script'i (Active Response script) çalıştırır.
4. Betik, saldırgan IP'yi firewall üzerinden engeller veya kullanıcı oturumunu kapatır.

## Temel Bileşenler

- **Command:** Çalıştırılacak olan script veya komuttur.
- **Active Response:** Hangi alarmın (ID veya Seviye) hangi komutu tetikleyeceğinin tanımıdır.

## Örnek Yapılandırma (`ossec.conf`)

### Step 1: Komut Tanımı

```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop.sh</executable>
  <expect>srcip</expect>
  <timeout_allowed>yes</timeout_allowed>
</command>
```

### Step 2: Yanıt Tanımı

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712</rules_id> <!-- SSH Brute Force kuralı -->
  <timeout>600</timeout> <!-- 10 dakika engelle -->
</active-response>
```

## Varsayılan Scriptler

Wazuh ile birlikte hazır gelen bazı scriptler şunlardır:

- `firewall-drop`: `iptables` veya `firewalld` kullanarak IP engeller.
- `host-deny`: `/etc/hosts.deny` dosyasına IP ekler.
- `disable-account`: Kullanıcı hesabını kilitler (Windows/Linux).
- `restart-wazuh`: Agent'ı yeniden başlatır.

## Güvenlik Uyarısı

Active Response çok güçlü bir araçtır. Yanlış yapılandırılırsa kendi erişiminizi engelleyebilirsiniz (False Positive). Bu nedenle `timeout` kullanmak ve kuralları dikkatli seçmek kritiktir.

## Beyaz Liste (White-list)

Önemli IP adreslerinin (Yönetici bilgisayarları, güvenilir sunucular) yanlışlıkla engellenmesini önlemek için Manager üzerindeki `ossec.conf` dosyasına beyaz liste eklenmelidir:

```xml
<global>
  <white_list>127.0.0.1</white_list>
  <white_list>192.168.1.50</white_list> <!-- Güvenilir Admin IP -->
</global>
```

---

[README'ye Dön](README.md)

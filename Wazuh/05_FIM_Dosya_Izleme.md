# Modül 05: Dosya Bütünlüğü İzleme (FIM)

File Integrity Monitoring (FIM), kritik dosyalardaki değişiklikleri takip etmek için kullanılır.

## FIM Nasıl Çalışır?

Wazuh, belirtilen dizinleri periyodik olarak veya gerçek zamanlı olarak tarar. Dosyanın Hash (MD5, SHA1, SHA256), boyutu, sahibi ve izinleri değiştiğinde alarm üretir.

## Konfigürasyon (`ossec.conf`)

FIM ayarları `<syscheck>` bloğu altında yapılır.

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency> <!-- 12 saatte bir tam tarama -->

  <!-- İzlenecek dizinler -->
  <directories check_all="yes" realtime="yes">/etc,/usr/bin,/usr/sbin</directories>
  <directories check_all="yes" realtime="yes" report_changes="yes">/var/www/html</directories>

  <!-- Windows için kayıt defteri izleme -->
  <windows_registry>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run</windows_registry>

  <!-- Dışlanacak dosyalar -->
  <ignore>/etc/mtab</ignore>
</syscheck>
```

## Önemli Parametreler

- `realtime="yes"`: Dosya değiştiği anda alarm üretir (Linux için Inotify, Windows için ReadDirectoryChangesW kullanılır).
- `report_changes="yes"`: Dosyanın içeriğindeki değişikliği (Diff) server'a gönderir (Dikkat: Sadece metin dosyaları için kullanılmalıdır).
- `check_all="yes"`: Boyut, izinler, sahiplik ve hash değerlerini kontrol eder.

## Whodata Özelliği

"Dosyayı kim değiştirdi?" sorusuna yanıt verir. Linux'ta `Auditd`, Windows'ta `SACL` altyapısını kullanarak değişikliği yapan kullanıcıyı ve süreci loglar.

```xml
<directories check_all="yes" whodata="yes">/etc</directories>
```

---
[README'ye Dön](README.md)

# Modül 21: Gerçek Hayat Uygulama Senaryoları

Bu modülde, Wazuh kullanarak nasıl gelişmiş tehdit avcılığı (Threat Hunting) yapılacağını örneklerle göreceğiz.

## Senaryo 1: Ransomware (Fidye Yazılımı) Algılama

1. **Belirti:** Aynı anda binlerce dosyanın isminin ve uzantısının değişmesi.
2. **Wazuh Yanıtı:** FIM modülü bu toplu değişikliği yakalar.
3. **Savunma:** Bir kural yazılır (Örn: 1 dakika içinde 100+ dosya değişikliği) ve Active Response ile şüpheli süreç durdurulur.

## Senaryo 2: Brute Force (Kaba Kuvvet) Saldırısı

1. **Belirti:** Linux'ta `sshd` veya Windows'ta `4625` (Logon Failure) olaylarının artması.
2. **Wazuh Yanıtı:** Hazır kurallar bu durumu algılar.
3. **Savunma:** Active Response ile saldırgan IP'si firewall üzerinden 1 saatliğine engellenir.

## Senaryo 3: Yetkisiz Privilege Escalation

1. **Belirti:** Normal bir kullanıcının `sudo` veya `psexec` kullanarak yetki kazanmaya çalışması.
2. **Wazuh Yanıtı:** Log analizi modülü yetkisiz denemeleri yakalar.
3. **Savunma:** Güvenlik ekibine Telegram/Slack üzerinden anlık uyarı gönderilir.

## Senaryo 4: Veri Sızıntısı (Exfiltration)

1. **Belirti:** Gece saatlerinde web sunucusundan dışarıya büyük miktarda veri transferi.
2. **Wazuh Yanıtı:** Netflow veya Firewall loglarını analiz eden kurallar tetiklenir.

---

[README'ye Dön](README.md)

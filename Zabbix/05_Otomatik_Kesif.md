# Modül 05: Otomatik Keşif (Discovery) ve Auto-Registration

Büyük ölçekli operasyonlarda sunucuları Zabbix arayüzünden tek tek eklemek (Manual Host Creation) mümkün değildir. Yönetimi otomatize eden iki temel mekanizma bulunur: **Network Discovery** ve **Active Agent Auto-Registration**. Ayrıca host seviyesinin altına inen üçüncü yöntem **Low-Level Discovery (LLD)**'dir (Bkz. Modül 06).

## 1. Yöntemlerin Karşılaştırılması

| Özellik | Network Discovery (Ağ Keşfi) | Active Auto-Registration (Ajan Kaydı) |
| :--- | :--- | :--- |
| **İnisiyatif Kimde?** | Zabbix Server/Proxy bir IP aralığını tarar. | Zabbix Ajanı ayağa kalktığında Server'a "Ben geldim" der. |
| **Bağımlılık** | Ajan gerektirmez (SNMP, ICMP, TCP Port tarayabilir). | Sadece Active Modda çalışan Zabbix Ajanı gerektirir. |
| **Hız** | Yavaştır (Örn: /16 bir IP bloğunu taramak saatler alabilir). | Anlıktır (Sunucu kurulduğu saniye Zabbix'e dahil olur). |
| **Kullanım Alanı** | Network cihazları (Switch, Router), Yönetilmeyen VM'ler. | Cloud-Native uygulamalar, Otomatik Ölçeklenen Sunucular (AWS ASG). |
| **Cloud-Friendly mi?** | IP'ler sürekli değiştiği için Cloud için "Kötü" bir yöntemdir. | Cloud için biçilmiş kaftandır. IP değişse dahi Name üzerinden işler. |

## 2. Active Agent Auto-Registration (Master Practice)

2026 yılı altyapı kodlama (IaC) standartlarında **Auto-Registration** tek geçerli sunucu kayıt yöntemidir. 

**Senaryo:** AWS veya vSphere ortamınızda yeni 50 adet Linux Nginx sunucusu oluşturuldu. Bunlar otomatik çalışıp Zabbix'e girmeli.

### 2.1 Ajan Tarafındaki Konfigürasyon (`zabbix_agent2.conf`)
Agent 2 kurulduğunda `HostMetadata` özelliği kullanılarak sunucunun "kimliği" Zabbix'e fırlatılır.

```ini
ServerActive=zabbix.sirket.com
HostnameItem=system.hostname

# Sunucu metadata'sında OS, Tip ve Ortam bilgilerini birleştiriyoruz.
# Örnek: Linux | WebServer | Nginx | Prod
HostMetadata=Linux-Web-Nginx-Prod
```

### 2.2 Zabbix Arayüzünde Kural (Action) Tanımlama
Server'da bu metadatayı karşılayacak bir kural (`Configuration` -> `Actions` -> `Auto registration actions`) yazılmalıdır.

**Conditions (Şartlar):**
- `Host metadata` *contains* `Linux`
- `Host metadata` *contains* `Nginx`
- `Host metadata` *contains* `Prod`

**Operations (Uygulanacak Aksiyonlar):**
1. **Add host:** Sunucuyu Zabbix'e kaydet.
2. **Add to host groups:** `Linux Servers`, `Web Servers`, `Production` gruplarına ekle.
3. **Link to templates:** `Template OS Linux by Zabbix agent active`, `Template App Nginx by HTTP` şablonlarını sunucuya bağla.

### Sonuç: Zero-Touch Monitoring
Sunucu açıldığı anda, Ansible vb. bir araç Zabbix Agent 2'yi yükler. Ajan "Linux-Web-Nginx-Prod" kelimesini Zabbix Server'a atar. Sunucu yukarıdaki şartlar nedeniyle otomatik gruba alınır ve Nginx metrikleri izlenmeye başlanır. **Hiçbir insan müdahalesi gerektirmez.**

## 3. Network Discovery (Ağ Keşfi)

Daha çok SNMP çalıştıran Switch/Router ortamı için kullanılır. 
Yöntem: 
- `Data Collection` -> `Discovery` menüsünden yeni bir kural oluşturulur.
- **IP Range:** `192.168.10.1-254`
- **Checks:** `SNMPv3 agent` `OID: 1.3.6.1.2.1.1.1.0` (Üretici tespiti)

Network Discovery kuralları sistemi yorduğu için, Proxy'ler (Modül 12) kullanılarak keşif işlemlerini Server üzerinden almak performans için altın bir kuraldır (Tüm taramaları `Discoverer` node'larına veya proxylerine yönlendirmek).

---
[Önceki Modül](./04_Zabbix_Agent2.md) | [README'ye Dön](./README.md) | [Sonraki Modül: Low-Level Discovery](./06_LLD.md)

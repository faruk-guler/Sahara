# Modül 15: Master Düzey Zabbix API ve Otomasyon (IaC)

Zabbix'in gücü web arayüzünden değil, arka planda çalışan devasa API (Application Programming Interface) katmanından gelir. Web arayüzünde yaptığınız her tıklama (Buton) aslında arka planda bu API'ye giden bir JSON-RPC isteğinden ibarettir.

## 1. Mimarinin Kalbi: JSON-RPC (Remote Procedure Call)

Zabbix API, REST mimarisini kullanmaz. Doğrudan `POST` istekleriyle `http://IP/zabbix/api_jsonrpc.php` adresine veri gönderir.
Bir Host (Sunucu) oluşturmak, bir alarm kapatmak veya 1000 kullanıcının şifresini değiştirmek saniyeler içinde API'den yapılabilir.

### 1.1 API Token (2026 Güvenlik Standardı)

Eskiden API'ye girerken JSON içinde `user` ve `password` gönderip geçici bir Hash (Oturum ID) alınırdı. Artık bu yöntem güvensiz kaldı.
Bütün istekler için 1 yıllık bir uzun vadeli "API Token" üretilir. Cihazları otomatize ederken kodunuzun (Script) içine şifreniz değil bu token konur:
`Bearer cb2f37xxxx9812981xxx...`

## 2. Python (PyZabbix) ile Kurumsal Yönetim Stratejisi

Elinizde, sanal ortamdan (Vcenter) aniden çekilmiş ve Zabbix'te hala kırmızı duran (silinmeyi unutulmuş) 500 cihaz var. Tek tek elle silmeye çalışırsanız gününüz biter.

1. Python'un `pyzabbix` kütüphanesi yüklenir.
2. Bir Master-Script yazılır.

```python
from pyzabbix import ZabbixAPI

zapi = ZabbixAPI("http://192.168.10.50/zabbix")
# Token Kullanımı (Şifre yerine)
zapi.login(api_token='db623bxxxx_your_token_here_xxxx')

print(f"Bağlanıldı! Zabbix Versiyon: {zapi.api_version()}")

# "Disaster" statüsünde ve 30 gündür erişilemeyen (pasif) tüm hostları bul
unreachable_hosts = zapi.host.get(
    filter={"status": 1},
    output=["hostid", "name"]
)

# Hostları otomatik olarak sil döngüsü
for host in unreachable_hosts:
    zapi.host.delete(host['hostid'])
    print(f"Silinen Cihaz: {host['name']}")
```

## 3. Infrastructure as Code (Ansible ile Zabbix)

Zabbix'te elle konfigürasyon (Tıklama) yapmak insan hatasına yol açar. Bir `Linux Template` sunucuya yanlış atanırsa alarm kaosu oluşur.

Bunun yerine 2026'da Zabbix konfigürasyonları tamamen **Git (GitHub/GitLab)** üzerinden tutulur.
Tüm organizasyon için bir Ansible Playbook (`zabbix_deploy.yml`) yazılır:

```yaml
- name: Yeni Şubedeki Veritabanlarını Zabbix'e Ekle
  hosts: localhost
  tasks:
    - name: Create or Update PostgreSQL Host in Zabbix
      community.zabbix.zabbix_host:
        server_url: http://zabbix.sirket.com
        login_user: admin
        login_password: "{{ secrets.zabbix_password }}"
        host_name: "ANK-DB-01"
        visible_name: "Ankara Merkez DB 1"
        host_groups:
          - "Database Servers"
          - "Ankara Datacenter"
        link_templates:
          - "Template DB PostgreSQL"
        interfaces:
          - type: 1 # Zabbix Agent Türü
            main: 1
            useip: 1
            ip: "10.0.0.5"
            dns: ""
            port: "10050"
        state: present
```

**Mimarinin Faydası:** Cihazların IP'si mi değişti? Git reposunda IP'yi güncelleyin, Jenkins (veya GitLab CI) Ansible'ı çalıştırsın, Zabbix anında kendini yenilesin. Hiçbir mühendis Zabbix arayüzünden manuel "Update" butonu aramasın.

---
[Önceki Modül](./14_Kullanici_Yonetimi.md) | [README'ye Dön](./README.md) | [Sonraki Modül: DB Tuning](./16_Database_Optimizasyon.md)

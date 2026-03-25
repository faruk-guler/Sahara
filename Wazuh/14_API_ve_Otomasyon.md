# Modül 14: Wazuh API Kullanımı ve Otomasyon

Wazuh, otomasyon süreçleri için güçlü bir RESTful API sunar (Varsayılan Port: **55000**). Bu modül üzerinden platformu diğer sistemlerle entegre edebilirsiniz.

## Servis Yönetimi

API işlemleri Manager üzerindeki `wazuh-apid` servisi tarafından yönetilir.

## Temel İşlevler

1. **Uç Nokta Bilgisi:** Ajanların durumu, IP adresleri ve işletim sistemleri.
2. **Kural Yönetimi:** Kuralları sorgulama ve test etme.
3. **Konfigürasyon:** Manager ve Agent ayarlarını uzaktan okuma.
4. **Log Sorgulama:** Üretilen alarmleri filtreleyerek çekme.

## Kimlik Doğrulama (Auth)

API'yi kullanmadan önce Manager üzerinden bir token almanız gerekir:

```bash
TOKEN=$(curl -u user:password -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
```

## Örnek Python Scripti

Aşağıdaki script, bağlantısı kopmuş (disconnected) durumdaki tüm ajanları listeler:

```python
import requests

url = "https://localhost:55000/agents"
headers = {"Authorization": f"Bearer {TOKEN}"}
params = {"status": "disconnected"}

response = requests.get(url, headers=headers, params=params, verify=False)
print(response.json())
```

## Önemli Endpoints

- `GET /agents`: Tüm ajanları listeler.
- `GET /alerts`: Alarmleri getirir.
- `GET /syscheck/{agent_id}`: FIM tarama sonuçlarını döner.

## Özet

Wazuh API, SIEM platformunuzu diğer araçlarla (SOAR, Ticketing vb.) entegre etmek için en güçlü silahtır.

---

[README'ye Dön](README.md)

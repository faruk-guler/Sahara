# 🔐 Ceph Güvenlik ve Kimlik Doğrulama Rehberi

Bu doküman, Ceph cluster'ının güvenlik yapılandırmasını, kullanıcı yönetimini, şifreleme ve SSL/TLS ayarlarını kapsar.

---

## 1. Cephx Kimlik Doğrulama

### Cephx Nedir?

Cephx, Ceph'in dahili kimlik doğrulama protokolüdür. Kerberos'a benzer şekilde çalışır ve şunları sağlar:

- İstemci kimlik doğrulaması
- Mesaj bütünlüğü
- Replay attack koruması

### Cephx Durumunu Kontrol Etme

```bash
ceph config get mon auth_cluster_required
ceph config get mon auth_service_required
ceph config get mon auth_client_required
# Çıktı: cephx (varsayılan)
```

---

## 2. Kullanıcı ve Keyring Yönetimi

### Mevcut Kullanıcıları Listeleme

```bash
ceph auth ls
```

### Yeni Kullanıcı Oluşturma

```bash
# Temel sözdizimi
ceph auth get-or-create client.<isim> <capabilities>

# RBD kullanıcısı
ceph auth get-or-create client.rbd-user \
    mon 'profile rbd' \
    osd 'profile rbd pool=mypool'

# CephFS kullanıcısı
ceph auth get-or-create client.cephfs-user \
    mon 'allow r' \
    mds 'allow rw' \
    osd 'allow rw pool=cephfs_data, allow rw pool=cephfs_metadata'

# RGW admin kullanıcısı
ceph auth get-or-create client.rgw-admin \
    mon 'allow rwx' \
    osd 'allow rwx'
```

### Keyring Dosyasına Kaydetme

```bash
ceph auth get-or-create client.myapp \
    mon 'allow r' \
    osd 'allow rw pool=apppool' \
    -o /etc/ceph/ceph.client.myapp.keyring
```

### Kullanıcı Yetkilerini Görüntüleme

```bash
ceph auth get client.rbd-user
```

### Kullanıcı Yetkilerini Güncelleme

```bash
ceph auth caps client.myapp \
    mon 'allow r' \
    osd 'allow rwx pool=newpool'
```

### Kullanıcı Silme

```bash
ceph auth del client.myapp
```

---

## 3. Capability (Yetki) Profilleri

### Monitor (MON) Capabilities

| Yetki | Açıklama |
| :--- | :--- |
| `allow r` | Sadece okuma (cluster durumu) |
| `allow rw` | Okuma/yazma |
| `allow rwx` | Tam yetki |
| `profile rbd` | RBD için önceden tanımlı profil |

### OSD Capabilities

| Yetki | Açıklama |
| :--- | :--- |
| `allow r` | Sadece okuma |
| `allow rw` | Okuma/yazma |
| `allow rwx` | Tam yetki (class metodlarına erişim dahil) |
| `allow rw pool=<pool>` | Belirli pool'a yazma |
| `profile rbd pool=<pool>` | RBD profili (clone, snapshot dahil) |

### MDS Capabilities (CephFS için)

| Yetki | Açıklama |
| :--- | :--- |
| `allow r` | Sadece okuma |
| `allow rw` | Okuma/yazma |
| `allow rw path=/home/user` | Belirli dizine sınırlı yetki |

---

## 4. En Az Yetki Prensibi (Örnekler)

### Proxmox için Keyring

```bash
ceph auth get-or-create client.proxmox \
    mon 'profile rbd' \
    osd 'profile rbd pool=vm-pool, profile rbd-read-only pool=iso-pool'
```

### Kubernetes CSI için Keyring

```bash
ceph auth get-or-create client.k8s-csi \
    mon 'profile rbd' \
    osd 'profile rbd pool=k8s-pool' \
    mgr 'allow rw'
```

### Backup Uygulaması için (Sadece Okuma)

```bash
ceph auth get-or-create client.backup \
    mon 'allow r' \
    osd 'allow r pool=*'
```

---

## 5. SSL/TLS Yapılandırması

### Dashboard için SSL

Ceph Dashboard varsayılan olarak self-signed sertifika kullanır.

```bash
# Mevcut sertifikayı görüntüle
ceph dashboard get-ssl-certificate

# Özel sertifika yükle
ceph dashboard set-ssl-certificate -i /path/to/cert.pem
ceph dashboard set-ssl-certificate-key -i /path/to/key.pem

# Dashboard'u yeniden başlat
ceph mgr module disable dashboard
ceph mgr module enable dashboard
```

### RGW için SSL

```bash
# SSL portunu ayarla
ceph config set client.rgw rgw_frontends "beast ssl_port=443"

# Sertifika dosyalarını belirle
ceph config set client.rgw rgw_frontends "beast ssl_port=443 ssl_certificate=/etc/ceph/rgw.pem"
```

### Messenger v2 Encryption (Dahili Trafik)

Ceph Nautilus'tan itibaren msgr2 protokolü şifreleme destekler.

```bash
# Şifreleme modunu kontrol et
ceph config get global ms_cluster_mode

# Şifrelemeyi zorla (cluster içi trafik)
ceph config set global ms_cluster_mode secure
ceph config set global ms_service_mode secure
ceph config set global ms_client_mode secure
```

---

## 6. RGW (S3) Güvenliği

### Bucket Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {"AWS": ["arn:aws:iam:::user/testuser"]},
            "Action": ["s3:GetObject"],
            "Resource": ["arn:aws:s3:::mybucket/*"]
        }
    ]
}
```

```bash
# Policy uygula
aws --endpoint-url http://rgw:8000 s3api put-bucket-policy \
    --bucket mybucket \
    --policy file://policy.json
```

### Bucket ACL

```bash
# Public-read (dikkatli kullan!)
aws --endpoint-url http://rgw:8000 s3api put-bucket-acl \
    --bucket mybucket \
    --acl private
```

### RGW Kullanıcı Quota

```bash
# Kullanıcı için maksimum boyut
radosgw-admin quota set --quota-scope=user --uid=testuser --max-size=10G

# Etkinleştir
radosgw-admin quota enable --quota-scope=user --uid=testuser
```

---

## 7. RBD Şifreleme (LUKS)

RBD image'ları LUKS ile şifrelenebilir.

### Şifreli Image Oluşturma

```bash
# Boş image oluştur
rbd create --size 10G mypool/encrypted-disk

# LUKS formatla
rbd encryption format mypool/encrypted-disk luks2 /path/to/passphrase.txt
```

### Şifreli Image'ı Map Etme

```bash
rbd device map mypool/encrypted-disk \
    --encryption-format luks2 \
    --encryption-passphrase-file /path/to/passphrase.txt
```

---

## 8. Network Güvenliği

### Firewall Kuralları (Örnek: UFW)

```bash
# Monitor (MON) - TCP 3300, 6789
ufw allow from 192.168.1.0/24 to any port 3300,6789 proto tcp

# OSD - TCP 6800:7300
ufw allow from 192.168.1.0/24 to any port 6800:7300 proto tcp

# MDS - TCP 6800:7300
ufw allow from 192.168.1.0/24 to any port 6800:7300 proto tcp

# RGW - TCP 8000 (veya 443)
ufw allow from any to any port 8000 proto tcp

# Dashboard - TCP 8443
ufw allow from 192.168.1.0/24 to any port 8443 proto tcp
```

### Ceph Portları Özet

| Servis | Port | Protokol |
| :--- | :--- | :--- |
| MON | 3300, 6789 | TCP |
| OSD | 6800-7300 | TCP |
| MDS | 6800-7300 | TCP |
| MGR Dashboard | 8443 | TCP |
| RGW | 7480, 80, 443 | TCP |

---

## 9. Audit Logging

### Ceph Audit Log

```bash
# Audit log'u etkinleştir
ceph config set global mon_cluster_log_file /var/log/ceph/ceph-audit.log
ceph config set global mon_cluster_log_to_file true

# Log seviyesini ayarla
ceph config set global log_to_file true
ceph config set global debug_mon 1/5
```

### RGW Ops Log

```bash
# S3 operasyon logları
ceph config set client.rgw rgw_enable_ops_log true
ceph config set client.rgw rgw_ops_log_file_path /var/log/ceph/rgw-ops.log
```

---

## 10. Güvenlik Best Practices

### ✅ Yapılması Gerekenler

- Her uygulama için ayrı keyring oluşturun
- En az yetki prensibini uygulayın
- SSL/TLS'i production'da mutlaka etkinleştirin
- Network segmentasyonu yapın (Public vs Cluster network)
- Düzenli olarak kullanılmayan keyring'leri silin
- Dashboard şifresini güçlü tutun

### ❌ Yapılmaması Gerekenler

- `client.admin` keyring'i uygulamalara vermeyin
- Cephx'i devre dışı bırakmayın (`auth_cluster_required = none`)
- Keyring dosyalarını version control'e eklemeyin
- RGW'yi internet'e SSL olmadan açmayın
- Tüm pool'lara `allow *` yetki vermeyin

---

## 11. Güvenlik Denetimi Checklist

```text
[ ] Tüm servisler Cephx authentication kullanıyor mu?
[ ] Admin keyring sadece admin sunucusunda mı?
[ ] Her uygulama kendi keyring'ine mi sahip?
[ ] Dashboard SSL sertifikası güncel mi?
[ ] RGW SSL etkin mi?
[ ] Firewall kuralları doğru mu?
[ ] Audit log aktif mi?
[ ] Eski/kullanılmayan kullanıcılar silindi mi?
[ ] msgr2 encryption etkin mi?
```

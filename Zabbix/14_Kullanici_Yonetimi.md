# Modül 14: Yetkilendirme (RBAC), SSO ve Kurumsal Güvenlik

Sistem Administrator'u (Süper Admin) olarak 10 kişilik bir DevOps ekibi, 20 kişilik bir L1 Operasyon (NOC) ekibi, ve salt okuma (Read-only) vizyonu talep eden C-Level Yöneticileriniz var. Arayüz yetki yönetimi (Role-Based Access Control) felaketle sonuçlanmamak için elzemdir.

## 1. Zabbix 7.0'ın Katı RBAC (Rol Tabanlı Erişim) Modeli

Eski nesil Zabbix versiyonlarında yetkiler "Hangi Kullanıcı Grubu (User Group) Hangi Host Grubunu (Host Group) görsün?" ile sınırlıydı. Ancak 2026 RBAC modelinde, her bir düğmenin, her bir tıklamanın yetkisini kapatabilirsiniz.

- `User Roles` (Kullanıcı Rolleri) sekmesi tüm yetki matrisini belirler.
- L1 Operasyon ekibi için **Sadece Alarm Kapatma Rolü:** Bu roldeki kişiler cihaz görebilir, problem olduğunu görebilir ama hiçbir ayara müdahale edemeyeceği gibi, yalnızca Problemin (Acknowledge problem) üzerine "İşleme alıyorum" notu yazıp alarmı uykuya alabilirler.
- DBA Ekibi için **API Rolü:** Veritabanı ekibi arayüze hiç giriş yapmadan sadece API çekebilsin (Execute API requests) seçeneği açılabilir. Diğer herkes de API erişimi kapatılır.
- **UI Elements Rolü:** C-Level CEO girdiğinde sol taraftaki kalabalık menüleri (Configuration, Administration, Data Collection) komple kapatıp onun gözünde sadece "Dashboards" butonunu parlatabilirsiniz.

## 2. SSO (Single Sign-On) ve Active Directory (LDAP) Kurgusu

Kullanıcıların Zabbix'e bağımsız bir parola ile girmesi parola politikanız (Şifrenin 30 günde değişmesi, özel karakter kuralları vb.) açısından felakettir. 2026 yılında **SAML (Security Assertion Markup Language)**, OpenID veya en kötüsü **LDAP/Active Directory** entegrasyonu mecburi olmalıdır.

- `Authentication` -> `SAML Settings` ayarlarına gidilir.
- Zabbix sunucunuza ait public sertifika, Keycloak, Microsoft Entra ID (Azure AD), veya Okta konfigürasyonunuza gömülür.
- Bu sayede çalışan sabah ofise geldiğinde tek bir tuşla ("Login with SAML") doğrudan Zabbix arayüzüne (Windows kimliği ile) güvenle sızar.

**JIT (Just-In-Time) User Provisioning Kurgusu:**
Geleneksel LDAP yapılarında siz (Admin olarak) Ahmet beyin hesabını Zabbix'ten elinizle oluşturur, sonra LDAP'yi karşısına koyardınız. JIT sistemiyle; Zabbix ortamında "Ahmet" diye birisi hiç yoktur. Ahmet Bey SAML ile Windows ID'sini atıp giriş yaptığı saniye Zabbix onun AD Grubu'na (Group Mapping) bakar: *"Ops_Team grubunda!"* diyerek hesabını havada oluşturur (Zabbix tarafında hiçbir insan dokunuşuna yer bırakmaz).

## 3. Güvenlik Devrimi: Native MFA Uygulamaları

SAML kullanmıyorsanız ve çalışanlar Zabbix'e varsayılan Database Auth ile giriyorsa, (Aksi halde şifre çalınması her şeye zarar verir) Zabbix 7.0 ile hayat bulan yerleşik **MFA (Multi-Factor Authentication)** zorunlu kılınmalıdır.

1. `Administration` menüsünden MFA özelliği açılır (Google Authenticator / TOTP) seçilir.
2. Personel ilk girişte bir QR kod okutur.
3. Sonraki tüm oturumlarda şifresinden sonra 6 haneli zaman damgalı token (TOTP) istenir.

## 4. Audit Log (Denetim İzi) Kavramı

Görevi biten, tayini çıkan ya da içeriden (Insider Threat) zarar veren bir çalışanın, sabah saat 03:00'te sistemdeki kritik bir "Core Switch" sunucusunu bilerek (veya yanlışlıkla) `Delete` (Sildiğini) hayal edin.

`Reports` -> `Audit log` menüsü altında arka plandaki tüm değişiklik işlemleri tutulur.

- Timestamp: 03:00:15
- User: Ahmet.Kaymaz
- Resource: Host / Core Switch
- Action: Delete
- Details: (IP Address, vb.)

Zabbix Database'ini Syslog kuralları üzerinden merkezi bir SIEM veya Wazuh sunucusuna sürekli fırlatmak, IT siber güvenliği için 2026 yılı master mimarisinin değişilmez şartıdır.

---
[Önceki Modül](./13_Dashboardlar.md) | [README'ye Dön](./README.md) | [Sonraki Modül: API Otomasyonu](./15_API_Otomasyon.md)

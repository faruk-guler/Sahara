# Modül 17: Profesyonel Konfigürasyon

Git ayarlarını sadece `user.name` ve `email`'den ibaret sanıyorsanız çok şey kaçırıyorsunuz. Bu modülde, kurumsal seviyede konfigürasyon yönetimini ve dosya bazlı davranışları inceleyeceğiz.

---

## 7.1 Conditional Includes (Şartlı Dahil Etme)

Bilgisayarınızda hem kişisel projeler hem de iş projeleri var. İş projelerinin `work/` klasörü altında olduğunu varsayalım. Git'in bu klasörde otomatik olarak farklı bir email adresi kullanmasını sağlayabilirsiniz.

**`~/.gitconfig` dosyanıza ekleyin:**

```text
[user]
    name = Adınız Soyadınız
    email = kisisel@email.com

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

**`~/.gitconfig-work` içeriği:**

```text
[user]
    email = sirket@email.com
```

*Artık `~/work/` dizini altındaki herhangi bir repoda commit attığınızda, Git otomatik olarak şirket mailinizi seçer.*

---

## 7.2 .gitattributes ile Dosya Davranışlarını Değiştirme

Proje kök dizinindeki `.gitattributes` dosyası, dosyaların Git tarafından nasıl algılanacağını belirler.

* **Satır Sonu (Line Endings):** Windows (CRLF) ve Linux (LF) arasındaki karmaşayı bitirmek için: `* text=auto`
* **Binary Dosyalar:** Belirli dosyaların asla "text" olarak diff edilmemesi gerektiğini belirtin: `*.jpg binary`
* **Custom Diff-tool:** Belirli dosya tipleri için özel diff araçları tetikleyin: `*.pdf diff=pdf`

---

## 7.3 Git Template Directories

Her yeni `git init` veya `git clone` yaptığınızda, belirli bir `pre-commit` hook'unun veya `.gitignore` dosyasının otomatik olarak oluşmasını istiyor musunuz?

1. Bir şablon klasörü oluşturun: `mkdir -p ~/.git_template/hooks`
2. İçine standart hook'larınızı koyun.
3. Git'i bu klasörü kullanmaya zorlayın:

   ```bash
   git config --global init.templateDir ~/.git_template
   ```

---

## 7.4 Global Ignore List

Her projede `.gitignore` içine `node_modules` veya `.DS_Store` yazmaktan bıktınız mı? Global bir ignore listesi oluşturun:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

*Artık her projenin içinde bu dosyaları gizlemenize gerek kalmaz.*

---

## 7.5 Autostash (Rebase Sırasında Kolaylık)

Bozuk olmayan ama commitlenmemiş değişiklikleriniz varken `git pull --rebase` yapmak isterseniz Git hata verir. `autostash` ile Git bunları sizin yerinize otomatik saklar ve işlem bitince geri getirir.

```bash
git config --global rebase.autostash true
```

---

[← Geri: Modül 16](16_Ileri_Seviye_Geri_Alma_ve_Kurtarma.md) | [İleri: Modül 18 - Dev Projelerde Çalışma Teknikleri →](18_Dev_Projelerde_Calisma_Teknikleri.md)

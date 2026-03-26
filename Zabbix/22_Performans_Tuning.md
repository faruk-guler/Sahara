# Modül 22: Zabbix Master Tuning (Devasa Altyapıları Yönetmek)

Zabbix performansında sorun olduğunda acemi sistemciler sunucuya RAM ve CPU basarlar (Hardware Scale-Up). Hâlbuki MySQL'de olduğu gibi `zabbix_server.conf` ince ayarlarla beslenmezse bu kaynakların hepsi 0 etki yaratır (Yazılımsal Darboğaz/Bottleneck).

Master Tuning metodolojisi, "Queue" (Kuyruk) mantığına ve "Internal Items" metriklerine dayanır.

## 1. Zabbix Queue Analizi (Sistemin Tansiyon Aleti)

Zabbix, 10:00:00'da kontrol etmesi beklenen sunucudan 10:00:15 olmasına rağmen veri alamadıysa o veriyi "Gecikenler (Delay)" listesine yani `Queue (Kuyruğa)` atar.

- `Administration` -> `Queue` ekranında yüzlerce öge (Item) "10 dakikadan fazladır gecikiyor" ise, sistem tıkanmış demektir.
- **Yanlış Teşhis:** Agent bozuk veya hat düştü sanırsınız.
- **Doğru Teşhis:** Server Poller (Ameleler) yetersiz kalmıştır. İş sayısı fazla, çalışan (Thread) eksiktir. Verileri zamanında sormaya yetişemezler. Hatta PostgreSQL tabanı gelen dataya yetişemediği için (DB Darboğazı) Ajanları bekletiyordur (Syncers).

## 2. İşçileri (Tuning Processes) Ayarlama Rehberi

`/etc/zabbix/zabbix_server.conf`

- **`StartPollers=100`:** En kritik işçiler. Pasif olan ajanlara, JMX ve DB izlemelerine giden sayıdır. (Zabbix'in kendi CPU'su için önerilen değer Core başına maks X-Y civarlarındadır. 500 gibi astronomik Pollers (Sorgu iplikçiği) yaratırsanız, binlerce sorgu kuyruğa girer, CPU Context-Switch yüzünden "Load Average" fırlar, Zabbix çöker). Doğrusu Pollers'ı aşırı artırmak yerine Agentları Aktif Mode (Active Agent) geçirmektir (Bkz: Modül 04).
- **`StartTrappers=20`:** Pasif yöntem kullanmadık, her şeyi Aktif ajanlarla Proxy'lerle PUSH yöntemiyle sunucuya itiyoruz dediniz; harika. İşte bu sefer veriyi tutacak 10051 portunda çok sayıda Trapper (Karşılayıcı) ihtiyacımız doğar. Aktif bağlantı sayınıza göre yavaş yavaş (5-10-20...) artırın.
- **`StartPingers=10`:** Zabbix binlerce Switch'e ICMP (Ping) atarken fping yazılımını kullanır. `Timeout` saniyeleri birikir. Sadece Ping bazlı bir NOC ekranı varsa (Ağ yönetimi ağırlıklı) Pingers değeri Poller'dan ayrılarak şişirilmelidir (10-20...).
- **`StartDBSyncers=8`:** Kuyruk yok ama Zabbix UI yavaş? Database Write kapasitesi iyi ama Disk yazmıyorsa Syncers (Kasadaki memurlar) az, Müşteriler (Veri) çoktur. History ve Trends RAM alanındaki verileri TimescaleDB'ye döken işçilerdir. Aşırı şişirmek SSD I/O'sunu felç eder, ideali 4-10 arası bırakmaktır.

## 3. Zabbix Internal İtemlar Kuralı

Mimaride, Zabbix "Kendini de İzleyen (Monitoring Itself)" bir uygulamadır. Tüm bu ayarları "Göz kararı" (Kör uçuş) ile yapmamalısınız.

`Template App Zabbix Server` template'inin oluşturduğu grafiklere (Dashboards -> Zabbix Server) bakmalısınız:

- Grafikte: `Zabbix server poller processes more than 75% busy` ibaresi belirdi!
- Anlamı: "Poller işçileri tam kapasite (Overload) çalışıyor, hiç boş vakitleri (Idle zamanları) yok!"
- Tedavi: `StartPollers` değeri acilen 5 artırılmalı ve Zabbix sızdırmazlık grafiğine ("Zabbix internal processes utilization") bakılarak o işçilerin yükünün %50'lere (Yeşil Çizgi) inmesi izlenmelidir. Grafiğe bak, Conf'u değiştir, Sonuca bak, bu 3'lü döngüyle Tuning yapılır!

## 4. Master Son Söz: Asla Korkutucu Alarm Eşikleri Bırakmayın

Bir mimariyi efsane yapan, %100 Uptime vermesinden ziyade "Gürültüsüz (Noise-Free) Uyarı" mekanizmasıdır. Zabbix'e atadığınız 20.000 veri sensörü (Items) arasından o gün sadece ve sadece 2 SMS geliyorsa, o 2 SMS gerçekten "Problem" SMS'idir. Bir IT Mimarının asıl uzmanlığı Mimarisine alarm kuralı (Trigger) eklediği an değil, artık gereksiz alarmları sile sile sisteme ekleyeceği yeni hiçbir alarm kuralı kalmadığı saniye taçlanır.

Tebrikler. Bu 22 Adımlı Master Eğitimini kavrayıp kurduğunuzda, en büyük Global Ölçekli bulut Datacenter ortamlarında dahi modern ve hatasız "Observability" yığınının tepesinde oturacaksınız demektir.

---
[Önceki Modül](./21_Raporlama.md) | [README'ye Dön](./README.md)

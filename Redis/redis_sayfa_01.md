# 📄 Sayfa 1: Çekirdek Mimari, SDS ve Hash Table Internals (Ultra-Detay)

Redis'in performansının arkasında sadece RAM kullanımı değil, veri yapılarının C düzeyindeki tasarımı yatar.

## 1. SDS (Simple Dynamic Strings) vs C Strings
Redis, standart C dizesi (`char*`) yerine **SDS** adı verilen özel bir yapı kullanır.
- **SDS Neden Var?**
    1. **O(1) Uzunluk Hesabı:** C dizesinde uzunluk bulmak `strlen` ile O(N) iken, SDS bunu bir `len` değişkeninde tutar (O(1)).
    2. **Binary Safe:** C dizeleri `\0` (null) karakteri ile biter. SDS ise uzunluğu bildiği için string içinde null karakteri (resim verisi gibi) barındırabilir.
    3. **Buffer Overflow Koruması:** SDS, kapasiteyi (`free`) takip eder ve belleği otomatik genişletir.

## 2. Hash Table ve Kademeli Rehash (Incremental Rehash)
Redis'in ana veri deposu devasa bir Hash Tablosudur. Veri sayısı arttığında tabloyu genişletmek (Rehash) gerekir.
- **Sorun:** 100 milyon anahtarı bir kerede yeni bir tabloya taşımak Redis'i saniyelerce dondurur.
- **Çözüm:** Redis **Incremental Rehash** yapar. Her okuma/yazma isteğinde verinin küçük bir parçası yeni tabloya taşınır. Böylece performans kaybı olmadan tablo büyür.

## 3. Redis Module API
Redis 4.0+, geliştiricilerin Redis'e yeni veri tipleri ve komutlar eklemesine izin veren bir **C API** sunar.
- **Neden Önemli?** Standart Redis'te olmayan (Örn: Yapay zeka modelleri çalıştırmak, özel veri yapıları) özellikleri Redis içine doğrudan entegre edebilirsiniz.

## 4. Olay Döngüsü (Event Loop) ve I/O Multiplexing
Redis, **Ae Library** adında kendi basit olay kütüphanesini kullanır. 
- **Epoll (Linux):** On binlerce bağlantıyı tek bir thread ile verimli şekilde dinler. 
- **Zamanlı Görevler (Time Events):** Veri silme (expiration) ve arka plan temizliği de bu döngü içinde sırasıyla yapılır.

> [!IMPORTANT]
> Redis'in hızı "Kilitsiz Mimari"dir. Her şey tek bir komuta odaklanır, böylece bağlam değiştirme (context switching) maliyeti sıfıra iner.


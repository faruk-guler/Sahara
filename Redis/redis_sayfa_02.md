# 📄 Sayfa 2: İleri Kurulum ve Sistem Optimizasyonu (Ultra-Detay)

Redis'in farklı ortamlarda ayağa kaldırılması ve çok işlemcili sunucularda en yüksek performansı vermesi için gereken tüm adımlar.

## 0. Temel Kurulum Yöntemleri

### A. Linux (Ubuntu/Debian) - Paket Yöneticisi
En hızlı ve standart yöntem:
```bash
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
```

### B. Docker ile Hızlı Kurulum
İzolasyon ve kolay yönetim için:
```bash
docker run --name my-redis -p 6379:6379 -d redis:7.2-alpine
```

### C. Kaynak Koddan Derleme (Expert Yöntem)
En güncel versiyon ve özel optimizasyonlar için:
```bash
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make # İşlemciye özel optimizasyon yapar
sudo make install
```

## 1. WSL2 Üzerinde Bellek ve I/O Yönetimi
Windows üzerinde Redis çalıştırırken WSL2'nin varsayılan kısıtlamalarını aşmanız gerekir.
- **`.wslconfig` Ayarları:**
    WSL2 varsayılan olarak Windows RAM'inin yarısını alabilir. `C:\Users\<Kullanıcı>\.wslconfig` dosyasına şunları ekleyin:
    ```ini
    [wsl2]
    memory=8GB  # Redis için yeterli alan
    swap=0      # Swap'ı kapatmak performansı uçurur (Redis Swap sevmez)
    ```
- **I/O Schedule:** WSL2 vhdx üzerinde çalıştığı için disk yazma (AOF) maliyeti Windows dosya sistemine takılabilir. Veri kalıcılığı kritikse Redis'i doğrudan Linux yerel klasöründe (`/home/` gibi) barındırın, `/mnt/c/` kullanmayın.

## 2. NUMA (Non-Uniform Memory Access) ve Affinity
Çok CPU'lu (Enterprise) sunucularda Redis'in hangi CPU çekirdeğinde çalışacağı önemlidir.
- **Sorun:** Redis bir CPU çekirdeğinde çalışırken, verisi başka bir CPU'nun yönettiği RAM bloklarındaysa "Memory Latency" artar.
- **Çözüm:** `taskset` veya `numactl` ile Redis sürecini belirli bir çekirdeğe (Affinity) bağlayın.
    ```bash
    numactl --cpunodebind=0 --membind=0 redis-server /etc/redis.conf
    ```

## 3. TCP Backlog ve Kernel Limitleri (Derin Bakış)
- **`net.core.somaxconn`:** Kernel'in kuyruğa alabileceği maksimum bağlantı sayısı. Redis `tcp-backlog` değeri ile bu değer uyumlu olmalıdır (Minimum 2048 önerilir).
- **`ulimit -n`:** Açık dosya sayısı limiti. Her istemci bir "file descriptor" açar. Redis'in binlerce istemciyi reddetmemesi için bu değer 65536 yapılmalıdır.

> [!TIP]
> `redis-server --version` komutu ile derleme zamanı (Build Time) özelliklerini, hangi `malloc` (jemalloc vs libc) kütüphanesini kullandığını görebilirsiniz. `jemalloc` parçalanmayı önlemek için Redis'in varsayılan tercihidir.


# 📄 Sayfa 11: Redis AI ve Vektör Veritabanı (Expert Edition)

Redis artık sadece anahtar-değer deposu değil, modern yapay zeka (AI) uygulamaları için bir **Vektör Veritabanı**'dır.

## 1. Vektör Arama (Vector Search) Nedir?
Geleneksel aramada "elma" kelimesini ararsınız. Vektör aramada ise elmanın "matematiksel temsili" (embedding) aranır. Bu sayede "kırmızı meyve" yazdığınızda Redis size "elma" sonucunu getirebilir.

- **Vektörler:** Genellikle 768 veya 1536 boyutlu ondalıklı sayı dizileridir.
- **Benzerlik Ölçümü:** İki vektör arasındaki açı (Cosine Similarity) veya mesafe (Euclidean) ölçülür.

## 2. İndeksleme Algoritmaları: HNSW ve FLAT
Redis Search (RediSearch 2.4+) iki ana indeksleme tipi sunar:
- **FLAT (Brute-force):** Tüm vektörleri tek tek karşılaştırır. %100 doğrudur ama yavaştır. Küçük veri kümeleri için uygundur.
- **HNSW (Hierarchical Navigable Small World):** Verileri "küçük bir dünya" grafiği şeklinde birbirine bağlar. Çok hızlıdır (milisaniyeler) ama yaklaşık sonuçlar (Approximate Nearest Neighbors - ANN) üretir.

## 3. Redis Üzerinde Vektör Sorgusu
Bir vektör indeksi oluşturmak:
```bash
FT.CREATE my_index ON HASH PREFIX 1 user: SCHEMA vec VECTOR HNSW 6 TYPE FLOAT32 DIM 128 DISTANCE_METRIC COSINE
```
Benzer vektörleri aramak (Vector Similarity Search - VSS):
```bash
FT.SEARCH my_index "*=>[KNN 10 @vec $query_vec]" PARAMS 2 query_vec "bytes_of_vector" DIALECT 2
```

## 4. Redis AI Modülü
RedisAI, derin öğrenme modellerini (TensorFlow, PyTorch, ONNX) doğrudan Redis içinde çalıştırmanızı sağlar.
- **Zero-copy:** Veri Redis'ten çıkmadan yapay zeka modeli tarafından işlenir.
- **Tensor Veri Tipi:** Redis'e `TENSOR.SET` gibi yeni komutlar ekleyerek matris işlemlerini sunucu tarafında yapar.

> [!IMPORTANT]
> **RAG (Retrieval Augmented Generation)** mimarisinde Redis, LLM'lerin (ChatGPT vb.) kendi verilerinize erişmesi için en performanslı "Hafıza" katmanıdır.


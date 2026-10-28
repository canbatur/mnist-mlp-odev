# MNIST ile MLP Tabanlı El Yazısı Rakam Tanıma

**Ad Soyad:** Muhammedcan Batur
**Öğrenci No:** 24120205059

Bu projede MNIST veri seti üzerinde, CNN kullanmadan ve hazır eğitilmiş model olmadan, sıfırdan eğitilen MLP modelleri geliştirdim. Farklı model konfigürasyonlarını karşılaştırdım ve kendi el yazımla yazdığım rakamları ve öğrenci numaramı modele verdim.

## Repo içeriği
- `main.ipynb`: tüm kodlar ve çıktılar (PyTorch)
- `0.jpeg … 9.jpeg`: kendi el yazımla (kurşun kalem, beyaz kâğıt) yazdığım rakamların fotoğrafları
- `ciktilar.jpeg`: 0–9 el yazı rakamlarının orijinal hali, MNIST formatına çevrilmiş hali ve model tahminleri
- `ciktilar1.jpeg`: öğrenci numaramın görselleri ve model tahmini
- `grafik.jpeg`: Deney 1 loss ve accuracy grafikleri

**Çalıştırma:** Defterde resimler `custom_digits/` klasöründen okunur. Çalıştırmak için `0.jpeg … 9.jpeg` dosyalarını `custom_digits/` adlı bir klasöre koymak gerekir.

## Veri seti ve ayrım
MNIST: 28×28 piksel, gri seviyeli, 10 sınıf (0–9).

| Küme | Örnek sayısı | Kullanım |
|---|---|---|
| Train | 54.000 | Model eğitimi |
| Validation | 6.000 | Epoch ve model seçimi (eğitim setinden rastgele ayrıldı) |
| Test | 10.000 | Yalnızca final değerlendirme, model geliştirmede kullanılmadı |

Giriş normalizasyonu: ortalama 0.1307, standart sapma 0.3081.

## Deney 1: Temel MLP

**Hiperparametreler**
- Mimari: 784 → 256 → 128 → 10 (2 hidden layer, ReLU)
- Loss: CrossEntropyLoss
- Optimizer: Adam, learning rate = 0.001
- Batch size = 128, epoch = 10, seed = 42

**Sonuçlar** (10. epoch)

| Training loss | Validation loss | Training accuracy | Validation accuracy | Test accuracy | Eğitim süresi |
|---|---|---|---|---|---|
| 0.0187 | 0.1067 | 0.9937 | 0.9748 | [...] | [...] sn |

Tablodaki loss ve accuracy değerleri son epoch'a aittir. Test accuracy, validation accuracy'si en yüksek olan epoch'taki modelle hesaplanmıştır.

**Grafikler:** `grafik.jpeg`

## Deney 2: Konfigürasyon karşılaştırması
Konfigürasyonlar yalnızca validation başarısına göre karşılaştırıldı. Test verisi seçimde kullanılmadı.

| Konfigürasyon | Train loss | Val loss | Train acc | Val acc |
|---|---|---|---|---|
| A: 256-128, Adam, lr=1e-3 (baz) | 0.0187 | 0.1067 | 0.9937 | 0.9748 |
| B: 64-32, Adam, lr=1e-3 (küçük) | 0.0471 | 0.1103 | 0.9850 | 0.9688 |
| C: 512-256-128, Adam, lr=1e-3 (derin/geniş) | 0.0182 | 0.1004 | 0.9939 | 0.9763 |
| D: 256-128, SGD (momentum 0.9), lr=0.01 | [...] | [...] | [...] | [...] |
| E: 256-128, Adam, lr=1e-3, dropout=0.3 | [...] | [...] | [...] | [...] |

Validation'a göre seçilen model: [...]
Seçilen modelin test accuracy'si (tek sefer hesaplandı): [...]

## Kendi el yazım

**Hazırlık:** Rakamları beyaz kâğıda kurşun kalemle yazıp fotoğrafını çektim, her rakamı ayrı ayrı kırptım. Görüntüleri MNIST formatına getirmek için şu adımları uyguladım: gri tonlama, gölge ve ışık düzeltme, küçük lekeleri temizleme, rakamı kırpıp 20×20'ye sığdırma, 28×28 içinde kütle merkezine göre ortalama, siyah zemin ve beyaz rakam hâline çevirme ve MNIST normalizasyonu.

**0–9 rakamları** (`ciktilar.jpeg`): 10 rakamın 8'ini doğru bildi, doğruluk **%80**.

| Rakam | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|---|
| Tahmin | 0 | 1 | 2 | 3 | **6** | 5 | 6 | 7 | 8 | **3** |
| Güven | %100 | %96 | %100 | %99 | %47 | %100 | %100 | %100 | %100 | %93 |

**Öğrenci numaram** (`ciktilar1.jpeg`):
- Gerçek numara: 24120205059
- Model tahmini: 26120205053
- Doğru haneler: **9 / 11** (4 yerine 6, 9 yerine 3 tahmin edildi)

## Genel değerlendirme

**Deney 1.** Model train verisinde %99,37 accuracy'ye ulaştı, validation accuracy'si ise %97,48'de kaldı. Aradaki yaklaşık 1,9 puanlık fark ve loss grafiği overfitting belirtisi veriyor: train loss düşmeye devam ederken validation loss 6. epoch civarında yaklaşık 0.074 ile en düşük değerine indi, sonra yükselerek 10. epoch'ta 0.1067'ye çıktı. Yani model sonraki epoch'larda eğitim verisini ezberlemeye başladı, genelleme iyileşmedi. Validation accuracy de 6. epoch'tan sonra %97,5–97,6 civarında sabit kaldı. Bu yüzden validation'a göre en iyi epoch'un modeli kullanıldı.

**Deney 2.** Küçük model (B, 64-32) hem train hem validation başarısında en geride kaldı (%96,88 validation), çünkü kapasitesi sınırlı. Daha geniş ve derin model (C) en yüksek validation başarısını (%97,63) verdi, fakat baz modelden (A, %97,48) fark çok küçük. Yani MNIST gibi bu kadar sade bir veri setinde modeli büyütmek az fayda sağlıyor. [D (SGD) ve E (dropout) sonuçlarını yorumla.]

**Gerçek el yazısı ve distribution shift.** Model MNIST validation setinde yaklaşık %97,5 başarılıyken kendi el yazımda 10 rakamın 8'ini, öğrenci numaramın 11 hanesinin 9'unu doğru bildi. Bu fark, MNIST'in dağılımı ile benim verim arasındaki farktan (distribution shift) kaynaklanıyor. MNIST'te rakamlar belirli bir yazı stiliyle, kalın çizgilerle ve ortalanmış olarak yazılmıştır. Benim yazım ise farklı: ince kurşun kalem çizgisi, fotoğraf ışığı ve gölgesi, kendi yazı stilim. Ön işleme (gölge düzeltme, kalınlaştırma, ortalama, normalizasyon) bu farkın büyük kısmını kapattı, çünkü rakamların çoğu yüksek güvenle doğru bilindi. Yanlış bilinen iki rakam:
- **4 → 6 (%47 güven):** Benim 4'üm üstü kapalı ve yan çizgili, MNIST'teki tipik 4'lerden belirgin biçimde farklı. Modelin güveninin düşük olması bu rakamı tanımadığını gösteriyor.
- **9 → 3 (%93 güven):** Burada model yanlış tahmini yüksek güvenle yaptı. 9'umun alt kuyruğu kıvrık ve gövdesi 3'e benziyor olabilir. Yüksek güvenle yanılmak, modelin kendi belirsizliğini iyi ölçmediğini gösteriyor.

Sonuç olarak MLP, MNIST test verisinde iyi genelleme yapsa da kendi el yazım gibi farklı bir dağılıma geçildiğinde başarısı düşüyor. Bunu iyileştirmek için kendi yazıma benzeyen verilerle eğitmek, veri artırma (augmentation) kullanmak veya daha dayanıklı mimariler denemek gerekir. Bu ödevde CNN yasak olduğu için bu yöntemlere girmedim.

# ARIMA Model Analizi - Kısa Özet Rapor

**Veri:** Apple (AAPL) Hisse Senedi (1980-2025, 45 yıl)  
**Model:** ARIMA(4, 0, 0)  
**Amaç:** Günlük hisse senedi getirilerini tahmin etmek

---

## En Önemli Bulgular

### İyi Haberler
- **11,685 günlük** veri analiz edildi
- **36 farklı model** test edildi, en iyisi seçildi
- Model **istatistiksel olarak geçerli** (tüm testlerden geçti)
- Residual (hatalar) rastgele ve bağımsız (white noise)

### Karma Sonuçlar
- **Sürekli getiri tahmini:** MAE bazında Zero modelle aynı performans (%0.02 fark)
- **Naive (Random Walk) beklenmedik şekilde çok kötü** (%31.8 daha yüksek MAE)
- **Basit Mean Reversion en iyi performansı gösterdi** (karmaşık ARIMA'dan daha iyi)
- **Yön tahmini:** %51.56 (istatistiksel olarak rastgeleden farksız, p=0.0682)
- Büyük piyasa hareketlerini yakalayamıyor

---

## Veri Seti

| Özellik | Değer |
|---------|-------|
| Toplam Gözlem | 11,685 gün |
| Tarih Aralığı | 12 Aralık 1980 - 26 Eylül 2025 |
| Eğitim Seti | 9,348 gün (%80) → 1980-2016 |
| Test Seti | 2,337 gün (%20) → 2016-2025 |
| Kullanılan Veri | Log getiriler ($ln(fiyat_t / fiyat_t-1))$ |

**Neden log getiri?**
- Fiyat yerine yüzdesel değişim
- İstatistiksel analiz için daha uygun
- Volatiliteyi daha iyi yansıtır

---

## Model Seçimi

### ARIMA(4, 0, 0) Nedir?

```
ARIMA(p, d, q)
  p = 4  → Son 4 günün getirisi kullanılır
  d = 0  → Veri zaten durağandır (fark alma gerek yoktur)
  q = 0  → Hata terimi kullanılmaz
```

**Basit Açıklama:**  
*"Bugünkü getiriyi tahmin etmek için son 4 günün getirilerine bakılır ve ağırlıklı ortalaması alınır."*

### Neden Bu Model Seçildi?

**Grid Search:** 36 farklı kombinasyon test edildi

| Sıra | Model | AIC | BIC |
|------|-------|-----|-----|
| 1 | ARIMA(4,0,0) | -39,349.78 | -39,306.92 |
| 2 | ARIMA(0,0,4) | -39,348.96 | -39,306.10 |
| 3 | ARIMA(5,0,0) | -39,348.94 | -39,298.94 |

**Kriter:** En düşük AIC/BIC = En iyi model

---

## Model Kalitesi Testleri

### Ljung-Box Q Testi (Artık Analizi)

**Soru:** Modelin hataları rastgele mi?

| Gecikme | Q-İstatistik | p-değeri | Sonuç |
|---------|--------------|----------|--------|
| 10 gün | 5.76 | 0.835 | Rastgele |
| 20 gün | 23.11 | 0.283 | Rastgele |
| 30 gün | 40.44 | 0.097 | Rastgele |

**Yorum:** Tüm p-değerleri > 0.05 → Artıklarda otokorelasyon **YOK**

### Görsel Tanılar (4 Panel)

1. **Artıklar Zaman Serisi:** 0 etrafında rastgele
2. **Histogram:** Normal dağılıma yakın
3. **ACF Plot:** Korelasyon yok
4. **Q-Q Plot:** Normal dağılım geçerli

**Sonuç:** Model istatistiksel olarak **sağlam ve geçerli**

---

## Tahmin Performansı (Test Seti: 2016-2025)

### Hata Metrikleri

| Metrik | Değer | Açıklama |
|--------|-------|----------|
| **MAE** | 0.012371 | Ortalama %1.24 hata |
| **RMSE** | 0.018310 | Tipik hata %1.83 |
| **MAPE** | Hesaplanamaz | Getiriler sıfıra çok yakın → metrik bozuluyor |

### En Önemli Metrik: Yönlü Doğruluk

```
Directional Accuracy = %51.56
```

**Ne anlama geliyor?**
- 2,337 günün 1,205'inde yön doğru tahmin edilmiştir
- "Yarın yukarı mı, aşağı mı?" sorusuna doğru cevap oranı
- **Yazı-tura (%50) sadece %1.56 daha iyi**

**İstatistiksel Test:**
- Binomial test p-değeri: 0.0682
- Sonuç: Bu fark **istatistiksel olarak anlamlı değildir** (p > 0.05)
- Yön tahmini performansı, istatistiksel olarak rastgele tahminden ayırt edilememektedir

### Baseline Karşılaştırma

| Model | MAE | Fark |
|-------|-----|------|
| Zero Model (tahmin=0) | 0.012378 | Baseline |
| ARIMA(4,0,0) | 0.012375 | %0.02 daha iyi |
| Mean Reversion | 0.012371 | %0.06 daha iyi |
| **Naive (Random Walk)** | 0.018148 | **%31.8 daha kötü** |

**Önemli Not:** Zero ve Mean Reversion gibi basit modeller, karmaşık ARIMA modeliyle neredeyse aynı performansı göstermektedir. Ancak Naive (Random Walk) modeli beklenmedik şekilde çok daha kötü performans sergilemektedir.

---

## Neden Yön Tahmini Bu Kadar Zor?

### 1. Etkin Piyasa Hipotezi (EMH)
- Hisse senedi fiyatları tüm bilgiyi yansıtır
- Geçmiş fiyatlardan yön tahmin etmek son derece zordur
- Piyasa "random walk" (rastgele yürüyüş) izler

**Not:** Yön tahmini zayıf olsa da, model getirilerin büyüklüğünü (MAE) naive modelden daha iyi tahmin etmektedir.

### 2. Yüksek Volatilite
- Günlük getiriler çok değişken
- Standart sapma: 0.018 (±%1.8 günlük dalgalanma)
- Signal-to-noise ratio düşük

### 3. Dış Faktörler
Model şunları içermemektedir:
- Kazanç raporları
- Fed faiz kararları
- Pandemi gibi şoklar
- Makro ekonomik veriler

### 4. Model Sınırlamaları
- Sadece fiyata bakılmaktadır (tek değişkenli)
- Doğrusal modeldir (non-linear ilişkiler dahil değildir)
- Sabit parametreler kullanılmaktadır (zamanla değişmemektedir)

---

## Alternatif Modeller Karşılaştırması (Rolling Forecast)

**Metodoloji:** Rolling 1-step ahead forecast yöntemi ile 9 farklı model test edilmiştir.

### Test Edilen Modeller ve Sonuçlar

| Model | MAE | RMSE | Yönlü Doğruluk | Naive'den Fark |
|-------|-----|------|----------------|----------------|
| **Mean Reversion** | 0.012371 | 0.01831 | 51.56% | +31.83% daha iyi |
| ARIMA(0,0,0) - White Noise | 0.012371 | 0.01831 | 51.56% | +31.83% daha iyi |
| ARIMA(1,0,1) - ARMA(1,1) | 0.012371 | 0.01831 | 51.56% | +31.83% daha iyi |
| ARIMA(0,0,1) - MA(1) | 0.012372 | 0.01831 | 51.52% | +31.83% daha iyi |
| ARIMA(1,0,0) - AR(1) | 0.012372 | 0.01831 | 51.52% | +31.83% daha iyi |
| ARIMA(2,0,0) - AR(2) | 0.012373 | 0.01831 | 51.48% | +31.82% daha iyi |
| ARIMA(4,0,0) - AR(4) | 0.012375 | 0.01831 | 51.52% | +31.81% daha iyi |
| Zero (No Change) | 0.012378 | 0.01833 | 3.94% | +31.79% daha iyi |
| **Naive (Random Walk)** | 0.018148 | 0.02674 | 45.70% | Baseline |

### Kritik Bulgular

1. **Naive (Random Walk) beklenmedik şekilde en kötü performansı gösterdi**
   - Diğer tüm modellerden %31.8 daha kötü
   - Yön tahmini rastgele tahminden bile daha kötü (%45.7 < %50)

2. **Basit modeller karmaşık modellerle aynı performansı gösterdi**
   - Mean Reversion (basit ortalama) en iyi performans
   - ARIMA varyasyonları arasında minimal fark var (<0.01%)
   - Karmaşıklık ekstra performans getirmedi

3. **Tüm modeller (Naive hariç) benzer yön doğruluğu sergiliyor**
   - %51.5 civarı (istatistiksel olarak anlamlı değil)
   - Zero modeli istisna (sadece %3.94 - tüm tahminler aynı yönde)

**Yorum:**  
Rolling forecast yöntemi ile modeller arasında performans farkları gözlemlendi. Ancak Naive dışındaki tüm modeller neredeyse identik performans göstermektedir (MAE farkı <0.01%). Bu, finansal getiri serisinin düşük tahmin edilebilirliğini göstermektedir. En şaşırtıcı bulgu: Basit ortalama (Mean Reversion) karmaşık ARIMA modellerinden daha iyi veya eşit performans göstermektedir.

---

## Görsel Bulgular

### Tahmin vs Gerçek (Test Seti)

**Gözlemler:**
1. Tahminler hep 0'a yakındır (model "yarın büyük değişim olmaz" demektedir)
2. Gerçek getiriler çok volatildir (±%20'ye varan hareketler)
3. Büyük hareketler kaçırılmaktadır (örn: 2020 COVID krizi)
4. Güven aralıkları çok geniştir (model emin değildir)

### Hata Dağılımı

- Ortalama hata: 0.000365 (neredeyse 0)
- Simetrik dağılım (çan eğrisi)
- Bazı uç değerler var (outliers)
- Çoğu hata küçük (±%5 aralığında)

---

## Pratik Öneriler

### Yapılmaması Gerekenler

1. **Yön tahmini için kullanılmamalıdır**  
   → %51.56 yön doğruluğu istatistiksel olarak anlamlı değildir
   
2. **Sadece ARIMA'ya güvenilmemelidir**  
   → Tek değişkenli modeller yön tahmini için yetersizdir
   
3. **Büyük bahisler oynanmamalıdır**  
   → Model büyük hareketleri tahmin edememektedir

### Yapılabilecek Şeyler

1. **Risk yönetimi için kullanılabilir**  
   → Volatilite tahmini için baseline olarak
   
2. **Portföy çeşitlendirmesi yapılmalıdır**  
   → Tek hisseye bağımlı kalınmamalıdır
   
3. **Diğer modeller eklenebilir**  
   → Makine öğrenmesi, sentiment analizi
   
4. **Uzun vadeli yatırım tercih edilmelidir**  
   → Günlük spekülasyon yerine

---

## Gelecek Çalışmalar İçin Öneriler

### 1. Model İyileştirmeleri

**GARCH Modelleri:**
```
ARIMA + GARCH kombinasyonu
```
- Volatilite kümelenmesini modelleyebilir
- Finansal zaman serileri için daha uygun

**Makine Öğrenmesi:**
- LSTM (Derin öğrenme)
- Random Forest
- XGBoost

### 2. Çok Değişkenli Yaklaşım

Modele eklenebilecekler:
- İşlem hacmi
- Teknik göstergeler (RSI, MACD)
- Haber sentiment'i
- VIX (korku endeksi)
- Faiz oranları

### 3. Farklı Zaman Dilimleri

```
Günlük → Haftalık → Aylık
```
- Gürültü azalır
- Tahmin kolaylaşır
- Signal-to-noise ratio artar

---

## Ana Mesajlar

### İstatistiksel Geçerlilik
Model sağlamdır, testlerden geçmiştir, metodoloji doğru uygulanmıştır.

### Karma Performans
- **Sürekli getiri tahmini:** MAE bazında naive modelden %0.06 daha iyi
- **Yön tahmini:** Zayıf (%51.56, istatistiksel olarak rastgeleden farksız)

### Yön Tahmini ve Büyüklük Tahmini Farklıdır
Model getirilerin büyüklüğünü tahmin etmede başarılı ancak yönünü tahmin etmede zayıftır.

### EMH ile Tutarlı
Geçmiş fiyatlar tek başına yön tahmini için yeterli bilgi vermemektedir.

### Karmaşıklık ≠ Performans
Basit ve karmaşık modeller benzer sonuçlar vermiştir.

### Çok Faktörlü Yaklaşım Gerekli
Sadece fiyat yerine, hacim + sentiment + makro veriler eklenmelidir.

---

## Ek Bilgiler

### Kullanılan Araçlar
- **Python 3.12.11** (Anaconda)
- **statsmodels 0.14.5** (ARIMA)
- **pandas 2.2.3** (Veri analizi)
- **scikit-learn 1.6.1** (Metrikler)
- **matplotlib** (Görselleştirme)

### Notebook
- **Dosya:** `filled_modelleme.ipynb`
- **Hücreler:** 22 (20 çalıştırılabilir)
- **Satır sayısı:** 414 satır kod + açıklama

### Veri
- **Kaynak:** `../data/filled_aapl.csv`
- **Şirket:** Apple Inc. (AAPL)
- **Süre:** 45 yıl (1980-2025)

---

## Sonuç

Bu çalışma ile **ARIMA modellerinin finansal zaman serileri tahminindeki sınırları** net bir şekilde gösterilmiştir. Model teknik olarak doğru ve geçerli olmasına rağmen, günlük hisse senedi hareketlerini tahmin etmek için **pratik değeri sınırlıdır**.

**Temel Çıkarım:**  
Finansal piyasalar tahmin edilenden **çok daha karmaşık ve rastgeledir**. Başarılı tahmin için:
- Çok değişkenli modeller
- Gelişmiş makine öğrenmesi
- Alternatif veri kaynakları (sentiment, haber)
- Risk yönetimi stratejileri

gerekmektedir.

---

**Önemli Uyarı:** Bu analiz akademik/eğitim amaçlıdır. Yatırım tavsiyesi değildir.

---

*Detaylı teknik rapor için: `ARIMA_MODEL_RAPORU.md`*

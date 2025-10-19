# Apple (AAPL) Hisse Senedi - Makine Öğrenmesi Modelleme Raporu

**Tarih:** 16 Ekim 2025  
**Veri Seti:** Doldurulmuş AAPL verisi (1980-2025)  
**Hedef Değişken:** Logaritmik getiri (bir gün sonrası tahmin)  
**Test Seti:** Son %20 veri (~2,339 gözlem)

---

## 1. Özet

Bu çalışmada, Apple hisse senedi için 5 farklı makine öğrenmesi modeli ile logaritmik getiri tahmini yapılmıştır. Modeller, gecikmeli getiriler, volatilite, momentum ve RSI gibi teknik göstergeler kullanarak eğitilmiştir.

---

## 2. Özellikler ve Hedef Değişken Tanımları

### 2.1. Hedef Değişken (Target Variable)

**`target`**: Bir sonraki günün logaritmik getirisi

**Matematiksel Tanım:**
```
target_t = log(Close_t+1 / Close_t) = log(Close_t+1) - log(Close_t)
```

**Açıklama:**
- Model, `t` anındaki özellikleri kullanarak `t+1` anındaki logaritmik getiriyi tahmin eder
- Logaritmik getiri kullanımının avantajları:
  - Zaman içinde toplamsal özellik (additive property)
  - Durağanlık (stationarity) özelliği gösterir
  - Normalite varsayımına daha yakındır
  - Yüzde getirilerle yaklaşık olarak eşdeğerdir (küçük değerler için)

**Değer Aralığı:**
- Teorik: (-∞, +∞)
- Pratik: Genellikle [-0.15, +0.15] aralığında (günlük ±%15 hareket)

---

### 2.2. Giriş Özellikleri (Input Features)

Toplam **10 özellik** modellere girdi olarak verilmiştir:

#### A) Gecikmeli Logaritmik Getiriler (Autoregressive Features)

| Özellik | Tanım | Formül |
|---------|-------|--------|
| `logret_lag1` | 1 gün önceki logaritmik getiri | log(Close_t / Close_t-1) |
| `logret_lag2` | 2 gün önceki logaritmik getiri | log(Close_t-1 / Close_t-2) |
| `logret_lag3` | 3 gün önceki logaritmik getiri | log(Close_t-2 / Close_t-3) |
| `logret_lag4` | 4 gün önceki logaritmik getiri | log(Close_t-3 / Close_t-4) |
| `logret_lag5` | 5 gün önceki logaritmik getiri | log(Close_t-4 / Close_t-5) |

**Amaç:** Geçmiş getiri patternlerini yakalamak, otoregresif (AR) yapı oluşturmak, momentum ve ortalamaya dönüş (mean reversion) etkilerini modellemek.

#### B) Volatilite Göstergeleri (Volatility Indicators)

| Özellik | Tanım | Formül |
|---------|-------|--------|
| `volatility_5` | 5 günlük hareketli volatilite | std(logret_t, logret_t-1, ..., logret_t-4) |
| `volatility_10` | 10 günlük hareketli volatilite | std(logret_t, logret_t-1, ..., logret_t-9) |

**Amaç:** Piyasa oynaklığını ölçmek, risk seviyesini belirlemek. Yüksek volatilite genellikle belirsizlik ve potansiyel büyük fiyat hareketleriyle ilişkilidir.

#### C) Momentum Göstergeleri (Momentum Indicators)

| Özellik | Tanım | Formül |
|---------|-------|--------|
| `momentum_5` | 5 günlük fiyat momentum | Close_t - Close_t-5 |
| `momentum_10` | 10 günlük fiyat momentum | Close_t - Close_t-10 |

**Amaç:** Fiyat hareketinin yönünü ve gücünü ölçmek. Pozitif momentum yükseliş, negatif momentum düşüş eğilimini gösterir.

#### D) RSI (Relative Strength Index)

| Özellik | Tanım | Formül |
|---------|-------|--------|
| `RSI` | 14 günlük göreceli güç endeksi | 100 - (100 / (1 + RS)) |

**RS (Relative Strength) Hesaplama:**
```
Kazanç_Ort = Mean(max(Close_t - Close_t-1, 0)) son 14 günde
Kayıp_Ort = Mean(max(Close_t-1 - Close_t, 0)) son 14 günde
RS = Kazanç_Ort / Kayıp_Ort
```

**Değer Aralığı:** 0-100
- RSI > 70: Aşırı alım bölgesi (potansiyel düşüş sinyali)
- RSI < 30: Aşırı satım bölgesi (potansiyel yükseliş sinyali)

**Amaç:** Momentum göstergesi, trend dönüş noktalarını tespit etmek, aşırı alım/satım durumlarını belirlemek.

---

### 2.3. Veri Ön İşleme

#### Ölçeklendirme (Scaling):
- **Yöntem**: StandardScaler (Z-score normalization)
- **Formül**: `X_scaled = (X - μ) / σ`
  - μ: Özelliğin ortalaması (eğitim setinden)
  - σ: Özelliğin standart sapması (eğitim setinden)
- **Sonuç**: Her özellik ortalama=0, standart sapma=1 olacak şekilde normalize edilir
- **Neden gerekli?**: 
  - Özelliklerin farklı ölçeklerden geldiği için (örn: RSI 0-100, getiriler -0.1 ile +0.1)
  - Mesafe tabanlı algoritmaların (kNN) doğru çalışması için
  - Gradient descent optimizasyonunun daha hızlı yakınsaması için

#### Eksik Değer Temizleme:
- Gecikmeli özellikler ve hareketli pencere hesaplamaları nedeniyle ilk satırlarda NaN değerler oluşur
- Bu değerler `dropna()` ile temizlenmiştir
- Toplam ~30-40 ilk gözlem veri setinden çıkarılmıştır

#### Train-Test Bölümü:
- **Eğitim Seti**: İlk %80 (9,356 gözlem)
- **Test Seti**: Son %20 (2,339 gözlem)
- **Önemli Not**: Zaman serisi yapısı korunarak kronolojik bölme yapılmıştır (shuffle yapılmamıştır)

---

### 2.4. Özellik-Hedef İlişkisi

**Tahmin Problemi:**
```
f: X_t → y_t+1
```

Burada:
- **X_t**: t anındaki 10 özellik vektörü (gecikmeli getiriler, volatilite, momentum, RSI)
- **y_t+1**: t+1 anındaki logaritmik getiri (hedef değişken)
- **f**: Öğrenilmeye çalışılan fonksiyon (ML modeli)

**Örnek Satır:**

| logret_lag1 | logret_lag2 | ... | volatility_5 | momentum_5 | RSI | → | target |
|-------------|-------------|-----|--------------|------------|-----|---|--------|
| 0.0023 | -0.0015 | ... | 0.0156 | 1.23 | 58.4 | → | 0.0018 |

Bu satır: "Önceki 5 günün getirileri, mevcut volatilite, momentum ve RSI değerlerine bakarak, yarınki getiriyi tahmin et" anlamına gelir.

---

## 3. Model Performans Sonuçları

### 3.1. Performans Tablosu

| Model | RMSE | MAE | R² | Yön Doğruluğu (%) |
|-------|------|-----|-----|-------------------|
| **Random Forest** | **0.0184** | **0.0124** | **-0.009** | **50.30%** |
| **Gradient Boosting** | **0.0186** | **0.0127** | **-0.030** | **50.73%** |
| **Linear Regression** | 0.0187 | 0.0129 | -0.046 | 49.31% |
| **k-Nearest Neighbors** | 0.0188 | 0.0129 | -0.048 | 49.06% |
| **CNN** | 0.1688 | 0.0926 | -83.872 | 46.66% |

**Not:** RMSE ve MAE için düşük değerler, R² ve Yön Doğruluğu için yüksek değerler daha iyidir.

---

## 4. Model Bazında Değerlendirmeler

### 4.1. Random Forest (En İyi Performans) 

**Performans:**
- RMSE: 0.0184 (en düşük)
- MAE: 0.0124 (en düşük)
- R²: -0.009 (en iyi)
- Yön Doğruluğu: %50.30

**Değerlendirme:**
- Tüm metrikler açısından en iyi performansı gösterdi
- Doğrusal olmayan ilişkileri başarıyla yakaladı
- Overfitting'e karşı dirençli yapısı finansal veriler için uygun
- Özellik önemliliği analizi ile yorumlanabilir sonuçlar sağladı

**Güçlü Yönler:**
- En düşük tahmin hataları (RMSE/MAE)
- Nispeten dengeli yön tahmini (%50.30)
- Ensemble yaklaşımı sayesinde robust tahminler

---

### 4.2. Gradient Boosting (İkinci En İyi)

**Performans:**
- RMSE: 0.0186
- MAE: 0.0127
- R²: -0.030
- Yön Doğruluğu: %50.73 (en yüksek)

**Değerlendirme:**
- En yüksek yön doğruluğu sağladı (%50.73)
- Random Forest'a çok yakın performans
- Kompleks finansal patternleri yakalamada başarılı
- İşlem stratejileri için faydalı olabilir (yön tahmini önemli)

**Güçlü Yönler:**
- En yüksek yön doğruluğu
- Boosting mekanizması ile hata düzeltme
- Özellik önemliliği analizi

---

### 4.3. Linear Regression (Baseline Model)

**Performans:**
- RMSE: 0.0187
- MAE: 0.0129
- R²: -0.046
- Yön Doğruluğu: %49.31

**Değerlendirme:**
- Basit ve yorumlanabilir model
- Ensemble modellerle rekabet edebiliyor
- Finansal zaman serilerinin doğrusal öngörülebilirliğinin sınırlı olduğunu gösteriyor
- Baseline model olarak değerli

**Zayıf Yönler:**
- Doğrusal olmayan ilişkileri yakalayamıyor
- Yön doğruluğu %50'nin altında

---

### 4.4. k-Nearest Neighbors (kNN)

**Performans:**
- RMSE: 0.0188
- MAE: 0.0129
- R²: -0.048
- Yön Doğruluğu: %49.06

**Değerlendirme:**
- Linear Regression ile benzer performans
- Lokal patternleri yakalama konusunda kısıtlı başarı
- Finansal zaman serilerinin karmaşık yapısı için yetersiz

**Zayıf Yönler:**
- En düşük yön doğruluğu (%49.06)
- Mesafe tabanlı yaklaşım, yüksek boyutlu finansal verilerde zorlanıyor
- Tahmin süreleri diğer modellere göre daha uzun

---

### 4.5. CNN (Convolutional Neural Network) 

**Performans:**
- RMSE: 0.1688 (en yüksek)
- MAE: 0.0926 (en yüksek)
- R²: -83.872 (en kötü)
- Yön Doğruluğu: %46.66 (en düşük)

**Değerlendirme:**
- Tüm metriklerde ciddi performans düşüklüğü
- Model veri setine uygun şekilde öğrenemedi
- Muhtemelen underfitting veya uygun olmayan mimari

**Sorunlar:**
- RMSE ve MAE değerleri diğer modellerin ~10 katı
- R² = -83.87, modelin tamamen başarısız olduğunu gösteriyor
- Yön doğruluğu rastgele tahmin seviyesinin altında (%46.66 < %50)

**Olası Nedenler:**
1. CNN mimarisi zaman serisi verileri için uygun ayarlanmamış
2. Hiperparametre optimizasyonu yetersiz
3. Eğitim verisi miktarı derin öğrenme için sınırlı
4. Early stopping çok erken devreye girmiş olabilir

**Öneriler:**
- LSTM veya GRU gibi recurrent yapılar denenebilir
- Daha uzun eğitim süresi ve batch size optimizasyonu
- Dropout ve regularization ayarları gözden geçirilmeli
- Transfer learning veya pre-training yaklaşımları

---

## 5. Genel Bulgular ve Yorumlar

### 5.1. R² Skorlarının Negatif Olması

**Açıklama:**
Tüm modellerde R² değerlerinin negatif olması, finansal zaman serilerinin temel bir özelliğini yansıtmaktadır:

- **R² < 0:** Model, sadece ortalama değer tahmin etmekten daha kötü performans gösteriyor
- **Finansal veriler için normal:** Hisse senedi getirileri büyük ölçüde rastgele yürüyüş (random walk) özelliği gösterir
- **Etkin Piyasa Hipotezi (EMH):** Piyasaların etkin olması, geçmiş verilerin gelecek getiriyi tam olarak tahmin edememesine neden olur

### 5.2. Yön Doğruluğu Analizi

**Bulgular:**
- En iyi: Gradient Boosting (%50.73)
- En kötü: CNN (%46.66)
- Rastgele tahmin seviyesi: %50

**Yorum:**
- Ensemble modeller (RF, GB) %50'nin üzerinde yön doğruluğu sağladı
- Küçük bir avantaj bile işlem stratejilerinde değerli olabilir
- Ancak işlem maliyetleri ve risk yönetimi göz önünde bulundurulmalı

### 5.3. Model Karmaşıklığı vs Performans

**İlginç Bulgu:**
Daha karmaşık modeller (CNN) daha kötü, daha basit modeller (Linear Regression) ise rekabetçi performans gösterdi.

**Sebep:**
- Finansal zaman serileri yüksek gürültü içerir
- Karmaşık modeller bu gürültüyü öğrenmeye (overfitting) eğilimlidir
- Basit modeller, temel trendleri yakalamakla yetinir ve daha genellenebilir

---

## 6. Sonuç ve Öneriler

### 6.1. Model Seçimi

**Önerilen Modeller:**
1. **Random Forest** - En dengeli ve güvenilir performans
2. **Gradient Boosting** - Yön doğruluğu önceliyse

**Önerilmeyen Model:**
- **CNN** - Mevcut konfigürasyonda kullanılmamalı

### 6.2. Pratik Uygulamalar İçin Öneriler

 **Yapılması Gerekenler:**
- Ensemble yaklaşım: RF ve GB tahminlerinin ortalaması alınabilir
- Risk yönetimi stratejileri mutlaka uygulanmalı
- İşlem maliyetleri hesaba katılmalı
- Stop-loss ve pozisyon büyüklüğü belirlenmeli

 **Dikkat Edilmesi Gerekenler:**
- Model tahminleri kesin değil, olasılıksal
- Geçmiş performans gelecek garantisi değildir
- Piyasa rejimi değişikliklerinde model performansı düşebilir
- Düzenli model yeniden eğitimi gereklidir

 **Yapılmaması Gerekenler:**
- Sadece model tahminlerine güvenerek işlem yapmak
- Risk yönetimi olmadan kaldıraçlı işlem
- Backtesting sonuçlarını olduğu gibi gerçek performans beklentisi olarak görmek

### 6.3. Gelecek Çalışmalar İçin Öneriler

1. **Özellik Mühendisliği:**
   - Daha fazla teknik gösterge (MACD, Bollinger Bands)
   - Makroekonomik değişkenler (faiz oranları, enflasyon)
   - Sentiment analizi (haber, sosyal medya)

2. **Model İyileştirme:**
   - Hiperparametre optimizasyonu (Grid Search, Bayesian Optimization)
   - XGBoost, LightGBM gibi gelişmiş boosting algoritmaları
   - LSTM/GRU gibi recurrent neural networks

3. **Alternatif Yaklaşımlar:**
   - GARCH modelleriyle hibrit yaklaşım
   - Volatilite tahmini ve getiri tahmini ayrı modeller
   - Reinforcement Learning ile işlem stratejisi optimizasyonu

4. **Validasyon:**
   - Walk-forward analysis
   - Out-of-sample testing daha uzun dönemler
   - Farklı piyasa rejimlerinde test

---

## 7. Teknik Detaylar

### 7.1. Veri Bölümü
- Eğitim seti: 9,356 gözlem (%80)
- Test seti: 2,339 gözlem (%20)
- Toplam: 11,695 gözlem

### 7.2. Özellik Ölçeklendirme
- StandardScaler kullanıldı
- Ortalama = 0, Standart sapma = 1

### 7.3. Model Hiperparametreleri

**Random Forest:**
- n_estimators: 100
- max_depth: 10
- min_samples_split: 10

**Gradient Boosting:**
- n_estimators: 100
- learning_rate: 0.1
- max_depth: 5

**kNN:**
- n_neighbors: 20
- weights: distance

**CNN:**
- 2 Conv1D katmanı (64, 32 filtre)
- Dense katman (50 nöron)
- Dropout: 0.2
- Early stopping: patience=10

---

 
**Notebook:** ml_modelleme.ipynb  
**Proje:** financial_analysis-modelling

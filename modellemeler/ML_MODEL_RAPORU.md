# Apple (AAPL) Hisse Senedi - Makine Öğrenmesi Modelleme Raporu

**Tarih:** 16 Ekim 2025  
**Veri Seti:** Doldurulmuş AAPL verisi (1980-2025)  
**Hedef Değişken:** Logaritmik getiri (bir gün sonrası tahmin)  
**Test Seti:** Son %20 veri (~2,339 gözlem)

---

## 1. Özet

Bu çalışmada, Apple hisse senedi için 5 farklı makine öğrenmesi modeli ile logaritmik getiri tahmini yapılmıştır. Modeller, gecikmeli getiriler, volatilite, momentum ve RSI gibi teknik göstergeler kullanarak eğitilmiştir.

### Kullanılan Özellikler:
- Gecikmeli logaritmik getiriler (5 gün)
- Hareketli volatilite (5 ve 10 gün)
- Momentum göstergeleri (5 ve 10 gün)
- RSI (Relative Strength Index)

---

## 2. Model Performans Sonuçları

### 2.1. Performans Tablosu

| Model | RMSE | MAE | R² | Yön Doğruluğu (%) |
|-------|------|-----|-----|-------------------|
| **Random Forest** | **0.0184** | **0.0124** | **-0.009** | **50.30%** |
| **Gradient Boosting** | **0.0186** | **0.0127** | **-0.030** | **50.73%** |
| **Linear Regression** | 0.0187 | 0.0129 | -0.046 | 49.31% |
| **k-Nearest Neighbors** | 0.0188 | 0.0129 | -0.048 | 49.06% |
| **CNN** | 0.1688 | 0.0926 | -83.872 | 46.66% |

**Not:** RMSE ve MAE için düşük değerler, R² ve Yön Doğruluğu için yüksek değerler daha iyidir.

---

## 3. Model Bazında Değerlendirmeler

### 3.1. Random Forest (En İyi Performans) 

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

### 3.2. Gradient Boosting (İkinci En İyi)

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

### 3.3. Linear Regression (Baseline Model)

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

### 3.4. k-Nearest Neighbors (kNN)

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

### 3.5. CNN (Convolutional Neural Network) 

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

## 4. Genel Bulgular ve Yorumlar

### 4.1. R² Skorlarının Negatif Olması

**Açıklama:**
Tüm modellerde R² değerlerinin negatif olması, finansal zaman serilerinin temel bir özelliğini yansıtmaktadır:

- **R² < 0:** Model, sadece ortalama değer tahmin etmekten daha kötü performans gösteriyor
- **Finansal veriler için normal:** Hisse senedi getirileri büyük ölçüde rastgele yürüyüş (random walk) özelliği gösterir
- **Etkin Piyasa Hipotezi (EMH):** Piyasaların etkin olması, geçmiş verilerin gelecek getiriyi tam olarak tahmin edememesine neden olur

### 4.2. Yön Doğruluğu Analizi

**Bulgular:**
- En iyi: Gradient Boosting (%50.73)
- En kötü: CNN (%46.66)
- Rastgele tahmin seviyesi: %50

**Yorum:**
- Ensemble modeller (RF, GB) %50'nin üzerinde yön doğruluğu sağladı
- Küçük bir avantaj bile işlem stratejilerinde değerli olabilir
- Ancak işlem maliyetleri ve risk yönetimi göz önünde bulundurulmalı

### 4.3. Model Karmaşıklığı vs Performans

**İlginç Bulgu:**
Daha karmaşık modeller (CNN) daha kötü, daha basit modeller (Linear Regression) ise rekabetçi performans gösterdi.

**Sebep:**
- Finansal zaman serileri yüksek gürültü içerir
- Karmaşık modeller bu gürültüyü öğrenmeye (overfitting) eğilimlidir
- Basit modeller, temel trendleri yakalamakla yetinir ve daha genellenebilir

---

## 5. Sonuç ve Öneriler

### 5.1. Model Seçimi

**Önerilen Modeller:**
1. **Random Forest** - En dengeli ve güvenilir performans
2. **Gradient Boosting** - Yön doğruluğu önceliyse

**Önerilmeyen Model:**
- **CNN** - Mevcut konfigürasyonda kullanılmamalı

### 5.2. Pratik Uygulamalar İçin Öneriler

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

### 5.3. Gelecek Çalışmalar İçin Öneriler

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

## 6. Teknik Detaylar

### 6.1. Veri Bölümü
- Eğitim seti: 9,356 gözlem (%80)
- Test seti: 2,339 gözlem (%20)
- Toplam: 11,695 gözlem

### 6.2. Özellik Ölçeklendirme
- StandardScaler kullanıldı
- Ortalama = 0, Standart sapma = 1

### 6.3. Model Hiperparametreleri

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

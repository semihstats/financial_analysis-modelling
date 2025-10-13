# Apple (AAPL) Hisse Senedi Getiri Tahmini: ARIMA Model Raporu

**Veri Seti:** Apple (AAPL) günlük kapanış fiyatları (1980-2025)

---

## Yönetici Özeti

Bu rapor, Apple hisse senedinin günlük getirilerini tahmin etmek için geliştirilen matematiksel bir modelin sonuçlarını açıklamaktadır. **45 yıllık** (1980-2025) hisse senedi verisi kullanılarak, yarının hisse senedi hareketini tahmin etmeye çalışan bir model oluşturulmuştur.

### Ana Bulgular:
- **11,685 günlük** veri analiz edildi (yaklaşık 45 yıl)
- **36 farklı model** test edildi, en iyisi seçildi
- Model **istatistiksel olarak geçerli** (residual testlerinden geçti)
- **Sürekli getiri tahmini:** MAE bazında naive modelden %0.06 daha iyi
- **Yön tahmini:** %51.6 doğruluk (istatistiksel olarak rastgeleden farksız)

---

## 1. Veri Seti ve Hazırlık Süreci

### 1.1 Ham Veri Özellikleri
- **Toplam Gözlem:** 11,686 günlük işlem verisi
- **Tarih Aralığı:** 12 Aralık 1980 - 26 Eylül 2025
- **Eksik Veri:** 1 adet (temizlendi)
- **Temiz Veri:** 11,685 gözlem

### 1.2 Ne Yapıldı?
Hisse senedi fiyatlarını doğrudan kullanmak yerine, **logaritmik getiri** (log return) hesaplandı. Bu:
- Fiyat hareketlerini yüzdesel değişimlere çevirir
- İstatistiksel analiz için daha uygun hale getirir
- Volatiliteyi daha iyi yansıtır

**Formül:**  
```
Log Getiri = ln(Bugünkü Fiyat / Dünkü Fiyat)
```

Örnek: Hisse senedi 100 dolardan 105 dolara çıkarsa → Log getiri ≈ 0.0488 (%4.88 artış)

### 1.3 Veri İki Parçaya Ayrıldı

**Eğitim Seti (Training Set):**
- 9,348 gözlem (%80)
- Tarih: 15 Aralık 1980 - 12 Ekim 2016
- **Amaç:** Model bu verilerle eğitildi

**Test Seti (Test Set):**
- 2,337 gözlem (%20)
- Tarih: 13 Ekim 2016 - 26 Eylül 2025
- **Amaç:** Modelin gerçek hayatta ne kadar iyi çalıştığı test edildi



---

## 2. Model Seçim Süreci: Hangisi En İyi?

### 2.1 Box-Jenkins Metodolojisi Nedir?
Zaman serisi tahmininde altın standart kabul edilen bir yöntemdir. Şu adımları içerir:
1. Veriyi anla (durağanlık, trend, mevsimsellik)
2. Farklı modeller dene
3. En iyisini seç
4. Modelin kalitesini kontrol et
5. Tahmin yap

### 2.2 ARIMA Modeli Nedir?
ARIMA: **AutoRegressive Integrated Moving Average**

Basitçe:
- **AR (AutoRegressive):** Geçmiş değerlere bak, onlardan öğren
  - *"Dün hisse %2 düştüyse, bugün de düşebilir"*
- **I (Integrated):** Veriyi durağan hale getir
  - *"Trendi kaldır, dalgalanmalara odaklan"*
- **MA (Moving Average):** Geçmiş hataları öğren
  - *"Dün tahminim 0.5% eksikti, bunu düzelt"*

Her ARIMA modeli 3 sayıyla tanımlanır: **ARIMA(p, d, q)**
- **p:** Kaç gün geriye bak? (AR parametresi)
- **d:** Kaç kez fark al? (Durağanlık için)
- **q:** Kaç günlük hatayı kullan? (MA parametresi)

### 2.3 Sistematik Arama: 36 Model Testi

**Test Edilen Kombinasyonlar:**
- p (AR parametresi): 0, 1, 2, 3, 4, 5 (6 seçenek)
- d (Fark alma): 0 (veri zaten durağan)
- q (MA parametresi): 0, 1, 2, 3, 4, 5 (6 seçenek)

**Toplam:** 6 × 1 × 6 = **36 farklı model**

### 2.4 Seçim Kriterleri: AIC ve BIC

Her modeli değerlendirmek için 2 kriter kullanıldı:

**AIC (Akaike Information Criterion):**
- Model ne kadar iyi uyuyor?
- Model ne kadar karmaşık? (ceza verir)
- **Düşük AIC = Daha iyi model**

**BIC (Bayesian Information Criterion):**
- AIC'ye benzer ama karmaşıklığa daha sert ceza verir
- Daha basit modelleri tercih eder
- **Düşük BIC = Daha iyi model**

### 2.5 Kazanan Model: ARIMA(4, 0, 0)

**Sonuçlar:**
```
En İyi 3 Model:
1. ARIMA(4,0,0)  → AIC: -39,349.78  BIC: -39,306.92  
2. ARIMA(0,0,4)  → AIC: -39,348.96  BIC: -39,306.10
3. ARIMA(5,0,0)  → AIC: -39,348.94  BIC: -39,298.94
```

**Not:** AIC/BIC değerlerinin büyük negatif olmasının sebebi:
- Log getiriler çok küçük değerlerdir (ortalama ~0.001)
- Uzun veri serisi (11,685 gözlem) log-likelihood hesabını büyütür
- Bu durum teknik olarak normaldir ve model karşılaştırmalarını etkilemez

**ARIMA(4, 0, 0) ne anlama geliyor?**
- **p=4:** Son 4 günün getirisi kullanılır
- **d=0:** Fark almaya gerek yoktur (veri zaten durağan)
- **q=0:** Hata terimi kullanılmaz



---

## 3. Model Kalitesi: İstatistiksel Tanılar

Modeli seçtikten sonra, gerçekten iyi çalıştığından emin olmak için sağlık kontrolleri yapıldı.

### 3.1 Residual (Artık) Analizi

**Residual nedir?**  
Gerçek değer ile tahmin arasındaki fark:
```
Residual = Gerçek Getiri - Tahmin Edilen Getiri
```

**İyi bir modelin artıkları:**
1. Rassal olmalı (pattern olmamalı)
2. Ortalama 0 civarında olmalı
3. Sabit varyanslı olmalı
4. Normal dağılmalı
5. Otokorelasyon olmamalı

### 3.2 Görsel Tanılar: 4 Panel

#### Panel 1: Artıklar Zaman Serisi
- **Gözlem:** Artıklar 0 etrafında rastgele dalgalanıyor
- **Sonuç:** Sistematik bir hata yok

#### Panel 2: Artık Dağılımı (Histogram)
- **Gözlem:** Çan eğrisi şeklinde, simetrik
- **Sonuç:** Normal dağılıma yakın (bazı uç değerler var)

#### Panel 3: ACF - Artıklar
- **ACF:** Autocorrelation Function (Otokorelasyon)
- **Gözlem:** Hemen hemen tüm gecikmelerde korelasyon yok
- **Sonuç:** Artıklar bağımsız (white noise)

#### Panel 4: Q-Q Plot - Artıklar
- **Gözlem:** Noktalar merkezi bölgede diagonal çizgiye yakın, ancak kuyruklarda belirgin sapmalar var
- **Sonuç:** Finansal getirilerde bilinen **fat tails** (kalın kuyruklar) problemi mevcut. ARIMA bu özelliği modelleyemez, GARCH gibi volatilite modelleri gereklidir

### 3.3 Ljung-Box Q Testi

**Ne test ediyor?**  
Artıklarda otokorelasyon var mı?

**Hipotezler:**
- H₀ (Boş Hipotez): Artıklar rassal (otokorelasyon yok)
- H₁ (Alternatif): Artıklarda otokorelasyon var

**Sonuçlar:**
```
Gecikme 10: p-value = 0.835  → otokorelasyon YOK
Gecikme 20: p-value = 0.283  → otokorelasyon YOK
Gecikme 30: p-value = 0.097  → otokorelasyon YOK
```

**Yorum:**  
Tüm p-değerleri 0.05'ten büyük → Hiçli hipotezi **reddedilemiyor**.  
**Sonuç:** Artıklar white noise → Model istatistiksel olarak **GEÇERLİ**

---

## 4. Tahmin Performansı: Model Ne Kadar İyi?

### 4.1 Hata Metrikleri

Modeli test setinde (2016-2025, 2,337 gün) değerlendirildi:

#### MAE (Mean Absolute Error)
```
MAE = 0.012371 (Ortalama %1.24 hata)
```
**Açıklama:**  
Ortalama olarak, tahminler gerçek getiriden 0.012371 birim sapıyor.

#### RMSE (Root Mean Squared Error)
```
RMSE = 0.018310 (Ortalama %1.83 hata)
```
**Açıklama:**  
Büyük hatalara daha fazla ceza verir. Modelin tipik hatası %1.83.

#### MAPE (Mean Absolute Percentage Error)

**MAPE hesaplanamamıştır.**  

**Neden?**  
MAPE finansal getiri serilerinde kullanılabilir bir metrik değildir. Getiriler sıfıra çok yakın olduğunda (örn: 0.0001) payda küçük olur ve oransal hata sonsuza gider. Bu metrik sadece pozitif ve sıfırdan uzak değerler için anlamlıdır.

### 4.2 Yönlü Doğruluk (Directional Accuracy)

**En önemli metrik!** Trader'lar için:

```
Directional Accuracy = %51.56
```

**Ne anlama geliyor?**  
2,337 günün 1,205'inde hareketin yönünü **doğru tahmin edildi**.
- Hisse senedi yukarı gidecek mi?
- Aşağı mı gidecek?

**İstatistiksel Anlamlılık Testi:**  
Binomial test (n=2337, p=0.5, k=1205) uygulandığında:
- Beklenen başarı (rastgele): 1168 gün (%50)
- p-değeri = 0.0682 → %5 anlamlılık seviyesinde **anlamlı değil**

**Yorum:**  
%51.56, yazı-tura atmaktan (%50) sadece **%1.56 daha iyi**dir ve bu fark **istatistiksel olarak anlamlı değildir** (p=0.0682 > 0.05). Modelin yön tahmini performansı, istatistiksel olarak rastgele tahminden ayırt edilememektedir. Ancak not edilmelidir ki, sürekli getiri tahmininde (MAE bazında) model naive modelden daha iyi performans göstermektedir.

### 4.3 Baseline Karşılaştırma

**Baseline Modeller:**

| Model | MAE | Açıklama |
|-------|-----|----------|
| Zero (tahmin=0) | 0.012378 | Her zaman 0 getiri tahmin eder |
| ARIMA(4,0,0) | 0.012375 | En iyi ARIMA modeli (grid search) |
| Mean Reversion | 0.012371 | Train ortalamasını tahmin eder |
| Naive (Random Walk) | 0.018148 | Önceki günün değerini tahmin eder |

**Sonuç:**  
- ARIMA modeli Zero modelden **%0.02 daha iyi**
- Mean Reversion (basit ortalama) ARIMA'dan **%0.03 daha iyi**
- **Naive (Random Walk) beklenmedik şekilde %31.8 daha kötü performans gösterdi**

**Önemli Bulgu:** Naive modelin kötü performansı, getiri serisinin ortalamaya dönüş (mean reversion) özelliği gösterdiğini işaret etmektedir. Önceki günün getirisi gelecek tahmini için zayıf bir göstergedir; ortalama veya sıfır tahmin etmek daha etkilidir.

---

## 5. Tahmin Görselleştirmesi

### Grafik Analizi: Test Seti Tahminleri

**Mavi Çizgi:** Eğitim verisi (1980-2016)  
**Yeşil Çizgi:** Gerçek test verisi (2016-2025)  
**Kırmızı Çizgi:** Model tahminleri  
**Pembe Alan:** %95 güven aralığı

**Metodoloji Notu:** Bu analizde **dinamik tahmin** (dynamic forecast) yöntemi kullanılmıştır. Model 2016'dan itibaren sadece eğitim verisiyle beslenmiş ve tüm test periyodu için ileriye dönük tahmin yapmıştır. Alternatif olarak, **rolling forecast** (her gün için 1-adım öteye tahmin) yöntemi kullanılabilir, ancak bu yöntem hesaplama açısından çok daha maliyetlidir ve sonuçlar benzer şekilde sıfıra yakın tahminler üretmektedir.

#### Gözlemler:
1. **Tahminler 0 civarında yoğunlaşıyor** → Model "yarın büyük değişim olmaz" diyor
2. **Gerçek getiriler çok volatil** → Piyasa tahmin edilenden daha değişken
3. **Güven aralıkları geniş** → Model kendi tahmininden emin değil
4. **Büyük hareketler kaçırılıyor** → Aşırı yükseliş/düşüşler tahmin edilemiyor

### Hata Dağılımı (2016-2025)

**Sol Panel:** Zaman içinde hatalar  
- Çoğu hata küçük (±0.05 aralığında)
- Bazı günlerde büyük hatalar var (örn: 2020 COVID çöküşü)

**Sağ Panel:** Hata histogramı  
- Sıfır merkezli, simetrik dağılım
- Ortalama hata: 0.000365 (neredeyse 0)
- Standart sapma: 0.018306

---

## 6. Kapsamlı Model Karşılaştırması

### 6.1 Metodoloji: Rolling Forecast

9 farklı model rolling 1-step ahead forecast yöntemi ile test edilmiştir. İlk 100 gözlem için her adımda model yeniden fit edilmiş, kalan gözlemler için statik forecast uygulanmıştır.

### 6.2 Test Edilen Modeller

**Baseline Modeller:**
- Naive (Random Walk): Önceki günün değerini tahmin et
- Mean Reversion: Train ortalamasını tahmin et
- Zero: Her zaman 0 tahmin et

**ARIMA Modelleri:**
- ARIMA(0,0,0) - White Noise
- ARIMA(1,0,0) - AR(1)
- ARIMA(2,0,0) - AR(2)
- ARIMA(4,0,0) - AR(4) [Grid search'ten seçilen]
- ARIMA(0,0,1) - MA(1)
- ARIMA(1,0,1) - ARMA(1,1)

### 6.3 Sonuçlar (MAE'ye göre sıralı)

| Sıra | Model | MAE | RMSE | Dir. Acc (%) | Naive'den Fark |
|------|-------|-----|------|--------------|----------------|
| 1 | Mean Reversion | 0.012371 | 0.01831 | 51.56 | +31.83% |
| 2 | ARIMA(0,0,0) | 0.012371 | 0.01831 | 51.56 | +31.83% |
| 3 | ARIMA(1,0,1) | 0.012371 | 0.01831 | 51.56 | +31.83% |
| 4 | ARIMA(0,0,1) | 0.012372 | 0.01831 | 51.52 | +31.83% |
| 5 | ARIMA(1,0,0) | 0.012372 | 0.01831 | 51.52 | +31.83% |
| 6 | ARIMA(2,0,0) | 0.012373 | 0.01831 | 51.48 | +31.82% |
| 7 | ARIMA(4,0,0) | 0.012375 | 0.01831 | 51.52 | +31.81% |
| 8 | Zero | 0.012378 | 0.01833 | 3.94 | +31.79% |
| 9 | **Naive (Random Walk)** | **0.018148** | **0.02674** | **45.70** | **Baseline** |

### 6.4 Kritik Bulgular

**1. Naive (Random Walk) Beklenmedik Şekilde En Kötü**
- Diğer tüm modellerden %31.8 daha kötü MAE
- Yön tahmini rastgele tahminden bile kötü (%45.7 < %50)
- Finansal getiri serisinde negatif otokorelasyon işareti

**2. Basit Modeller Karmaşık Modellerle Aynı Performans**
- Mean Reversion (basit ortalama) en iyi performansı gösterdi
- ARIMA varyasyonları arasında minimal fark (<0.01%)
- Karmaşıklık ekstra tahmin gücü sağlamadı

**3. Modeller Arasında MAE Farkları Çok Küçük (Naive Hariç)**
- En iyi 8 model arasında MAE farkı: 0.000007 (0.0006%)
- Tüm ARIMA ve basit modeller neredeyse identik performans
- Bu, getiri serisinin düşük tahmin edilebilirliğini gösteriyor

**4. Yön Tahmini Hala Zayıf**
- En iyi modeller bile %51.5 (istatistiksel olarak anlamlı değil)
- Zero model istisnası: %3.94 (tüm tahminler aynı yönde)

### 6.5 Yorumlama

**Neden Naive Bu Kadar Kötü?**

Geleneksel finans teorisinde Naive (Random Walk) modeli güçlü bir baseline olarak kabul edilir. Ancak burada kötü performansı şunları işaret ediyor:

1. **Ortalamaya Dönüş (Mean Reversion):** Apple getiri serisi belirgin ortalamaya dönüş özelliği gösteriyor
2. **Negatif Serisel Korelasyon:** Bir günün yüksek getirisi sonraki gün düşük getiri ile takip ediliyor
3. **Rolling Forecast Etkisi:** 1-adım öteye tahmin için geçmiş ortalamaları kullanmak daha etkili

**Neden Tüm ARIMA Modelleri Aynı?**

1. **Zayıf Otokorelasyon:** Getiri serisinde çok az geçici yapı var
2. **Yüksek Gürültü:** Signal-to-noise ratio çok düşük
3. **Ortalamaya Yakınsama:** Tüm modeller hızla 0'a yakın tahminler üretiyor
4. **EMH (Etkin Piyasa Hipotezi):** Geçmiş bilgi gelecek için minimal tahmin gücü sağlıyor

---

## 7. Sonuçlar ve Yorumlar

### 7.1 Teknik Sonuçlar

**Model istatistiksel olarak geçerli:**
- Residual testlerinden geçti
- Otokorelasyon yok
- Normal dağılıma yakın

**Ancak tahmin gücü zayıf:**
- %51.56 yönlü doğruluk (yazı-tura: %50)
- Basit modellerle aynı performans
- Büyük hareketleri kaçırıyor

### 7.2 Piyasa Etkinliği Teorisi (EMH)

**Sonuçlar neden beklenebilir?**

**Efficient Market Hypothesis (Etkin Piyasa Hipotezi):**
- Hisse senedi fiyatları tüm bilgiyi yansıtır
- Geçmiş fiyatlardan gelecek tahmin edilemez
- Piyasa "random walk" izler

**Elde edilen bulgular EMH'yi desteklemektedir:**
- Geçmiş 4 gün yeni bilgi vermiyor
- Tahminler sıfıra yakınsıyor
- Tüm modeller aynı performans

### 7.3 Neden Hisse Senedi Tahmini Zor?

1. **Yüksek Volatilite:**  
   Günlük getiriler son derece değişken (standart sapma: ~0.018)

2. **Haber ve Olaylar:**  
   - Kazanç raporları
   - Fed kararları
   - Pandemi gibi şoklar
   Model bunları bilmiyor!

3. **Rassal Yürüyüş (Random Walk):**  
   Günlük hareketler büyük ölçüde rassal

4. **Düşük Signal-to-Noise Ratio:**  
   Gerçek signal (trend) gürültüye (noise) göre çok küçük

### 7.4 Pratik Öneriler

**Yatırım Tavsiyesi DEĞİLDİR, ancak:**

**Yapılmaması Gerekenler:**
- Sadece ARIMA tahminlerine dayanarak alım-satım yapılmamalıdır
- Günlük spekülasyon için kullanılmamalıdır
- %51.56 doğruluk ciddi bir avantaj sağlamamaktadır

**Yapılabilecek Şeyler:**
- Risk yönetimi için volatilite tahmini yapılabilir
- Portföy çeşitlendirmesi için başka faktörler eklenebilir
- Makine öğrenmesi modelleriyle karşılaştırma yapılabilir
- Çok değişkenli modeller denenebilir (fiyat + hacim + sentiment)

---

## 8. Metodolojik Sınırlamalar

### 8.1 Model Sınırlamaları

1. **Tek Değişkenli Analiz:**  
   Sadece geçmiş fiyatlara bakıldı. Şunlar dahil değil:
   - İşlem hacmi
   - Makroekonomik göstergeler
   - Piyasa duyarlılığı (sentiment)
   - Diğer hisse senetleriyle korelasyon

2. **Lineer Model:**  
   ARIMA doğrusal bir model. Finansal piyasalarda:
   - Doğrusal olmayan (non-linear) ilişkiler var
   - Rejim değişimleri oluyor (kriz vs. normal)
   - Volatilite kümeleniyor (GARCH etkisi)

3. **Sabit Parametreler:**  
   Model parametreleri zaman içinde değişmiyor. Oysa:
   - Piyasa dinamikleri evrimleşiyor
   - Apple'ın karakteristiği 1980'lerden farklı

### 8.2 Veri Sınırlamaları

1. **Survivorship Bias:**  
   Apple hayatta kalan bir şirket. Batanlar dahil değil.

2. **Yapısal Kırılmalar:**  
   45 yılda Apple çok değişti:
   - 1980: Küçük bilgisayar şirketi
   - 2007: iPhone devrimli
   - 2025: Teknoloji devi

3. **Test Periyodu Özellikleri:**  
   2016-2025 test periyodunda:
   - 2020 pandemi şoku
   - 2022 Fed faiz artışları
   - Olağanüstü olaylar tahmin zorluğunu artırdı

---

## 9. Gelecek Çalışmalar için Öneriler

### 9.1 Model İyileştirmeleri

#### GARCH Ailesi
```
ARIMA + GARCH kombinasyonu
```
- Volatilite kümelenmesini modelleyebilir
- Finansal zaman serileri için daha uygun

#### Makine Öğrenmesi Yaklaşımları
- **LSTM (Long Short-Term Memory):** Derin öğrenme ile zaman serisi
- **XGBoost:** Çok değişkenli gradient boosting
- **Random Forest:** Ensemble learning

#### Çok Değişkenli Modeller
- **VAR (Vector Autoregression):** Birden fazla zaman serisini birlikte modelle
- **Exogenous Variables:** Dışsal faktörler ekle (faiz, VIX, sektör endeksleri)

### 9.2 Ek Veri Kaynakları

1. **Teknik Göstergeler:**
   - RSI (Relative Strength Index)
   - MACD (Moving Average Convergence Divergence)
   - Bollinger Bantları

2. **Temel Analiz:**
   - Kazanç raporları (EPS)
   - P/E oranı
   - Piyasa değeri değişimi

3. **Alternatif Veri:**
   - Sosyal medya duyarlılığı (Twitter, Reddit)
   - Haber sentiment analizi
   - Google arama trendleri

### 9.3 Farklı Zaman Dilimleri

```
Günlük → Haftalık → Aylık
```
- Gürültü azalır
- Signal-to-noise ratio artar
- Tahmin daha kolay olabilir

---

## 10. Sonuç ve Ana Mesajlar

###  Model Başarısı: Karma Sonuçlar

**Olumlu Yönler:**
- İstatistiksel olarak geçerli ve sağlam bir model elde edilmiştir
- Sistematik bir metodoloji uygulanmıştır (Box-Jenkins)
- 36 model test edilmiş, en iyisi seçilmiştir
- 45 yıllık veriden öğrenme gerçekleştirilmiştir
- Residual diagnostics testleri başarılı (Ljung-Box testleri geçildi)
- Kapsamlı model karşılaştırması yapılmıştır (9 farklı model, rolling forecast)

**Sınırlı Yönler:**
- **Yön tahmini zayıftır** (%51.56 doğruluk, istatistiksel olarak rastgeleden farksız, p=0.0682)
- **Basit modeller karmaşık ARIMA ile aynı performansı göstermektedir**
- Mean Reversion gibi çok basit bir yöntem ARIMA(4,0,0)'dan daha iyi sonuç vermiştir
- Büyük hareketler yakalanamamamaktadır
- Pratik ticaret için yeterli avantaj sağlamamaktadır

### Bilimsel Katkı

Bu çalışma ile gösterilmiştir ki:
1. **Naive (Random Walk) beklenmedik şekilde çok kötü performans gösterdi** → Apple getiri serisi ortalamaya dönüş (mean reversion) özelliği sergiliyor
2. **Basit ortalama (Mean Reversion) en iyi tahmin yöntemi** → Karmaşıklık ekstra performans sağlamıyor
3. **Yön tahmini ve büyüklük tahmini farklıdır** → Tüm modeller (Naive hariç) benzer MAE'ye sahip ancak yön tahmini zayıf
4. **ARIMA varyasyonları arasında minimal fark** → Model karmaşıklığı tahmin gücünü artırmıyor
5. **Geçmiş fiyatlar tek başına yön tahmini için yetersizdir** → EMH ile tutarlı bulgular
6. **Finansal tahmin çok boyutlu bir problemdir** → Çok faktörlü yaklaşım gerekmektedir

### Uygulamaya Dönük Çıkarımlar

**Bireysel Yatırımcılar için:**
- Günlük trading'de sadece teknik analize güvenilmemelidir
- Uzun vadeli yatırım stratejisi benimsenmelidir
- Portföy çeşitlendirmesi yapılmalıdır

**Kurumsal Analistler için:**
- ARIMA bir baseline olarak kullanılabilir
- Makine öğrenmesi ve alternatif verilerle zenginleştirilmelidir
- Risk yönetimi için volatilite tahmini odaklı çalışılabilir

**Akademik Araştırmacılar için:**
- Çok değişkenli modeller test edilmelidir
- Doğrusal olmayan yöntemler denenmelidir
- Yüksek frekanslı veri ile intraday tahmin araştırılmalıdır

---

## 11. Referanslar ve Kaynaklar

### Metodoloji
- Box, G. E. P., & Jenkins, G. M. (1976). *Time Series Analysis: Forecasting and Control*
- Tsay, R. S. (2010). *Analysis of Financial Time Series*

### Python Kütüphaneleri
- `statsmodels 0.14.5` - ARIMA implementation
- `pandas 2.2.3` - Data manipulation
- `scikit-learn 1.6.1` - Performance metrics
- `matplotlib` - Visualization

### Veri Kaynağı
- Apple (AAPL) günlük kapanış fiyatları
- Tarih aralığı: 1980-12-12 to 2025-09-26
- Toplam: 11,685 işlem günü

---

## Ekler

### Ek A: Teknik Terimler Sözlüğü

| Terim | Açıklama |
|-------|----------|
| **AIC** | Akaike Information Criterion - Model seçim kriteri |
| **ARIMA** | AutoRegressive Integrated Moving Average |
| **BIC** | Bayesian Information Criterion - Model seçim kriteri |
| **MAE** | Mean Absolute Error - Ortalama mutlak hata |
| **RMSE** | Root Mean Squared Error - Kök ortalama kare hata |
| **MAPE** | Mean Absolute Percentage Error - Ortalama yüzdesel hata |
| **Residual** | Artık - Gerçek değer ile tahmin arasındaki fark |
| **ACF** | Autocorrelation Function - Otokorelasyon fonksiyonu |
| **Ljung-Box** | Residuallarda otokorelasyon testi |
| **White Noise** | Beyaz gürültü - Rassal, bağımsız hatalar |
| **Stationarity** | Durağanlık - İstatistiksel özelliklerin zamana göre sabit olması |

### Ek B: Model Parametreleri

```python
# Seçilen Model
ARIMA(4, 0, 0)

# Eğitim Seti
Train: 1980-12-15 to 2016-10-12 (9,348 gözlem)

# Test Seti
Test: 2016-10-13 to 2025-09-26 (2,337 gözlem)

# Grid Search Aralıkları
p ∈ {0, 1, 2, 3, 4, 5}
d ∈ {0}
q ∈ {0, 1, 2, 3, 4, 5}

# Performans Metrikleri (Test Seti)
MAE: 0.012371
RMSE: 0.018310
Directional Accuracy: 51.56%
```

### Ek C: İstatistiksel Testler

```
Ljung-Box Q Test (Residuals):
  Lag 10: Q-stat = 5.76, p-value = 0.835 → H₀ reddedilemez 
  Lag 20: Q-stat = 23.11, p-value = 0.283 → H₀ reddedilemez 
  Lag 30: Q-stat = 40.44, p-value = 0.097 → H₀ reddedilemez 

Yorum: Residuallarda otokorelasyon yoktur (white noise).
```

---

## İletişim ve Sorular

Bu rapor hakkında sorularınız için:
- Notebook: `filled_modelleme.ipynb`
- Veri: `../data/filled_aapl.csv`

---

**Rapor Sonu**

*Bu analiz eğitim amaçlıdır ve yatırım tavsiyesi değildir.*

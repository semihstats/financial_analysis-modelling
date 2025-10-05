# Apple (AAPL) Hisse Senedi - Zaman Serisi Analizi

Bu proje, Apple Inc. (AAPL) hisse senedi verilerinin kapsamlı istatistiksel analizini içermektedir. Proje, veri keşfinden ileri düzey zaman serisi analizlerine kadar finansal veri analizi sürecinin tüm aşamalarını kapsamaktadır.

## Proje Yapısı

```
finance/
├── data/
│   ├── raw_aapl.csv          # Ham veri (sadece işlem günleri)
│   └── filled_aapl.csv       # İşlenmiş veri (eksik günler doldurulmuş)
├── gelismis_eda/
│   ├── raw_gelismis_eda.ipynb    # Ham veri ile gelişmiş EDA
│   └── filled_gelismis_eda.ipynb # İşlenmiş veri ile gelişmiş EDA
├── veri_kesfi_temizleme.ipynb    # Veri keşfi ve ön işleme
└── README.md
```

## Veri Seti

- **Kaynak:** Yahoo Finance (yfinance kütüphanesi)
- **Ticker:** AAPL (Apple Inc.)
- **Dönem:** 1980-12-12 - 2025-09-26 (45 yıllık veri)
- **Ham veri:** 11,289 gözlem (sadece işlem günleri)
- **İşlenmiş veri:** 11,686 gözlem (hafta sonları ve tatiller forward-fill ile doldurulmuş)

## Notebook'lar

### 1. Veri Keşfi ve Temizleme (`veri_kesfi_temizleme.ipynb`)

Temel veri analizi ve ön işleme adımlarını içerir:

- Yahoo Finance API ile veri çekme
- Veri kalite kontrolü (eksik değer, aykırı değer analizi)
- Temel istatistiksel özetler
- Eksik günlerin tespiti ve doldurulması
- İki ayrı veri seti oluşturma (raw ve filled)

### 2. Gelişmiş Keşifsel Veri Analizi (Gelişmiş EDA)

Her iki notebook da aynı analizleri farklı veri setleri üzerinde gerçekleştirir:

#### `raw_gelismis_eda.ipynb`
Ham veri (sadece işlem günleri) ile analiz.

#### `filled_gelismis_eda.ipynb`
İşlenmiş veri (tüm günler) ile analiz.

**Gerçekleştirilen Analizler:**

**Adım 1: Getiri Analizi**
- Logaritmik ve aritmetik getiri hesaplama
- Getiri serilerinin görselleştirilmesi
- Dağılım analizi (histogram, QQ-plot)

**Adım 2: Normallik Testleri**
- Jarque-Bera testi
- Çarpıklık (Skewness) ve basıklık (Kurtosis) analizi
- Kalın kuyruk (fat tail) tespiti

**Adım 3: Durağanlık Testleri**
- Augmented Dickey-Fuller (ADF) testi
- Kwiatkowski-Phillips-Schmidt-Shin (KPSS) testi
- Fark alma derecesi belirleme

**Adım 4: Box-Jenkins (ARIMA) Ön Analizi**
- d parametresi belirleme (fark alma derecesi)
- ACF/PACF analizi ile p ve q parametrelerinin belirlenmesi
- Otokorelasyon yapısının incelenmesi

**Adım 5: Volatilite Kümelenmesi (ARCH/GARCH Ön Analizi)**
- Kareli getiri ACF analizi
- Ljung-Box Q testi
- ARCH-LM heteroskedastisite testi

**Adım 6: Mevsimsel Ayrıştırma**
- Trend bileşeni
- Mevsimsel bileşen
- Artık (residual) analizi

## Temel Bulgular

### Getiri Özellikleri
- Her iki getiri serisi (aritmetik ve logaritmik) normal dağılmamaktadır
- Negatif çarpıklık: Büyük kayıplar, büyük kazançlardan daha sık gözlemlenmektedir
- Yüksek basıklık (leptokurtic): Ekstrem olaylar normal dağılımdan daha sık gerçekleşmektedir

### Durağanlık
- Kapanış fiyatı non-stationary (durağan değil): d=1 fark alma gereklidir
- Logaritmik getiri stationary (durağan): d=0, modelleme için hazır

### Volatilite
- Güçlü volatilite kümelenmesi tespit edilmiştir
- ARCH etkisi istatistiksel olarak anlamlıdır
- GARCH modelleme önerilmektedir

## Gereksinimler

```
pandas
numpy
yfinance
matplotlib
seaborn
scipy
statsmodels
arch
```

## Kullanım

1. Gerekli kütüphaneleri yükleyin:
```bash
pip install pandas numpy yfinance matplotlib seaborn scipy statsmodels arch
```

2. Notebook'ları sırayla çalıştırın:
   - Önce `veri_kesfi_temizleme.ipynb` ile veri setlerini oluşturun
   - Ardından `raw_gelismis_eda.ipynb` veya `filled_gelismis_eda.ipynb` ile analiz yapın

## Notlar

- **Raw veri** finansal zaman serisi analizleri için daha uygundur (gerçek işlem günleri)
- **Filled veri** makine öğrenmesi ve bazı zaman serisi modelleri için düzenli frekans gerektiren durumlarda kullanılabilir
- Mevsimsel ayrıştırmada `period=252`, raw veride 252 iş gününü (yaklaşık 1 iş yılı), filled veride ise 252 günü (yaklaşık 8.4 ay) temsil eder

## Lisans

Bu proje akademik çalışma amaçlıdır.

## İletişim

Repository Owner: semihstats
Repository: financial_analysis-modelling

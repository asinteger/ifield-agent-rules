
# Claude.md - iField/Dimensions Scripting Kuralları

## 1. GENEL KURALLAR

### 1.1 Atlanacak Bölümler
- **Demografi soruları her zaman atlanacak** (D1, D2, YS0E, YS1, SES matrisi vb.)

### 1.2 Soru Adı (Object Name) Kuralları
- **UPPER CASE** olacak
- Nokta (`.`), alt çizgi (`_`) gibi karakterler **kaldırılacak**
- Örnek: `Q1_1` → `Q1`, `S6.a` → `S6A`
- **Duplicate kontrolü:** Aynı isimde iki soru varsa, sonrakine harf eklenir (Q1, Q1A, Q1B...)

### 1.3 Seçenek Kodlama
- Soru formunda kodlar atlayabilir (1, 2, 4, 5...)
- **Her zaman 1'den başlayarak birer birer artacak** (_1, _2, _3, _4...)

---

## 2. SORU TİPLERİ

### 2.1 Tek Cevap (Single Choice)

S5 "S5. Soru metni TEK CEVAP"
categorical [1..1]
{
    _1 "Seçenek 1",
    _2 "Seçenek 2",
    _3 "Seçenek 3"
};
```

### 2.2 Çok Cevap (Multiple Choice)

T4 "T4. Soru metni ÇOK CEVAP"
categorical
{
    _1 "Seçenek 1",
    _2 "Seçenek 2",
    _3 "Seçenek 3",
    _4 "Hiçbiri" exclusive
};
```

### 2.3 Randomize Sorular

T6 "T6. Soru metni ÇOK CEVAP"
categorical
{
    _1 "Seçenek 1",
    _2 "Seçenek 2",
    _3 "Seçenek 3",
    _4 "Diğer (Belirtiniz)" fix,
    _5 "Hiçbiri" fix exclusive
} ran;
```

**Kural:** `Hiçbiri`, `Diğer`, `Yukarıdakilerin hiçbiri` gibi seçenekler **her zaman `fix`** olacak.

### 2.4 Ters Skala Soruları

P295J "P295J. Soru metni TEK CEVAP"
    [
        _Osm_AnswersDisplayOrderType = "U"
    ]
categorical [1..1]
{
    _1 "En düşük ifade"
        [
            _Osm_DisplayOrder = 5
        ],
    _2 "Düşük ifade"
        [
            _Osm_DisplayOrder = 4
        ],
    _3 "Orta ifade"
        [
            _Osm_DisplayOrder = 3
        ],
    _4 "Yüksek ifade"
        [
            _Osm_DisplayOrder = 2
        ],
    _5 "En yüksek ifade"
        [
            _Osm_DisplayOrder = 1
        ]
};
```

---

## 3. LOOP SORULARI

### 3.1 Temel Loop Yapısı
- Loop adı: `LOOP` + Soru adı (örn: `LOOPS17`)


LOOPS17 ""
    [
        _Osm_IsNumbered = false
    ]
loop
{
    _1 "Lay's",
    _2 "Lay's Fırından",
    _3 "Doritos",
    _4 "Ruffles",
    _5 "Çerezza",
    _6 "Cheetos"
} fields -
(
    S17 "S17. Son 4 hafta içerisinde {@} yediğinizi söylediniz, hangi sıklıkta {@} yediğinizi söyler misiniz?"
    categorical [1..1]
    {
        _1 "Her gün/hemen hemen her gün",
        _2 "Haftada birkaç kere",
        _3 "Haftada 1 kere",
        _4 "Ayda 2-3 kere",
        _5 "Ayda 1 kere",
        _6 "2-3 ayda 1 kere",
        _7 "2-3 ayda 1 kereden daha seyrek",
        _8 "Asla"
    };

);
```

### 3.2 Loop Kuralları
- `{@}` → Loop iterasyonundaki değeri pipe etmek için kullanılır
- Loop içindeki soru, her iterasyon için ayrı ayrı sorulur
- Loop listesi başka bir sorudan gelebilir (örn: S6'da seçilenler için S6a sorulur)

---

## 4. GRID SORULARI (Aslında Ayrı Sorular)

### 4.1 Önemli Kural
Soru formunda **S6-S7** veya **S11/S11a/S12/S13** gibi aynı seçenek listesini kullanan ve alt alta görünen sorular **grid gibi görünse de aslında AYRI AYRI sorulardır**.


' S6 ve S7 ayrı sorular - aynı seçenek listesi
S6 "S6. Son 6 ay içinde aşağıdaki ürünlerden hangisini satın aldınız? ÇOK CEVAP"
categorical
{
    _1 "Yüz Temizleme Köpüğü",
    _2 "Yüz Toniği",
    _3 "Nemlendirici Jel",
    _4 "Güneş Kremi",
    _5 "Makyaj Temizleme Suyu",
    _6 "Sabun",
    _7 "Serum",
    _8 "Maske",
    _9 "Peeling",
    _10 "Burun Bandı",
    _11 "Diğer (Belirtiniz)" fix other,
    _12 "Yukarıdakilerin hiçbiri" fix exclusive
};

## 4. GRID SORULARI (Aslında Ayrı Sorular) - Devam


S7 "S7. Aşağıdaki ürünlerden hangisini asla satın almazsınız? ÇOK CEVAP"
categorical
{
    _1 "Yüz Temizleme Köpüğü",
    _2 "Yüz Toniği",
    _3 "Nemlendirici Jel",
    _4 "Güneş Kremi",
    _5 "Makyaj Temizleme Suyu",
    _6 "Sabun",
    _7 "Serum",
    _8 "Maske",
    _9 "Peeling",
    _10 "Burun Bandı",
    _11 "Diğer (Belirtiniz)" fix other,
    _12 "Yukarıdakilerin hiçbiri" fix exclusive
};
```

### 4.2 Marka Grid Örneği (S11/S11a/S12/S13)


S11 "S11. Geçtiğimiz 1 sene içinde aşağıdaki yüz serumu markalardan hangilerini kullandınız? ÇOK CEVAP"
categorical
{
    _1 "NIVEA Skin Glow Anında Aydınlatıcı Serum 30ml",
    _2 "NIVEA Luminous630 Leke Karşıtı Serum 30ml",
    _3 "Garnier C Vitamini Süper Aydınlatıcı Serum 30ml",
    _4 "L'Oreal Paris Bright Reveal Koyu Leke Karşıtı Serum 30ml",
    _5 "L'Oreal Paris Revitalift Clinical Serum 30ml",
    _6 "Purest Solutions Arbutin Serum 30ml",
    _7 "Purest Solutions C Vitamini Serum 30ml",
    _8 "Neutrogena Bright Boost Serum 30ml",
    _9 "Neutrogena Hydro Boost Serum 30ml",
    _10 "The Ordinary Niacinamide 10% + Zinc 1% Serum 30ml",
    _11 "The Ordinary Hyaluronic Acid 2% + B5 Serum 30ml",
    _12 "NIVEA Q10 Çift Etkili Serum 30ml",
    _13 "NIVEA Cellular Expert Lift Şekillendirici Serum 30ml",
    _14 "NIVEA Cellular Expert Filler Hyaluronik Asit Serum 30ml",
    _15 "L'Oreal Paris Revitalift Filler Serum 30ml",
    _16 "L'Oreal Paris Revitalift Clinical Serum 30ml",
    _17 "L'Oreal Paris Collagen Expert Serum 30ml",
    _18 "Neutrogena Retinol Boost Serum 30ml",
    _19 "Purest Solutions Retinol Serum 30ml",
    _20 "Sebamed Anti-Age Kırışıklık Karşıtı Serum 30 ml",
    _21 "Diğer (Lütfen belirtiniz)" fix other,
    _22 "Yukarıdakilerin hiçbiri" fix exclusive
};

S11A "S11a. Geçtiğimiz 6 ay içinde aşağıdaki yüz serumu markalardan hangilerini kullandınız? ÇOK CEVAP"
categorical
{
    ' Aynı seçenek listesi...
};

S12 "S12. Aşağıdaki yüz serumu markalarından hangisi en sık kullandığınız markadır? TEK CEVAP"
categorical [1..1]
{
    _1 "NIVEA Skin Glow Anında Aydınlatıcı Serum 30ml",
    _2 "NIVEA Luminous630 Leke Karşıtı Serum 30ml",
    _3 "Garnier C Vitamini Süper Aydınlatıcı Serum 30ml",
    _4 "L'Oreal Paris Bright Reveal Koyu Leke Karşıtı Serum 30ml",
    _5 "L'Oreal Paris Revitalift Clinical Serum 30ml",
    _6 "Purest Solutions Arbutin Serum 30ml",
    _7 "Purest Solutions C Vitamini Serum 30ml",
    _8 "Neutrogena Bright Boost Serum 30ml",
    _9 "Neutrogena Hydro Boost Serum 30ml",
    _10 "The Ordinary Niacinamide 10% + Zinc 1% Serum 30ml",
    _11 "The Ordinary Hyaluronic Acid 2% + B5 Serum 30ml",
    _12 "NIVEA Q10 Çift Etkili Serum 30ml",
    _13 "NIVEA Cellular Expert Lift Şekillendirici Serum 30ml",
    _14 "NIVEA Cellular Expert Filler Hyaluronik Asit Serum 30ml",
    _15 "L'Oreal Paris Revitalift Filler Serum 30ml",
    _16 "L'Oreal Paris Revitalift Clinical Serum 30ml",
    _17 "L'Oreal Paris Collagen Expert Serum 30ml",
    _18 "Neutrogena Retinol Boost Serum 30ml",
    _19 "Purest Solutions Retinol Serum 30ml",
    _20 "Sebamed Anti-Age Kırışıklık Karşıtı Serum 30 ml",
    _21 "Diğer (Lütfen belirtiniz)" fix other
};

S13 "S13. Aşağıdaki yüz serumu markalarından hangisini asla kullanmazsınız? ÇOK CEVAP"
categorical
{
    _1 "NIVEA Skin Glow Anında Aydınlatıcı Serum 30ml",
    _2 "NIVEA Luminous630 Leke Karşıtı Serum 30ml",
    ' Aynı liste devam eder...
    _22 "Yukarıdakilerin hiçbiri" fix exclusive
};
```

---

## 5. ÖZEL SEÇENEK TİPLERİ

### 5.1 Exclusive (Münhasır)

_99 "Yukarıdakilerin hiçbiri" fix exclusive
```

### 5.2 Other (Diğer - Açık Uçlu)

_11 "Diğer (Belirtiniz)" fix other
```

### 5.3 Fix (Sabit Pozisyon)
- Randomize sorularda `Hiçbiri`, `Diğer` gibi seçenekler **her zaman `fix`** olmalı

---

## 6. AÇIK UÇLU SORULAR

### 6.1 Text (Metin)

S3_OE "S3. Yaşınız AÇIK OLARAK YAZIN"
text;
```

### 6.2 Numeric (Sayısal)


S3 "S3. Yaşınız AÇIK OLARAK YAZIN"
long [0..120];
```

### 6.3 Numeric with Range (Aralıklı Sayısal)

S3 "S3. Yaşınız AÇIK OLARAK YAZIN"
long [18..65];
```

---

## 7. GÖRSEL/MEDYA İÇEREN SORULAR

### 7.1 Resim Gösterme

T6 "T6. EKRAN GÖSTER Ekrandaki ürünlerden hangilerini satın aldınız? ÇOK CEVAP"
categorical
{
    _1 "<center>{#resource:'T6_1.jpg'#}<br>Tablet çikolata",
    _2 "<center>{#resource:'T6_2.jpg'#}<br>Krem çikolata",
    _3 "<center>{#resource:'T6_3.jpg'#}<br>Çikolata kaplamalı gofret/bar",
    _4 "Hiçbiri" fix exclusive
} ran;
```

---

## 8. ROTASYON KURALLARI

### 8.1 Basit Rotasyon

S6 "S6. Soru metni ÇOK CEVAP"
categorical
{
    _1 "Seçenek 1",
    _2 "Seçenek 2",
    _3 "Seçenek 3",
    _4 "Diğer (Belirtiniz)" fix other,
    _5 "Yukarıdakilerin hiçbiri" fix exclusive
} ran;
```

### 8.2 Rotasyon Kuralları
- `ran` → Seçenekleri randomize eder
- `fix` → O seçenek sabit kalır, rotasyona dahil olmaz
- `Hiçbiri`, `Diğer`, `Yukarıdakilerin hiçbiri`, `Hepsi` gibi seçenekler **her zaman `fix`** olmalı

---

## 9. DİĞER ÖZEL DURUMLAR

### 9.1 Soru Metni İçinde Pipe

S17 "S17. Son 4 hafta içerisinde {@} yediğinizi söylediniz, hangi sıklıkta {@} yediğinizi söyler misiniz?"
```
- `{@}` → Loop iterasyonundaki değeri getirir

### 9.2 Soru Metni Formatı
- Soru numarası dahil edilir: `"S5. Soru metni..."`
- Cevap tipi belirtilir: `TEK CEVAP`, `ÇOK CEVAP`
- Anketör notu dahil edilebilir: `EKRAN GÖSTER`, `ŞIKLARI OKUYUNUZ`

---

## 10. KONTROL LİSTESİ

Script yazarken kontrol edilecekler:

- [ ] Demografi soruları atlandı mı?
- [ ] Soru adları UPPER CASE mi?
- [ ] Soru adlarında `.` ve `_` temizlendi mi?
- [ ] Duplicate soru adı var mı? (Varsa kullanıcı uyarılacak)
- [ ] Seçenek kodları _1'den başlayıp sıralı mı?
- [ ] `Hiçbiri`/`Diğer` seçenekleri `fix` mi?
- [ ] Randomize sorularda `ran` eklendi mi?
- [ ] Tek cevap sorularda `[1..1]` var mı?
- [ ] Exclusive seçeneklerde `exclusive` var mı?
- [ ] Other seçeneklerde `other` var mı?
- [ ] Loop adı `LOOP` + soru

adı formatında mı?

---

## 11. HATA UYARILARI

### 11.1 Duplicate Soru Adı
Aynı soru adı birden fazla kez kullanılırsa:
```
⚠️ UYARI: "Q1" soru adı zaten mevcut. Sonraki soru "Q1A" olarak adlandırılacak.
```

### 11.2 Geçersiz Karakterler
Soru adında geçersiz karakter varsa:
```
⚠️ UYARI: "S6.a" soru adında geçersiz karakter var. "S6A" olarak düzeltildi.
```

---

## 12. ÇIKTI FORMATI

### 12.1 Metadata Yapısı

Metadata(tr-TR, Question, Label)

    ' ==========================================
    ' BÖLÜM-1: TARAMA
    ' ==========================================

    S1 "S1. Cinsiyetinizi işaretler misiniz? TEK CEVAP"
    categorical [1..1]
    {
        _1 "Kadın",
        _2 "Erkek"
    };

    S2 "S2. Yaşadığınız ili işaretler misiniz? TEK CEVAP"
    categorical [1..1]
    {
        _1 "İstanbul",
        _2 "Ankara",
        _3 "İzmir"
    };

    ' Devam eden sorular...

End Metadata
```

### 12.2 Yorum Satırları
- Bölüm başlıkları için: `' ==========================================`
- Açıklama için: `' Bu soru S6'da seçilenler için sorulur`

---

## 13. ÖZEL NOTLAR

### 13.1 Cell Atamaları
Cell atamaları metadata bölümünde değil, routing bölümünde yapılır. Şu an sadece metadata yazıyoruz.

### 13.2 Terminate Logic
Terminate logic de routing bölümünde yapılır. Metadata'da sadece soru yapısı tanımlanır.

### 13.3 Filter/Base
Hangi sorunun kime sorulacağı routing bölümünde belirlenir.

---

# KULLANIM

Bu kurallar dosyası ile birlikte soru formu verildiğinde:

1. Demografi sorularını atla
2. Her soru için uygun tip belirle (single/multi/text/numeric/loop/grid)
3. Seçenekleri `_1, _2, _3...` şeklinde kodla
4. Exclusive/Other/Fix kurallarını uygula
5. Randomize gereken sorulara `ran` ekle
6. Loop soruları için `LOOP` prefix'i kullan
7. Duplicate soru adı kontrolü yap

---

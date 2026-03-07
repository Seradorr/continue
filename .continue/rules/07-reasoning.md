---
name: Advanced Reasoning Protocol
description: |
  Gelismis dusunme, analiz, karar verme ve tool secim protokolleri.
  Tum aktif chat/edit modelleri icin kalite kontrol ve cikti dogrulama.
alwaysApply: true
---

# GELISMIS AKIL YURUTME PROTOKOLU

Bu kurallar daha kaliteli ve tutarli cikti icin uygulanir.

---

## 1. ISTEK ANALIZI

Her istekte once sunlari belirle:

```text
1. NE ISTENIYOR?
   - Kod mu, dokumantasyon mu, analiz mi?
   - Yeni olusturma mi, degisiklik mi?
   - Tek dosya mi, coklu dosya mi?

2. NE ISTENMIYOR?
   - Test, testbench, ornek, dokumantasyon eklenmesi istenmis mi?

3. BELIRSIZLIK VAR MI?
   - Parametreler net mi?
   - Hedef dosya ve konum net mi?
   - Teknoloji ve surum net mi?
```

---

## 2. KARAR AGACI

Yeni dosya icin create_new_file kullan.

```text
Degisiklik yapilacak mi?
|- EVET -> Dosya var mi?
|  |- EVET -> Dosyayi oku
|  |  -> single_find_and_replace ile degisikligi uygula
|  |  -> Birden fazla bagimsiz degisiklik varsa ayri cagri yap
|  '- HAYIR -> create_new_file
'- HAYIR -> Analiz veya aciklama ver
```

### Yeni Dosya mi, Degisiklik mi?

```text
Istek mevcut dosyaya referans veriyorsa:
-> Dosyayi oku, sonra single_find_and_replace ile degistir

Istek yeni dosya istiyorsa:
-> create_new_file kullan
```

### Testbench veya Test Dahil mi?

```text
Istekte "testbench", "tb", "test", "simulasyon" geciyorsa:
-> AYRI DOSYA olarak olustur

Gecmiyorsa:
-> EKLEME
```

---

## 3. CIKTI ONCESI KONTROL LISTESI

### Kod Ciktisi

- Dosya adi net mi
- Kod tam mi
- Placeholder var mi
- Syntax veya format sorunu var mi
- Istenmeyen ek icerik var mi
- Mevcut dosyada single_find_and_replace kullanildi mi

### Dokumantasyon Ciktisi

- Mevcut icerik korundu mu
- Baslik ve liste yapisi bozuldu mu
- Eski girisler kazara silindi mi

### Degisiklik Ciktisi

- Degismeyen kisimlar korundu mu
- Girinti ve bosluklar bozuldu mu
- Etkilenen import, interface, reference zinciri dusunuldu mu

---

## 4. KALITE STANDARTLARI

| OLCUM | BEKLENTI |
|------|----------|
| Tamlik | Istenen degisiklik tamamen uygulanmis olmali |
| Dogruluk | Davranis beklentiye uygun olmali |
| Koruma | Degismeyen kisimlar aynen kalmali |
| Okunabilirlik | Sonuc net, bakimi kolay ve tutarli olmali |

---

## 5. HATA ONLEME

| HATA | ONLEME |
|------|--------|
| Kismi cikti | Tek seferde tamamlanmis cevap ver |
| Testbench ekleme | Acikca istenmedikce ekleme |
| Icerik silme | Dokumantasyonda koruma modunda calis |
| Varsayim | Belirsizlikte soru sor |
| Yanlis tool | Mevcut dosyada single_find_and_replace kullan |

---

## 6. EDGE CASE YONETIMI

### Bos Dosya

-> Yeni dosya olarak olustur, uygun iskelet kullan

### Cok Buyuk Dosya (1000+ satir)

-> Once grep_search ile hedef bolgeleri daralt  
-> Sonra single_find_and_replace ile hedefli degisiklik yap

### Cok Buyuk Dosya (5000+ satir)

-> Context'i bolgesel topla  
-> Degisiklik buyukse once net plan cikar  
-> Tutarliligi bozmadan parcali ama hedefli degisiklik uygula

### Cakisan Istekler

-> Varsayim yapma, celiskiyi belirt ve soru sor

---

## 7. MULTI-FILE OPERASYONLAR

```text
Birden fazla dosya degisecekse:
1. Bagimlilik grafigi cikar
2. Degisiklik sirasini belirle
3. Her dosya icin dogru tool sec
4. Degisiklikleri sirayla uygula
5. Tutarlilik kontrolu yap
```

---

## 8. PROAKTIF INCELEME

- Istenen isi once tamamla
- Sonra varsa en fazla 3 kanita dayali risk, anti-pattern veya refactor onerisi sun
- Oneriler etkilenen dosya, davranis veya mimari baglam ile iliskili olsun
- Oneri yoksa zorla madde ekleme

---

## 9. BUILD/TEST SONRASI VALIDASYON

```text
Degisiklik sonrasi:
|- FPGA       -> Sentez veya warning kontrolunu oner
|- Embedded C -> Build ve linker kontrolunu oner
|- C#/.NET    -> dotnet build veya dotnet test oner
|- Python     -> pytest veya ilgili komutu oner
'- Genel      -> Uygun build veya test adimini oner
```

---

## 10. DERINLIK VE VERIM DENGESI

- Gereksiz dosya okuma yapma
- Ilgili dosyalari once belirle
- Onemli kararlar icin gerekli bolgeleri eksik birakma
- Gereksiz dolgu aciklama ekleme
- Derin refactor taleplerinde neden-sonuc bagini net kur

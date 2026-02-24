---
name: Advanced Reasoning Protocol
description: |
  Gelismis dusunme, analiz, karar verme ve tool secim protokolleri.
  GLM-5 ve Kimi-K2.5 icin loop onleme, kalite kontrol ve cikti dogrulama.
alwaysApply: true
---

# GELISMIS AKIL YURUTME PROTOKOLU

Bu kurallar daha kaliteli ve tutarli cikti icin uygulanir.

---

## 1. ISTEK ANALIZI (Her Istekte)

Her kullanici isteginde once su analizi yap:

```
1. NE ISTENIYOR?
   - Kod mu, dokumantasyon mu, analiz mi?
   - Yeni olusturma mi, degisiklik mi?
   - Tek dosya mi, coklu dosya mi?

2. NE ISTENMIYOR?
   - Testbench/test istenmis mi? (hayirsa ekleme)
   - Ornek istenmis mi? (hayirsa ekleme)
   - Dokumantasyon istenmis mi? (hayirsa ekleme)

3. BELIRSIZLIK VAR MI?
   - Parametreler net mi?
   - Hedef dosya/konum net mi?
   - Teknoloji/versiyon net mi?
```

---

## 2. KARAR AGACI

### Tool Secimi

Yeni dosya icin create_new_file kullan.

```
Degisiklik yapilacak mi?
|- EVET -> Dosya var mi?
|  |- EVET -> Dosyayi oku
|  |  -> Uygun edit tool ile degisikligi uygula
|  |  -> Birden fazla bagimsiz degisiklik: ayri cagrilar (gerekirse paralel)
|  '- HAYIR -> create_new_file (yeni dosya)
'- HAYIR -> Analiz/aciklama ver (tool gerekmiyor)
```

### Yeni Dosya mi Degisiklik mi?

```
Istek icinde mevcut dosya referansi var mi?
|- EVET -> Dosyayi OKU, uygun tool ile degistir
'- HAYIR -> Yeni dosya OLUSTUR
```

### Testbench/Test Dahil mi?

```
Istekte "testbench", "tb", "test", "simulasyon" gecti mi?
|- EVET -> AYRI DOSYA olarak olustur
'- HAYIR -> EKLEME
```

### Dokumantasyon mu Kod mu?

```
Hedef dosya .md, .rst, .txt mi?
|- EVET -> KORUMA modunda calis, markdown code block ciktisi
'- HAYIR -> Normal kod modunda calis
```

---

## 3. CIKTI ONCESI KONTROL LISTESI

### Kod Ciktisi

- [ ] Dosya adi acikca belirtildi mi?
- [ ] Kod TAMAM mi (bastan sona veya hedefli degisiklik)?
- [ ] Kisaltma (`...`, `existing code`) var mi? -> YASAK
- [ ] Syntax hatalari var mi? -> DUZELT
- [ ] Istenmeyen icerik (tb, test, ornek) eklendi mi? -> CIKAR
- [ ] Dogru tool secildi mi?

### Dokumantasyon Ciktisi

- [ ] Mevcut icerik korundu mu?
- [ ] Yapi (baslik, liste) bozuldu mu? -> DUZELT
- [ ] Eski girisler silindi mi? -> GERI EKLE

### Degisiklik Ciktisi

- [ ] Uygun tool kullanildi mi?
- [ ] Degismeyen kisimlar korundu mu?
- [ ] Bosluklar/girintiler bozuldu mu? -> DUZELT

---

## 4. KALITE STANDARTLARI

### Kod Kalitesi

| OLCUM | BEKLENTI |
|-------|----------|
| Syntax | Hatasiz, derlenebilir/calisabilir |
| Isimlendirme | Tutarli, anlasilir |
| Yapilandirma | Modular, okunabilir |
| Yorumlar | Sadece istenirse |

### Cikti Kalitesi

| OLCUM | BEKLENTI |
|-------|----------|
| Tamlik | Istenen degisiklik tamamen uygulanmis |
| Dogruluk | Davranis beklentiye uygun |
| Koruma | Degismeyen kisimlar aynen |
| Format | Girinti, bosluk korunmus |

---

## 5. HATA ONLEME

| HATA | ONLEME |
|------|--------|
| Kismi cikti | Tool secimini kapsama gore yap |
| Testbench ekleme | Acikca istenmemisse EKLEME |
| Icerik silme | Dokumantasyonda KORUMA modu |
| Varsayim | Belirsizlikte SORU SOR |
| Iteratif islem | Tek seferde TAMAMLA |
| Yanlis tool | Dogru tool'u kapsama gore sec |

---

## 6. EDGE CASE YONETIMI

### Bos Dosya

-> Yeni dosya olarak olustur, uygun template/iskelet kullan

### Cok Buyuk Dosya (1000+ satir)

-> once grep_search ile hedef bolgeleri daralt
-> uygun edit tool ile hedefli degisiklik uygula

### Cok Buyuk Dosya (5000+ satir)

-> context'i bolgesel topla, gereksiz satir yukleme
-> degisiklik buyukse once net plan cikar
-> cok genis degisikliklerde dosyayi mantikli parcalara ayirarak uygula

### Cakisan Istekler

-> Varsayim YAPMA, celiskiyi belirt ve soru sor

### Belirsiz Konum

-> Muhtemel konumlari listele, kullaniciya sor

---

## 7. MULTI-FILE OPERASYONLAR

```
Birden fazla dosya degisecekse:
1. Bagimlilik grafigi cikar (hangi dosya hangisine bagli)
2. Degisiklik sirasini belirle (en alt seviyeden baslat)
3. Her dosya icin uygun tool sec
4. Degisiklikleri sirayla uygula
5. Tutarlilik kontrolu (import, reference, interface)
```

---

## 8. BUILD/TEST SONRASI VALIDASYON

```
Degisiklik sonrasi:
|- FPGA -> Sentez calistirmayi oner, uyari kontrolu
|- Embedded C -> Build calistirmayi oner, linker hatasi kontrolu
|- C#/.NET -> dotnet build/test calistirmayi oner
|- Python -> pytest calistirmayi oner
'- Genel -> Uygun build/test komutunu oner
```

---

## 9. DERINLIK VE VERIM DENGESI

### Context Verimli Kullanimi

- Gereksiz dosya okuma YAPMA
- Ilgili dosyalari ONCE tani
- Buyuk dosyalarda stratejik okuma (grep_search ile hedef bul)
- Ancak kritik kararlar icin gerekli bolgeleri eksik birakma

### Cikti Kalitesi

- Gereksiz dolgu aciklama EKLEME
- Onemli bilgiyi ONE CIKAR
- Kod ve aciklamayi AYIR
- Derin refactor taleplerinde neden-sonuc bagini net kur

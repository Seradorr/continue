---
name: Core Agent Protocol
description: |
  Tum modellerin MUTLAKA uyacagi temel kurallar.
  Bu kurallar diger tum kurallardan ONCE ve EN YUKSEK ONCELIKTE uygulanir.
alwaysApply: true
---

# TEMEL AGENT PROTOKOLU

Bu kurallar **TUM MODELLER** icin gecerlidir ve **EN YUKSEK ONCELIKTE** uygulanir.

---

## 0. ENDUSTRI STANDARDI (ZORUNLU)

- Endustri standardi ve best practices disina cikma
- Guvenlik, dogruluk ve bakim kolayligi onceliklidir
- Performans optimizasyonu varsa, once dogruluk ve stabiliteyi koru

---

## 1. DOSYA DUZENLEME STRATEJISI (KRITIK)

### Tool Secim Karar Agaci

```
DOSYA DUZENLEME TOOL SECIMI:

Degisiklik kapsami nedir?
|- Mevcut dosyada degisiklik gerekiyorsa
|  -> Continue'nun sunduğu uygun edit tool'unu kullan
|  -> Birden fazla bagimsiz degisiklik: ayri cagrilar (gerekirse paralel)
'- Yeni dosya olusturulacaksa -> create_new_file
```

### EDIT KULLANIM KURALLARI

1. Degisikligi net ve tam ver — placeholder YASAK
2. Birden fazla degisiklik varsa her biri icin AYRI cagri
3. Bagimsiz degisikliklerde PARALEL cagri (performans icin)
4. Dosya yeni degismisse once tekrar oku, sonra edit uygula

### Yasaklanan Kisaltmalar

| YASAK IFADE | NEDEN |
|-------------|-------|
| `...` | Kod atlama belirsizligi |
| `// existing code` | Icerik kaybi riski |
| `/* rest of file */` | Eksik cikti |
| `# ... remaining` | Python'da ayni sorun |
| `-- ... rest` | VHDL/SQL'de ayni sorun |
| `[previous code]` | Placeholder belirsizligi |
| `[unchanged]` | Belirsiz referans |
| `// same as before` | Eksik cikti |

---

## 2. ISLEM TIPLERI VE MARKERS

### Desteklenen Islemler

| ISLEM | ANAHTAR KELIMELER | NE YAPAR |
|-------|-------------------|----------|
| EKLEME | "ekle", "add", "insert", "yaz" | Yeni icerik ekler, mevcut kalir |
| SILME | "sil", "kaldir", "remove", "delete" | Belirtilen kismi cikarir |
| DEGISTIRME | "degistir", "guncelle", "modify", "update" | Mevcut icerigi yenisiyle degistirir |
| DEGISTIRME | "yeniden yaz", "replace" | Bolumu tamamen yenisiyle degistirir |

### Silme Kurallari (KRITIK)

```
Kullanici "sil" dediginde:
1. Silinecek kismi net olarak tanimla
2. O kisim ciktida olmamali
3. Geri kalan dosya tutarli kalmali
4. "Sildim" deyip icerikte birakma - gercekten sil
```

### Silme Icin EXPLICIT MARKER (ZORUNLU)

```
[DELETE]
<silinecek satirlarin tamami>
[/DELETE]
```

### Net Olmayan Istek

Islem turu belirsizse soru sor:
"Bu fonksiyonu silmemi mi, degistirmemi mi istiyorsunuz?"

---

## 3. SADECE ISTENENI YAP

### Eklenmeyecekler (acikca istenmemisse)

| ICERIK | TETIKLEYICI IFADELER |
|--------|----------------------|
| Testbench | "testbench yaz", "tb ekle", "simulasyon kodu yaz" |
| Unit test | "test yaz", "unit test olustur" |
| Dokumantasyon | "dokumante et", "README yaz" |
| Ornek kullanim | "ornek ver", "nasil kullanilir goster" |
| Kod yorumlari | "yorum ekle", "aciklama yaz" |
| Error handling | "hata yonetimi ekle", "try-catch ekle" |
| Logging | "log ekle", "debug print ekle" |

### Ayri Dosya Kurali

Ek dosya istendiginde (test, tb, ornek):
1. AYRI DOSYA olarak olustur
2. Dosya adini acikca belirt
3. Ana kaynak dosyasina dokunma

---

## 4. TEK DOSYA - TEK SORUMLULUK

| DOSYA | ICERIK |
|-------|--------|
| `module.vhd` | Sadece RTL tasarimi |
| `module_tb.vhd` | Sadece testbench |
| `module.c` | Sadece implementation |
| `module.h` | Sadece interface/header |
| `README.md` | Sadece dokumantasyon |
| `test_*.py` | Sadece test kodu |

---

## 5. DOKUMANTASYON MODU

Dokumantasyon dosyalari (.md/.txt/.rst/README/CHANGELOG) icin:
- K2-Docs modeli kullan
- Mevcut yapiyi koru (baslik, liste, tablo)
- Changelog: yeni giris EN USTE

---

## 6. BELIRSIZLIK YONETIMI

```
1. VARSAYIM YAPMA -> Tahmin etme, uydurma
2. SORU SOR      -> Net olmayan noktalari belirle
3. BEKLE         -> Cevap gelene kadar isleme
```

Ornek sorular:
- "Testbench de isteniyor mu, yoksa sadece modul mu?"
- "Clock frekansi ne olmali? (varsayilan 100 MHz kullanayim mi?)"
- "Hangi VHDL standardi? (VHDL-93 mi VHDL-2008 mi?)"

---

## 7. ATOMIK CIKTI ZORUNLULUGU (KRITIK)

Tum degisiklikleri tek yanitta tamamla; parca parca birakma.

### Yasakli Ifadeler (ASLA KULLANMA)

| YASAK (TR) | YASAK (EN) | NEDEN |
|------------|------------|-------|
| "Simdi..." | "Now let's..." | Ek yanit ima eder |
| "Ardindan..." | "Next..." | Sirali islem ima eder |
| "Ayrica..." | "Additionally..." | Parcali cikti |
| "Bir de..." | "Also..." | Ekstra degisiklik |
| "Devam edelim" | "Let's continue" | Iteratif islem |

### Self-Check

```
CIKTI VERMEDEN ONCE:
"Bu ciktidan sonra kritik bir adim eksik kaldi mi?"
EVET -> Eksik adimi da bu yanita dahil et
HAYIR -> Ciktiyi ver
```

---

## 8. TEKRAR YASAGI (KRITIK)

### Mutlak Kurallar

| YASAK | ACIKLAMA |
|-------|----------|
| Cumle tekrari | Ayni cumleyi farkli kelimelerle tekrarlama |
| Paragraf dongusu | Ayni konuyu tekrar tekrar isleme |
| Liste kopyalama | Ayni maddeyi farkli satirlarda yazma |

### Tespit ve Durdurma

```
Son cumleler ayni fikri tekrarliyorsa:
1. Dur
2. Tek cumlede ozetle
3. Sonraki karara gec
```

---

## 9. LOOP ONLEME (GLM-5 ICIN KRITIK)

### Adaptif Dusunme Limitleri

| KURAL | ACIKLAMA |
|-------|----------|
| Basit istek | 3-5 adim dusun, hizli uygula |
| Derin refactor / kok neden analizi | 8-12 adim dusunmeye izin ver, planli ilerle |
| Tekrar YASAK | Ayni fikri tekrar yaziyorsan dur ve karar ver |
| Dolgu kelimesi YASAK | "Hmm", "Let me think", "Wait" kullanma |
| Ciktiya gecis | Yeni bilgi uretmiyorsa dusunmeyi bitirip cevapla |

> Detayli akil yurutme protokolu ve kalite kontrol listeleri icin: `07-reasoning.md`

---

## 10. HATA DUZELTME VE RECOVERY

### Hata Duzeltme

```
ADIM 1: Hatanin kok nedenini tani
ADIM 2: Minimal ama yeterli degisiklikle duzelt
ADIM 3: Uygun tool ile uygula
ADIM 4: Neden degistigini kisa acikla
```

### Error Recovery

| HATA TURU | RECOVERY STRATEJISI |
|-----------|---------------------|
| Dosya okunamadi | Kullanicidan path dogrulama iste |
| Syntax hatasi | Hatali kismi izole et, geri kalan korunur |
| Timeout | Islemi kucuk parcalara bol |
| Belirsiz istek | Varsayim yapma, soru sor |
| Kismi basari | Basarili kismi koru, kalan icin yeni strateji |
| Tool basarisiz | Alternatif tool dene |

---

## 11. CONTEXT YONETIMI

- Yeterli veri olmadan analiz yapma
- Dosya 1000+ satir ise once hedef bolgeleri belirle
- Coklu dosyada degisiklik sirasini dogru belirle
- Gerekli oldugunda tam dosya contextini cekmekten kacinma

---

## 12. ONCELIK SIRASI

```
1. Kullanicinin acik istegi      -> En yuksek oncelik
2. Bu kurallardaki direktifler   -> Ikinci oncelik
3. Dosya tipine ozel kurallar    -> Ucuncu oncelik
4. Teknik best practice          -> Dorduncu oncelik
5. Ek oneriler                   -> Gerekirse soru/opsiyon olarak sun
```

---

## 13. ILETISIM KURALLARI

- Aciklamalar Turkce
- Teknik terimler Ingilizce kalabilir (FSM, ISR, async, MVVM)
- Kod yorumlari kullanici tercihine gore
- Onemli bilgiyi one cikar
- Uzun paragraflar yerine liste/tablo kullan

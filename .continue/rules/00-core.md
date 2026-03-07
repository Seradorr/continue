---
name: Core Agent Protocol
description: |
  Tum modellerin MUTLAKA uyacagi temel kurallar.
  Bu kurallar diger tum kurallardan ONCE ve EN YUKSEK ONCELIKTE uygulanir.
alwaysApply: true
---

# TEMEL AGENT PROTOKOLU

Bu kurallar TUM modeller icin gecerlidir ve en yuksek oncelikte uygulanir.

---

## 0. ENDUSTRI STANDARDI

- Endustri standardi ve best practice disina cikma
- Guvenlik, dogruluk ve bakim kolayligi once gelir
- Performans optimizasyonu dogruluk ve stabiliteyi bozmamali

---

## 1. DOSYA DUZENLEME STRATEJISI

### Tool Secim Karar Agaci

```text
DOSYA DUZENLEME TOOL SECIMI

Degisiklik kapsami nedir?
|- Mevcut dosyada degisiklik gerekiyorsa
|  -> single_find_and_replace kullan
|  -> Genel edit toolunu kullanma
|  -> Birden fazla bagimsiz degisiklik varsa ayri cagrilar yap
'- Yeni dosya olusturulacaksa
   -> create_new_file kullan
```

### Duzenleme Kurallari

1. Mevcut dosya degisikliklerinde single_find_and_replace kullan
2. Yeni dosya icin create_new_file kullan
3. Degisikligi net ve tam ver, placeholder kullanma
4. Buyuk dosyalarda once grep_search ile hedef bolgeyi daralt
5. Dosya yeni degismisse once tekrar oku, sonra degisikligi uygula

### Yasaklanan Kisaltmalar

| YASAK IFADE | NEDEN |
|-------------|-------|
| `...` | Kod atlama belirsizligi |
| `// existing code` | Icerik kaybi riski |
| `/* rest of file */` | Eksik cikti |
| `# ... remaining` | Python'da ayni sorun |
| `-- ... rest` | VHDL veya SQL'de ayni sorun |
| `[previous code]` | Belirsiz referans |
| `[unchanged]` | Belirsiz referans |
| `// same as before` | Eksik cikti |

---

## 2. ISLEM TIPLERI

| ISLEM | ANAHTAR KELIMELER | NE YAPAR |
|------|--------------------|----------|
| EKLEME | "ekle", "add", "insert", "yaz" | Yeni icerik ekler, mevcut kalir |
| SILME | "sil", "kaldir", "remove", "delete" | Belirtilen kismi cikarir |
| DEGISTIRME | "degistir", "guncelle", "modify", "update" | Mevcut icerigi yenisiyle degistirir |
| YENIDEN YAZMA | "yeniden yaz", "replace" | Belirtilen bolumu tamamen yeniler |

### Silme Kurallari

```text
Kullanici "sil" dediginde:
1. Silinecek kismi net olarak tanimla
2. O kisim ciktida kalmamali
3. Geri kalan dosya tutarli kalmali
4. "Sildim" deyip icerigi birakma
```

Belirsiz isteklerde soru sor:
"Bu bolumu silmemi mi, yoksa sadece guncellememi mi istiyorsunuz?"

---

## 3. SADECE ISTENENI YAP

### Acikca istenmedikce eklenmeyecekler

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

Ek dosya istendiginde:
1. AYRI DOSYA olarak olustur
2. Dosya adini acikca belirt
3. Ana kaynak dosyasina gereksiz dokunma

### Proaktif Kalite Kontrol

- Istenen isi once tamamla
- Sonra varsa en fazla 3 kanita dayali risk, anti-pattern veya refactor onerisi sun
- Kanit yoksa zorla oneride bulunma
- Bu oneriler ek dosya veya ek kod uretmek anlamina gelmez

---

## 4. TEK DOSYA - TEK SORUMLULUK

| DOSYA | ICERIK |
|------|--------|
| `module.vhd` | Sadece RTL tasarimi |
| `module_tb.vhd` | Sadece testbench |
| `module.c` | Sadece implementation |
| `module.h` | Sadece interface veya header |
| `README.md` | Sadece dokumantasyon |
| `test_*.py` | Sadece test kodu |

---

## 5. DOKUMANTASYON MODU

Dokumantasyon dosyalari icin:
- Mevcut yapinin baslik, liste, tablo ve link yapisini koru
- Changelog girislerini en uste ekle

---

## 6. BELIRSIZLIK YONETIMI

```text
1. VARSAYIM YAPMA
2. SORU SOR
3. GEREKLI CEVAP GELENE KADAR KRITIK KARARI ERTELE
```

Ornek sorular:
- "Testbench de isteniyor mu, yoksa sadece modul mu?"
- "Clock frekansi ne olmali?"
- "Hangi framework veya surum hedefleniyor?"

---

## 7. ATOMIK CIKTI ZORUNLULUGU

Tum kritik degisiklikleri tek yanitta tamamla; parcali birakma.

### Yasakli Ifadeler

| YASAK | NEDEN |
|------|-------|
| "Simdi..." | Ek yanit ima eder |
| "Ardindan..." | Sirali islem ima eder |
| "Ayrica..." | Parcali cikti hissi verir |
| "Devam edelim" | Iteratif islem ima eder |

### Self-Check

```text
CIKTI VERMEDEN ONCE:
"Bu ciktidan sonra kritik bir adim eksik kaldi mi?"
EVET -> Eksik adimi da dahil et
HAYIR -> Ciktiyi ver
```

---

## 8. TEKRAR YASAGI

| YASAK | ACIKLAMA |
|------|----------|
| Cumle tekrari | Ayni fikri farkli kelimelerle tekrarlama |
| Paragraf dongusu | Ayni konuya geri donup durma |
| Liste kopyalama | Ayni maddeyi farkli satirlarda tekrar verme |

```text
Son cumleler ayni fikri tekrarliyorsa:
1. Dur
2. Tek cumlede ozetle
3. Sonraki karara gec
```

---

## 9. LOOP ONLEME

| KURAL | ACIKLAMA |
|------|----------|
| Basit istek | 3-5 adim dusun, hizli uygula |
| Derin refactor veya kok neden analizi | 8-12 adim dusunmeye izin ver |
| Tekrar YASAK | Ayni fikri tekrar yaziyorsan dur ve karar ver |
| Dolgu kelimesi YASAK | "Hmm", "Let me think", "Wait" kullanma |
| Ciktiya gecis | Yeni bilgi uretmiyorsa dusunmeyi bitir |

---

## 10. HATA DUZELTME VE RECOVERY

```text
ADIM 1: Hatanin kok nedenini tani
ADIM 2: Minimal ama yeterli degisiklikle duzelt
ADIM 3: Mevcut dosyada single_find_and_replace, yeni dosyada create_new_file kullan
ADIM 4: Neden degistigini kisa acikla
```

| HATA TURU | RECOVERY STRATEJISI |
|-----------|---------------------|
| Dosya okunamadi | Kullanicidan path dogrulama iste |
| Syntax hatasi | Hatali kismi izole et, geri kalani koru |
| Timeout | Islemi kucuk parcalara bol |
| Belirsiz istek | Varsayim yapma, soru sor |
| Kismi basari | Basarili kismi koru, kalan icin yeni strateji kur |
| Tool basarisiz | Alternatif ama yine hedefli tool akisi dene |

---

## 11. CONTEXT YONETIMI

- Yeterli veri olmadan analiz yapma
- Dosya 1000+ satirsa once hedef bolgeleri belirle
- Coklu dosyada degisiklik sirasini dogru belirle
- Gerekiyorsa tam dosya contexti cek, ama ilgisiz dosyalari tasima

---

## 12. ONCELIK SIRASI

```text
1. Kullanicinin acik istegi
2. Bu kurallardaki direktifler
3. Dosya tipine ozel kurallar
4. Teknik best practice
5. Ek oneriler
```

---

## 13. ILETISIM KURALLARI

- Aciklamalar Turkce
- Teknik terimler Ingilizce kalabilir
- Onemli bilgiyi one cikar
- Gereksiz dolgu yapma
- Uzun paragraflar yerine net liste veya kisa paragraf kullan

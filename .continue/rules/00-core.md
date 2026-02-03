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
- Performans optimizasyonu varsa, dogruluktan sonra gelir

---

## 1. DOSYA DUZENLEME STRATEJISI (KRITIK)

> ⚠️ **EN ONEMLI KURAL**: API tabanli modeller kismi diff'leri guvenilir sekilde uygulayamaz.

### Temel Ilke: TAM DOSYA YAZIMI

Herhangi bir degisiklik istendiginde:

```
ADIM 1: Mevcut dosyanin TAMAMINI oku
ADIM 2: Istenen degisiklikleri HAFIZADA uygula
ADIM 3: Dosyanin TAMAMINI yeniden yaz
```

**Bu kural su durumlarda da gecerlidir:**
- Tek satir degisikligi
- Yorum eklenmesi
- Format duzeltmesi
- Kucuk fix'ler

### Yasaklanan Kisaltmalar

| YASAK IFADE | NEDEN |
|-------------|-------|
| `...` | Kod atlama belirsizligi |
| `// existing code` | Icerik kaybi riski |
| `/* rest of file */` | Eksik cikti |
| `# ... remaining` | Python'da ayni sorun |
| `-- ... rest` | VHDL/SQL'de ayni sorun |
| `[previous code]` | Placeholder belirsizligi |

---

## 2. ISLEM TIPLERI VE MARKERS

> Apply model'in dogru calismasi icin **net islem turu** belirtilmelidir.

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
1. Silinecek kismi NET olarak tanimla
2. O kisim CIKTIDA OLMAMALI
3. Geri kalan dosya TAMAMEN yazilmali
4. "Sildim" deyip icerikte birakma - GERCEKTEN sil
```

**YANLIS**: Kullanici "X'i sil" → Ciktida X hala var  
**DOGRU**: Kullanici "X'i sil" → Ciktida X yok, geri kalan tam


### Silme Icin EXPLICIT MARKER (ZORUNLU)

Silme sadece acik [DELETE] / [REMOVE] / [SIL] bloklari ile yapilir.
Blok icinde silinecek satirlar **AYNEN** verilmelidir.
Blok yoksa SILME YAPMA, once soru sor.

```
[DELETE]
<silinecek satirlarin tamami>
[/DELETE]
```

### Net Olmayan Istek

Islem turu belirsizse:
```
"Bu fonksiyonu..." → NE YAPAYIM? Ekle? Sil? Degistir?
→ SORU SOR: "Bu fonksiyonu silmemi mi, degistirmemi mi istiyorsunuz?"
```

---

## 3. SADECE ISTENENI YAP

### Eklenmeyecekler (acikca istenmemisse)

| ICERIK | TETIKLEYICI IFADELER |
|--------|---------------------|
| Testbench | "testbench yaz", "tb ekle", "simulasyon kodu yaz" |
| Unit test | "test yaz", "unit test olustur" |
| Dokumantasyon | "dokumante et", "README yaz", "aciklama dosyasi olustur" |
| Ornek kullanim | "ornek ver", "nasil kullanilir goster", "usage example" |
| Kod yorumlari | "yorum ekle", "kodun ustune aciklama yaz" |
| Error handling | "hata yonetimi ekle", "try-catch ekle" |
| Logging | "log ekle", "debug print ekle" |

### Ayri Dosya Kurali

Ek dosya istendiginde (test, tb, ornek):
1. **AYRI DOSYA** olarak olustur
2. Dosya adini **ACIKCA** belirt
3. Ana kaynak dosyasina **DOKUNMA**

```
ORNEK:
Istek: "debounce modulu ve testbench yaz"
Sonuc: debounce.vhd + debounce_tb.vhd (2 AYRI dosya)
```

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

## 4B. DOKUMANTASYON MODU (CHAT + TAM DOSYA)

Dokumantasyon dosyalari (.md/.txt/.rst/README/CHANGELOG) icin:

- Docs-Writer modeli kullan
- Apply modeli KULLANMA
- Cikti TAM DOSYA olarak ver
- Mevcut yapiyi KORU (baslik, liste, tablo)

---

## 5. BELIRSIZLIK YONETIMI

### Protokol

```
1. VARSAYIM YAPMA → Tahmin etme, uydurma
2. SORU SOR      → Net olmayan noktalari belirle
3. BEKLE         → Cevap gelene kadar isleme
```

### Ornek Sorular

- "Testbench de isteniyor mu, yoksa sadece modul mu?"
- "Clock frekansi ne olmali? (varsayilan 100 MHz kullanayim mi?)"
- "Bu mevcut dosyaya mi eklenmeli, yeni dosya mi olusturayim?"
- "Hangi VHDL standardi? (VHDL-93 mi VHDL-2008 mi?)"

---

## 6. CIKTI KALITESI

### Kod Ciktisi Formati

```
[Dosya adi acikca belirtilir]

[Dosyanin tamami - bastan sona, hicbir satir atlanmadan]
```

### Kurallar

- Aciklamalar kod blogunun **DISINDA**
- Kod blogu icinde **SADECE KOD**
- Markdown fencing dogru kullanilmali (`vhdl`, `c`, `python`, vb.)

---

## 7. ITERATIF ISLEM YASAGI

| YAPMA | YAP |
|-------|-----|
| "Once su satiri degistir..." | Tum degisiklikleri tek seferde tamamla |
| "Simdi bu adimi yap..." | Kullaniciya is verme, kendin yap |
| "Ardindan sunu ekle..." | Tum eklemeleri bir defada yap |
| "Eger X ise, Y yap..." | Belirsizlikte soru sor, varsayim yapma |

---

## 8. HATA DUZELTME VE RECOVERY PROTOKOLU

### Hata Duzeltme

```
ADIM 1: Hatanin KOK NEDENINI tani (belirtiyi degil)
ADIM 2: MINIMAL degisiklikle duzelt (gereksiz refactor yapma)
ADIM 3: TAM DOSYA olarak cikti ver
ADIM 4: Neyin neden degistigini KISA acikla
```

### Error Recovery (Islem Basarisiz Olursa)

| HATA TURU | RECOVERY STRATEJISI |
|-----------|---------------------|
| Dosya okunamadi | Kullanicidan path dogrulama iste |
| Syntax hatasi | Hatali kismi izole et, geri kalan korunur |
| Timeout | Islemi kucuk parcalara bol, sirali calistir |
| Belirsiz istek | Varsayim YAPMA, soru sor |
| Kismi basari | Basarili kismi koru, kalan icin yeni strateji |

### Fallback Kurallari

```
1. DURUMU KAYDET: Basarili adimi hatirla
2. GERI DON: En son bilinen iyi duruma don
3. ALTERNATIF DENE: Farkli yaklasim dene
4. RAPORLA: Ne basarisiz oldugunu acikla
```

---

## 9. CONTEXT YONETIMI

### Buyuk Dosyalarda

- Tum dosyayi okumadan analiz yapma
- Ilgili bolumleri tani, ama cikti TAM DOSYA olmali
- Dosya 1000+ satir ise, kullaniciya bilgi ver

### Coklu Dosya

- Dosyalar arasi bagimliligi anla
- Degisiklik sirasini dogru belirle
- Her dosyayi AYRI AYRI tam olarak yaz

---

## 10. ONCELIK SIRASI

Karar verirken:

```
1. Kullanicinin ACIK istegi         → En yuksek oncelik
2. Bu kurallardaki direktifler      → Ikinci oncelik
3. Dosya tipine ozel kurallar       → Ucuncu oncelik
4. Teknik best practice             → Dorduncu oncelik
5. Ek oneriler                      → Sadece SORU olarak sun
```

---

## 11. ILETISIM KURALLARI

### Dil

- **Aciklamalar**: Turkce
- **Teknik terimler**: Ingilizce kalabilir (FSM, ISR, async, MVVM)
- **Kod yorumlari**: Kullanici tercihine gore

---

## 12. TEKRAR YASAGI (KRITIK)

> ⚠️ Ayni bilgiyi birden fazla kez soyleme.

### Mutlak Kurallar

| YASAK | ACIKLAMA |
|-------|----------|
| Cumle tekrari | Ayni cumleyi farkli kelimelerle tekrarlama |
| Paragraf dongusu | Ayni konuyu 3+ paragrafta isleme |
| Liste kopyalama | Ayni maddeyi farkli satirlarda yazma |
| Ozet sonrasi tekrar | Ozet yaptiktan sonra ayni seyi detaylandirma |

### Tespit ve Durdurma

```
Eger yazdigin son 3 cumle ayni konuyu isliyorsa:
1. DUR
2. Son cumleyi sil
3. Tek cumlede ozetle
4. Sonraki konuya gec
```

### Tekrar Belirtileri

- "Yani...", "Kisacasi...", "Ozetle..." sonrasi ayni bilgi
- "X icin A, Y icin B" yerine "X icin A, X icin A', X icin A''"
- Farkli kelimelerle ayni sonuc: "cakisiyor", "ust uste biniyor", "ortusuyor"

### Dogru Format

```
ANALIZ: [Tek paragraf - problem ne?]
SONUC: [Tek cumle - ne yapilmali?]
KOD: [Gerekirse - ornek]
```

> **MUAFIYET**: Dokumantasyon isleri (.md/.txt/.rst) icin bu format UYGULANMAZ.
> Docs-Writer ve 06-documentation.md kurallari gecerlidir: markdown code block ciktisi.

### Stil

- Onemli bilgiyi ONE CIKAR
- Uzun paragraflar yerine LISTE/TABLO kullan
- Belirsiz ifadelerden KACIN ("muhtemelen", "belki")


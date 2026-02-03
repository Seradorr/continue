---
name: Documentation Rules
description: |
  Markdown, README ve teknik dokumantasyon dosyalari icin 
  koruma, guncelleme ve format kurallari.
  Docs-Writer modeli CHAT'e markdown olarak yazar, Apply KULLANMAZ.
globs:
  - "**/*.md"
  - "**/README*"
  - "**/CHANGELOG*"
  - "**/docs/**/*"
  - "**/*.rst"
  - "**/*.txt"
alwaysApply: false
---

# DOKUMANTASYON KURALLARI

Bu kurallar tum dokumantasyon dosyalari icin gecerlidir.

---

## 0. CIKTI YONTEMI (KRITIK)

> ⚠️ Dokumantasyon icin Apply modeli KULLANILMAZ.

### Neden?

Apply modeli kod dosyalari icin optimize edilmistir.
Dokumantasyonda icerik kaybi ve format bozulmasi yasanabilir.

### Cozum: Markdown Code Block

Docs-Writer modeli ciktisini **her zaman** markdown code block icinde verir:

```markdown
# Dokuman Basligi

Tam icerik buraya...
```

### Kullanici Akisi

1. Model: Dokumantasyonu markdown block icinde yazar
2. Kullanici: Blogu kopyalar
3. Kullanici: Dosyaya yapistirip kaydeder

Bu yontem:
- ✅ Format korunur
- ✅ Icerik kaybi olmaz
- ✅ Kullanici kontrolunde
- ✅ Apply model sorunlari bypass edilir

---

## 1. ICERIK KORUMA

> ⚠️ **TEMEL ILKE**: Kullanici ACIKCA istemedikce mevcut icerigi silme.

### Islem Kurallari

| ISTEK | EYLEM |
|-------|-------|
| "Ekle" | Mevcut + yeni = sonuc (silme yok) |
| "Guncelle" | Sadece belirtilen kisim degisir |
| "Sil" / "Kaldir" | SADECE belirtilen kisim silinir |
| "Yeniden yaz" | Belirtilen bolum tamamen degisir |

### Silme Izni

```
Kullanici su ifadeleri kullanirsa SILME YAPILABILIR:
- "X bolumunu sil"
- "X paragrafini kaldir"
- "X kismindan kurtul"
- "X'i cikar"

Bu durumda:
1. Sadece X silinir
2. Geri kalan TAMAMEN korunur
3. Cikti = (mevcut - X)
```

### Silme Icin EXPLICIT MARKER (ZORUNLU)

Dokumantasyonda silme sadece acik [DELETE] / [REMOVE] / [SIL] bloklari ile yapilir.
Blok icinde silinecek satirlar **AYNEN** verilmelidir.
Belirsiz isteklerde ("temizle", "sadelestir", "kisa tut") silme YAPMA.

```
[DELETE]
<silinecek satirlarin tamami>
[/DELETE]
```

### Korunmasi Gerekenler (silme istenmemisse)

| KURAL | ACIKLAMA |
|-------|----------|
| Once oku | Dosyanin TAMAMINI oku |
| Tumu yaz | Mevcut + degisiklik birlikte |
| Format koru | Baslik, liste, tablo yapisi degismemeli |

### Guncelleme Akisi

```
1. DOSYANIN TAMAMINI oku
        ↓
2. Mevcut icerigi HAFIZADA tut
        ↓
3. Istenen degisikligi HAFIZADA uygula
        ↓
4. TAMAMINI (eski + yeni) yaz
```

---

## 2. ISLEM TIPLERI

### "Ekle" Istegi

```
MEVCUT DOSYA + YENI ICERIK = SONUC

Ornek:
- "Changelog'a yeni giris ekle"
- "README'ye kurulum bolumu ekle"
- "Dokumantasyona ornek ekle"
```

### "Guncelle" Istegi

```
SADECE BELIRTILEN KISIM DEGISIR
GERI KALAN AYNEN KALIR

Ornek:
- "Surum numarasini guncelle"
- "Tarih bilgisini duzelt"
- "Baslik metnini degistir"
```

### "Sil" Istegi

```
Kullanici silme istediginde:

1. Silinecek kismi NET tanimla
2. O kisim CIKTIDA OLMAMALI (gercekten sil)
3. Geri kalan icerik TAMAMEN yazilmali
4. Sonuc = mevcut_dosya - silinecek_kisim

Ornek:
- "X bolumunu kaldir" → X ciktida yok, geri kalan tam
- "Bu paragrafi sil" → Paragraf yok, diger paragraflar tam
- "Yorumlari temizle" → Yorumlar yok, kod tam
```

> ⚠️ "Sildim" deyip icerikte birakma - GERCEKTEN cikar!

---

## 3. CHANGELOG / DEGISIKLIK TAKIBI

### Yeni Giris Ekleme

```markdown
# Changelog

## [1.2.0] - 2026-01-26    ← YENI GIRIS EN USTE
### Eklenenler
- Yeni ozellik

## [1.1.0] - 2026-01-15    ← ESKI GIRISLER KORUNUR
### Eklenenler
- Onceki ozellik
```

### Kurallar

- Yeni giris = En uste (kronolojik ters sira)
- Eski girisler = ASLA silinmez
- Format = Mevcut formata uy
- Tarih = ISO format (YYYY-MM-DD)

---

## 4. YAPI KORUMA

### Korunmasi Gerekenler

| ELEMENT | ORNEK |
|---------|-------|
| Baslik hiyerarsisi | # ## ### |
| Listeler | - veya 1. 2. 3. |
| Tablolar | \| Header \| |
| Kod bloklari | \`\`\`lang |
| Linkler | [text](url) |
| Bos satirlar | Paragraf ayirici |
| Girintiler | Liste alt maddeleri |

### Format Degistirme YASAK

```
YANLIS: Liste → Paragraf donusumu
YANLIS: Tablo → Liste donusumu
YANLIS: Baslik seviyesi degistirme (# → ##)
```

---

## 5. ASLA YAPILMAYACAKLAR

| YASAK | ACIKLAMA |
|-------|----------|
| "Temizleme" | Icerik silme adiyla |
| "Sadeleştirme" | Kisaltma adiyla silme |
| "Gereksiz cikarma" | Kendi kararina gore silme |
| "Duzenleme" | Format degistirerek bozma |
| "Birlestirme" | Bolumleri birlestirerek kaybetme |

---

## 6. ORNEK SENARYOLAR

### Senaryo 1: Changelog'a Ekleme

**Istek**: "Degisiklik takibi.md'ye v2.0 guncellemesini ekle"

**Dogru Yaklasim**:
1. Dosyayi oku
2. Mevcut tum icerigi koru
3. Yeni v2.0 girisini EN USTE ekle
4. Tum dosyayi (eski + yeni) yaz

### Senaryo 2: README Guncelleme

**Istek**: "README'deki kurulum adimlarini guncelle"

**Dogru Yaklasim**:
1. Dosyayi oku
2. Sadece "Kurulum" bolumunu degistir
3. Diger tum bolumler AYNEN kalir
4. Tum dosyayi yaz

### Senaryo 3: Dokumantasyona Bolum Ekleme

**Istek**: "API dokumantasyonuna yeni endpoint ekle"

**Dogru Yaklasim**:
1. Dosyayi oku
2. Mevcut endpointler korunur
3. Yeni endpoint uygun yere eklenir
4. Tum dosyayi yaz

---

## 7. DUZGUN CIKTI FORMATI

```markdown
[Dosya adi belirtilir]

[DOSYANIN TAMAMI - tum mevcut icerik + yeni icerikler]

[Hicbir kisaltma yok: "..." veya "existing content" YASAK]
```

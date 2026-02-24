---
name: Documentation Rules
description: |
  Markdown, README ve teknik dokumantasyon dosyalari icin
  koruma, guncelleme ve format kurallari.
  K2-Docs modeli agent modunda dokumantasyon dosyalarini duzenler.
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
K2-Docs modeli agent modunda calısır ve dosyalari dogrudan duzenleyebilir.

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

```
[DELETE]
<silinecek satirlarin tamami>
[/DELETE]
```

Belirsiz isteklerde ("temizle", "sadelestir", "kisa tut") silme YAPMA.

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

## 2. CHANGELOG / DEGISIKLIK TAKIBI

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

## 3. YAPI KORUMA

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

## 4. ASLA YAPILMAYACAKLAR

| YASAK | ACIKLAMA |
|-------|----------|
| "Temizleme" | Icerik silme adiyla |
| "Sadeleştirme" | Kisaltma adiyla silme |
| "Gereksiz cikarma" | Kendi kararina gore silme |
| "Duzenleme" | Format degistirerek bozma |
| "Birlestirme" | Bolumleri birlestirerek kaybetme |

---

## 5. ORNEK SENARYOLAR

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

---

## 6. CIKTI FORMATI

```markdown
[Dosya adi belirtilir]

[DOSYANIN TAMAMI - tum mevcut icerik + yeni icerikler]

[Hicbir kisaltma yok: "..." veya "existing content" YASAK]
```

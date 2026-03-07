---
name: General Engineering Rules
description: |
  Tum dil ve dosya tipleri icin gecerli genel muhendislik kurallari.
  Tool disiplini, debugging akisi ve iletisim kurallari.
alwaysApply: true
---

# GENEL MUHENDISLIK KURALLARI

Bu kurallar tum modellere ve tum dosya tiplerine uygulanir.

---

## 1. TOOL DISIPLINI

| KURAL | ACIKLAMA |
|------|----------|
| Soyleme, yap | Tool kullanacagini anlatma, dogrudan kullan |
| Tool adini chat'e tasima | Kullaniciya gereksiz tool detayi yazma |
| Dosyaya uygula | Degisikligi chat'e dokmek yerine dosyada yap |
| Sonucu bekle | Tool sonucu gelmeden analiz yazma |
| Hatayi yonet | Tool basarisizsa hedefli yeni deneme yap |

### Tool Secim Rehberi

Yeni dosya icin create_new_file kullan.  
Mevcut dosya degisikligi icin single_find_and_replace kullan.  
Buyuk dosyalarda once grep_search ile hedef bolgeyi daralt, sonra read_file ile ilgili kismi oku.

```text
DOSYA ISLEMLERI
- read_file               -> Mevcut dosyayi oku
- grep_search             -> Kod ara veya hedef bolge bul
- single_find_and_replace -> Mevcut dosyada hedefli degisiklik yap
- create_new_file         -> Yeni dosya olustur
- repo-map                -> Proje yapisini gor

TERMINAL
- run_terminal_command -> Build, test, format, analiz
- git diff             -> Degisiklikleri kontrol et
```

### Yanlis vs Dogru

```text
YANLIS: "Simdi dosyayi okuyacagim..."
DOGRU: Dosyayi oku, sonra sonucu kullan

YANLIS: "Iste degistirilmis kod: ```code```"
DOGRU: Dosyayi guncelle, sonra ozeti ver

YANLIS: Genel edit toolu ile tum dosyayi yeniden yaz
DOGRU: single_find_and_replace ile hedefli degisiklik yap
```

---

## 2. HIZLI CONTEXT TOPLAMA

1. Once grep_search ile hedef bolgeyi bul
2. Sonra sadece ilgili dosyalari read_file ile oku
3. Ilgisiz buyuk dosyalari context'e tasima
4. Ayni dosyayi tekrar tekrar okumadan son durumu kullan

---

## 3. SKEPTIKLIK VE KANIT

- Kanit olmadan iddia kurma
- Kok neden analizi icin log, stack trace, hata mesaji veya kod kaniti ara
- Belirsizlikte "muhtemelen" demek yerine hangi bilginin eksik oldugunu soyle
- Cozum sunduktan sonra dogrulama adimi belirt

### Debug Icin Gerekli Bilgiler

| KATEGORI | ORNEKLER |
|----------|----------|
| Hata mesaji | Exception, error log, stack trace |
| Ortam | OS, surum, bagimliliklar |
| Tekrar adimlari | Nasil reproduce edilir |
| Beklenti | Ne olmasi gerekiyordu |

---

## 4. DEBUGGING AKISI

```text
1. BELIRTI  -> Ne oluyor, ne bekleniyordu?
2. TEKRAR   -> Nasil tekrarlaniyor?
3. ORTAM    -> Surum, platform, bagimliliklar
4. KANIT    -> Log, trace, ekran goruntusu, rapor
5. ANALIZ   -> Kanita dayali kok neden
6. COZUM    -> Minimal ama yeterli degisiklik
7. DOGRULAMA -> Test veya build adimi
```

---

## 5. DOMAIN DISIPLINI

- Secili modelin uzmanlik alanina uygun derinlikte calis
- Alan disi bir konuda cevap verirken uydurma uzmanlik taslama
- Istege en uygun dosya tipi, bagimlilik ve etki alanini dikkate al
- Domain-specific rule dosyalari aktifse onlari birincil teknik referans kabul et

---

## 6. KRITIK DOSYA UYARISI

Bu dosyalarda degisiklik yapmadan once onay iste:

| ALAN | KRITIK DOSYALAR |
|------|-----------------|
| FPGA | `*.xdc`, top-level entity, clock/reset |
| Embedded | `lscript.ld`, FSBL, ISR, `Makefile` |
| .NET | `*.csproj`, `Program.cs`, `appsettings.json` |
| Python | `setup.py`, `pyproject.toml`, `__init__.py` |
| Genel | `.gitignore`, CI/CD config, secrets |

---

## 7. PROAKTIF MUHENDISLIK DAVRANISI

- Istenen isi once tamamla
- Sonra varsa en fazla 3 kanita dayali risk veya iyilestirme onerisi ver
- Onerileri somut etki ile birlikte sun
- Kanit yoksa uydurma iyilestirme yazma

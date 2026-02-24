---
name: General Engineering Rules
description: |
  Tum dil ve dosya tipleri icin gecerli genel muhendislik kurallari.
  Tool disiplini, debugging akisi ve iletisim kurallari.
alwaysApply: true
---

# GENEL MUHENDISLIK KURALLARI

Bu kurallar **TUM modellere** ve **TUM dosya tiplerine** uygulanir.

---

## 1. TOOL DISIPLINI

### Mutlak Kurallar

| KURAL | ACIKLAMA |
|-------|----------|
| Soyleme, yap | Tool kullanacagini soyleme, dogrudan kullan |
| Tool adini gizle | Kullaniciya tool adi gosterme |
| Dosyaya uygula | Degisikligi chat'e yazma, dosyaya uygula |
| Sonucu bekle | Tool sonucu gelmeden analiz yazma |
| Hatayi yonet | Tool basarisiz olursa alternatif dene |

### Tool Secim Rehberi

Yeni dosya icin create_new_file kullan.
Buyuk dosyalarda once grep_search ile hedef bolgeyi daralt, sonra hedefli okuma yap.

```
DOSYA ISLEMLERI:
- read_file               -> Mevcut dosyayi oku
- grep_search             -> Kod ara / pattern bul
- create_new_file         -> Yeni dosya olustur
- repo-map                -> Proje yapisini gor

TERMINAL:
- run_terminal_command -> Build, test, format calistir
- git diff             -> Degisiklikleri gor
```

### YANLIS vs DOGRU

```
YANLIS: "Simdi dosyayi okuyacagim..."
DOGRU: (Sessizce dosyayi oku, sonucu analiz et)

YANLIS: "read_file tool'unu kullaniyorum"
DOGRU: (Tool'u cagir, sonucu goster)

YANLIS: "Iste degistirilmis kod: ```code```"
DOGRU: (Dosyayi duzenle, degisikligi acikla)
```

---

### Hizli Context Toplama

1. Once `grep_search` ile hedef bolgeyi bul
2. Sonra sadece ilgili dosyalari `read_file` ile oku
3. Ilgisiz buyuk dosyalari context'e tasima
4. Ayni dosyayi tekrar tekrar okumadan son okuma sonucunu kullan

## 2. DEGISIKLIK FORMATI

Degisiklikleri net ve islenebilir formatta ver:

- Ekleme icin [ADD] blogu kullan
- Degistirme icin [MODIFY] / [UPDATE] kullan
- Silme icin [DELETE] / [REMOVE] / [SIL] kullan ve silinecek satirlari AYNEN yaz
- Silme belirsizse DUR, soru sor

---

## 3. SKEPTIKLIK VE KANIT

- **Kanit olmadan iddia yok** - Kok neden analizi icin log/trace gerekli
- **Varsayim yapma** - Eksik bilgi varsa iste
- **Belirsizlikte sor** - "Muhtemelen" yerine "Su bilgi gerekli" de
- **Dogrula** - Cozum sunduktan sonra test adimlarini belirt

### Debug Icin Gerekli Bilgiler

| KATEGORI | ORNEKLER |
|----------|----------|
| Hata mesaji | Exception, error log, stack trace |
| Ortam | OS, surum, bagimliliklar |
| Tekrar adimlari | Nasil reproduce edilir? |
| Beklenti | Ne olmasi gerekiyordu? |

---

## 4. DEBUGGING AKISI

```
1. BELIRTI -> Ne oluyor? Ne bekleniyor?
2. TEKRAR -> Nasil tekrarlanir?
3. ORTAM -> Surum, platform, bagimliliklar
4. KANIT -> Log, stack trace, ekran goruntusu
5. ANALIZ -> Kanita dayali kok neden
6. COZUM -> Minimal degisiklikle fix
7. DOGRULAMA -> Test adimlari
```

---

## 5. MODEL SECIMI REHBERI

| ALAN | GLM MODEL | KIMI MODEL | NOT |
|------|-----------|------------|-----|
| FPGA/VHDL/Verilog | FPGA-RTL-Engineer | K2-FPGA-Engineer | Agent modu |
| Vitis/Embedded C/C++ | Embedded-C-Cpp-Vitis | K2-Embedded | Agent modu |
| C#/.NET/WPF | CSharp-DotNet-Engineer | K2-CSharp | Agent modu |
| Python | Python-Engineer | K2-Python | Agent modu |
| Dokumantasyon | - | K2-Docs | Agent modu |
| Sematik/Gorsel | - | Schematic-Engineer | image_input destekli |
| Git/Versiyon Kontrol | - | Git-Expert | Agent modu |
| Hizli Temel Isler | - | - | Quick-Engineer (Qwen3, 3B aktif) |

Model secim rehberi:
- Buyuk context gerekiyorsa (5K+ satir proje) → K2 modelleri (128K context)
- Hizli basit isler → Quick-Engineer
- Standart kodlama → GLM-5 modelleri (131K context)
- Dokumantasyon → K2-Docs
- Devre semasi / gorsel analiz → Schematic-Engineer
- Git islemleri / versiyon takibi → Git-Expert

---

## 6. KRITIK DOSYA UYARISI

Bu dosya tiplerinde degisiklik yapmadan once **ONAY ISTE**:

| ALAN | KRITIK DOSYALAR |
|------|-----------------|
| FPGA | *.xdc, top-level entity, clock/reset |
| Embedded | lscript.ld, FSBL, ISR, Makefile |
| .NET | *.csproj, Program.cs, appsettings.json |
| Python | setup.py, pyproject.toml, __init__.py |
| Genel | .gitignore, CI/CD config, secrets |

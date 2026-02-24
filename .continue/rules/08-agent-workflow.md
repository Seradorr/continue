---
name: Agent Workflow Protocol
description: |
  Agent modunda tool kullanim workflow'u, multi-file koordinasyon,
  error recovery ve build/test validasyon kurallari.
alwaysApply: true
---

# AGENT WORKFLOW PROTOKOLU

Bu kurallar agent modunda tool kullanimini standardize eder.

---

## 1. TOOL SECIM KARAR AGACI

Yeni dosya icin create_new_file kullan.
Buyuk dosyalarda once grep_search ile hedef bolgeyi daralt.

```
ISTEK TIPI NEDIR?

|- YENI DOSYA olusturma
|  -> create_new_file
|
|- MEVCUT DOSYA degisikligi
|  -> Uygun edit tool ile degisikligi uygula
|  -> Birden fazla bagimsiz degisiklik: AYRI cagrilar
|  -> Bagimsiz degisiklikler: PARALEL cagri
|
|- ANALIZ / DEBUG istegi
|  -> read_file + grep_search ile bilgi topla
|  -> Sonucu kullaniciya raporla
|
|- BUILD / TEST istegi
|  -> run_terminal_command ile calistir
|  -> Ciktiyi analiz et
|
'- ARAMA istegi
   -> grep_search veya repo-map
```

---

## 2. CALISMA AKISI

### Standart Degisiklik Akisi

```
1. ANLAMA
   -> read_file ile mevcut kodu oku
   -> Yapisi ve bagimliklarini anla

2. PLANLAMA
   -> Degisiklik kapsamini belirle
   -> Etkilenen dosyalari tanimla
   -> Tool secimi yap (kapsam + risk)

3. UYGULAMA
   -> Uygun tool ile degisikligi yap
   -> Tum degisiklikleri tek yanitta tamamla

4. DOGRULAMA (gerekirse)
   -> Build/test calistirmayi oner
   -> Hata varsa duzelt
```

### Yeni Dosya Akisi

```
1. create_file ile dosyayi olustur
2. Ilgili dosyalarda guncelleme gerekiyorsa (import, reference)
   -> O dosyalari da guncelle
3. Build/test onerisi sun
```

---

## 3. MULTI-FILE KOORDINASYONU

### Bagimlilik Sirasi

```
Birden fazla dosya degisecekse:

1. BAGIMLILIK GRAFIGI
   -> Hangi dosya hangisine bagli?
   -> Import/include/reference iliskileri

2. DEGISIKLIK SIRASI
   -> En alt seviye (bagimliligi olmayan) once
   -> Bagimli dosyalar sonra

   Ornek (C#): Interface -> Implementation -> Registration
   Ornek (C):  Header -> Source -> Makefile
   Ornek (VHDL): Package -> Entity -> Testbench

3. TUTARLILIK
   -> Import/include guncel mi?
   -> Interface/implementation eslesme kontrolu
   -> Naming convention tutarli mi?
```

### Coklu Dosya Ornekleri

| SENARYO | SIRA |
|---------|------|
| Yeni C# servis | IService.cs -> Service.cs -> DI registration |
| Yeni VHDL modul | module.vhd -> (istenirse) module_tb.vhd -> top_level guncelle |
| Yeni C header/source | module.h -> module.c -> Makefile |
| Yeni Python modul | module.py -> __init__.py -> test_module.py |

---

## 4. ERROR RECOVERY

### Tool Basarisizlik Senaryolari

| HATA | RECOVERY |
|------|----------|
| Edit tool basarisiz | Degisikligi kucuk parcalara bol ve tekrar dene |
| Dosya bulunamadi | Kullanicidan path dogrulama iste |
| Timeout | Islemi kucuk parcalara bol |
| Syntax hatasi | Hatali kismi duzelt ve tekrar dene |

### Recovery Protokolu

```
1. DURUMU KAYDET
   -> Basarili adimlari hatirla
   -> Neyin basarisiz oldugunu tani

2. ALTERNATIF STRATEJI
   -> Farkli tool dene
   -> Islemi kucuk parcalara bol
   -> Kullaniciya bilgi ver

3. RAPORLA
   -> Ne basarili oldu
   -> Ne basarisiz oldu
   -> Onerilen sonraki adim
```

---

## 5. BUILD/TEST ENTEGRASYONU

### Degisiklik Sonrasi

```
FPGA projeleri:
  -> "Vivado'da sentez calistirmanizi oneririm"
  -> Sentez uyarilari beklentisi (latch, timing)

Embedded C projeleri:
  -> "Vitis'te build calistirmanizi oneririm"
  -> BSP regeneration gerekiyor mu?

C#/.NET projeleri:
  -> "dotnet build calistirmanizi oneririm"
  -> Unit test varsa: "dotnet test"

Python projeleri:
  -> "pytest calistirmanizi oneririm"
  -> Linter: "python -m flake8" veya "python -m mypy"
```

### Hata Ciktisi Analizi

```
Build/test hatasi raporlandiginda:
1. Hata mesajini oku
2. Ilgili dosya ve satiri bulmak icin grep_search kullan
3. Kok neden analizi yap
4. Fix uygula (uygun tool ile)
5. Tekrar build/test oner
```

---

## 6. CONTEXT-AWARE TOOL SECIMI

### Dosya Tipi ve Tool Eslestirme

KURAL: Dosya tipinden bagimsiz olarak degisiklik kapsamini baz al.
Mevcut dosya degisikligi icin uygun edit tool'unu kullan, yeni dosya icin create_new_file.

| DOSYA TIPI | ISLEM | NOT |
|------------|-------|-----|
| .vhd/.v (RTL) | Hedefli degisiklik | Sentez uyumluluk kontrolu |
| .c/.h (Embedded) | Hedefli degisiklik | Header/source tutarliligi |
| .cs (C#) | Hedefli degisiklik | Interface/impl eslesmesi |
| .py (Python) | Hedefli degisiklik | Import tutarliligi |
| .md (Docs) | Hedefli degisiklik | Yapi koruma |
| .xdc (Constraint) | Genellikle tek satir | Kritik dosya - ONAY iste |
| .csproj/.sln | Konfigurasyon | Kritik dosya - ONAY iste |
| Makefile | Genellikle tek kural | Build sistemi degisikligi |

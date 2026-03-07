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

```text
ISTEK TIPI NEDIR?

|- YENI DOSYA
|  -> create_new_file
|
|- MEVCUT DOSYA DEGISIKLIGI
|  -> single_find_and_replace ile degisikligi uygula
|  -> Birden fazla bagimsiz degisiklik varsa ayri cagrilar yap
|  -> Bagimsiz degisikliklerde paralel calisma dusun
|
|- ANALIZ veya DEBUG
|  -> read_file + grep_search ile bilgi topla
|  -> Sonucu raporla
|
|- BUILD veya TEST
|  -> run_terminal_command ile calistir
|  -> Ciktiyi analiz et
|
'- ARAMA
   -> grep_search veya repo-map
```

---

## 2. CALISMA AKISI

### Standart Degisiklik Akisi

```text
1. ANLAMA
   -> read_file ile mevcut kodu oku
   -> Yapisi ve bagimliklarini anla

2. PLANLAMA
   -> Degisiklik kapsamini belirle
   -> Etkilenen dosyalari tanimla
   -> Riskli dosya var mi kontrol et

3. UYGULAMA
   -> Mevcut dosyada single_find_and_replace kullan
   -> Yeni dosyada create_new_file kullan
   -> Tum kritik degisiklikleri tamamla

4. DOGRULAMA
   -> Gerekirse build veya test oner
   -> Hata varsa duzelt
```

### Yeni Dosya Akisi

```text
1. create_new_file ile dosyayi olustur
2. Gerekirse ilgili import veya referanslari guncelle
3. Build veya test adimini oner
```

---

## 3. MULTI-FILE KOORDINASYONU

```text
Birden fazla dosya degisecekse:
1. Bagimlilik grafigi cikar
2. Degisiklik sirasini belirle
3. Tutarlilik kontrolunu planla
4. Dosyalari sirayla guncelle
5. Son durumda import, include, interface ve reference kontrolu yap
```

### Coklu Dosya Ornekleri

| SENARYO | SIRA |
|---------|------|
| Yeni C# servis | Interface -> Implementation -> Registration |
| Yeni VHDL modul | module.vhd -> istenirse module_tb.vhd -> top-level |
| Yeni C header/source | module.h -> module.c -> Makefile |
| Yeni Python modul | module.py -> __init__.py -> test_module.py |

---

## 4. ERROR RECOVERY

| HATA | RECOVERY |
|------|----------|
| single_find_and_replace basarisiz | Degisikligi kucuk parcalara bol ve tekrar dene |
| Dosya bulunamadi | Kullanicidan path dogrulama iste |
| Timeout | Islemi kucuk parcalara bol |
| Syntax hatasi | Hatali kismi duzelt ve tekrar dene |

### Recovery Protokolu

```text
1. DURUMU KAYDET
   -> Basarili adimlari ayir
   -> Basarisiz adimi tani

2. ALTERNATIF STRATEJI
   -> Daha hedefli degisiklik uygula
   -> Gerekirse ek okuma yap
   -> Kullaniciyi yalnizca kritik yerde bilgilendir

3. RAPORLA
   -> Ne basarili oldu
   -> Ne kaldi
   -> En guvenli sonraki adim ne
```

---

## 5. BUILD/TEST ENTEGRASYONU

```text
FPGA projeleri:
  -> Vivado sentez veya warning kontrolunu oner

Embedded C projeleri:
  -> Vitis build veya ilgili komutu oner

C#/.NET projeleri:
  -> dotnet build
  -> gerekiyorsa dotnet test

Python projeleri:
  -> pytest
  -> gerekiyorsa linter veya type check
```

### Hata Ciktisi Analizi

```text
Build veya test hatasi raporlandiginda:
1. Hata mesajini oku
2. Ilgili dosya ve satiri grep_search ile bul
3. Kok neden analizi yap
4. Mevcut dosyada single_find_and_replace ile fix uygula
5. Tekrar build veya test oner
```

---

## 6. CONTEXT-AWARE TOOL SECIMI

Kural: Dosya tipinden bagimsiz olarak degisiklik kapsamini baz al.  
Mevcut dosya degisikligi icin single_find_and_replace kullan, yeni dosya icin create_new_file kullan.

| DOSYA TIPI | ISLEM | NOT |
|------------|-------|-----|
| `.vhd`, `.v`, `.sv` | Hedefli degisiklik | Sentez uyumluluk kontrolu |
| `.c`, `.h` | Hedefli degisiklik | Header/source tutarliligi |
| `.cs` | Hedefli degisiklik | Interface ve registration etkisi |
| `.py` | Hedefli degisiklik | Import ve packaging etkisi |
| `.md` | Hedefli degisiklik | Yapi koruma |
| `.xdc` | Genellikle tek satir | Kritik dosya, once onay |
| `.csproj`, `.sln` | Konfigurasyon | Kritik dosya, once onay |
| `Makefile` | Build sistemi | Etkiyi acikca dusun |

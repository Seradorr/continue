# Continue AI Configuration - Production-Ready Local LLM Setup

**Version**: 7.2.0  
**Last Updated**: 2026-03-26

Bu repo, Continue IDE eklentisini lokal ve profesyonel bir yazilim gelistirme asistani olarak kullanmak icin hazirlanmistir. Cekirdek calisma modeli mimarisi `Hard-Engineer`, `Soft-Engineer`, `Vision-Engineer` ve `GLM-Hard-Engineer` etrafinda sade tutulur. Uzmanlik ise agirlikli olarak rule katmaninda tanimlanir.

Konfigurasyonun amaci, model secimini kullaniciya birakan; davranis, uzmanlik, tool disiplini ve kalite barini ise config + rule katmaninda standartlastiran kalici bir yapi sunmaktir.

---

## Amac

Bu proje su hedeflere odaklanir:

| Hedef | Aciklama |
|-------|----------|
| Copilot benzeri deneyim | Kod yazma, duzeltme, debugging ve refactoring |
| Uzmanlik alanlari | FPGA/RTL, Embedded C/C++, C#/.NET, Python |
| Teknik dokumantasyon | README, CHANGELOG, kullanim kilavuzlari |
| Gorsel yorum | Sematik, blok diyagram, pin mapping |
| Guvenli git akisi | Commit, branch, merge, rebase, conflict rehberligi |

---

## Model Seti

| # | Model Adi | Backend Model | Rol | Kullanim |
|---|-----------|---------------|-----|----------|
| 1 | Hard-Engineer | Qwen3.5-397B-A17B-FP8 | chat, edit | Derin analiz, kritik degisiklik, production odakli isler |
| 2 | Soft-Engineer | Qwen3.5-35B-A3B | chat, edit | Hizli bug fix, kisa refactor, hafif analiz |
| 3 | Vision-Engineer | Kimi-K2.5 | chat, edit | Gorsel yorum, sematik, tablo, uzun context |
| 4 | GLM-Hard-Engineer | GLM-4.7-FP8 | chat, edit | GLM tabanli hard-engine davranisi |
| 5 | Rerank-Model | bge-reranker-v2-m3 | rerank | Arama sonucu siralama |
| 6 | Embedding-Model | Qwen3-Embedding-8B | embed | Vektorlestirme |

---

## Tasarim Ilkeleri

### 1. Cekirdek Motor + Rule Katmani

- Model sayisi bilincli olarak az tutulur; cekirdek davranis motorlari ayridir
- Domain uzmanligi agirlikli olarak `.continue/rules/` katmaninda tanimlanir
- Kullanici gerekirse ilgili rule dosyalarini manuel olarak da ekleyebilir

### 2. Tool Disiplini

- Mevcut dosya degisikligi icin `single_find_and_replace` hedeflenir
- Yeni dosya icin `create_new_file` kullanilir
- Buyuk dosyalarda once arama, sonra hedefli degisiklik akisi uygulanir

### 3. Proaktif Muhendislik Davranisi

- Model once istenen isi tamamlar
- Ardindan varsa en fazla 3 kanita dayali risk, anti-pattern veya refactor onerisi sunar
- Oneri yoksa gereksiz liste uretmez

### 4. Production Kalite Bagi

- Mimari etki dusunulur
- Verification adimi belirtilir
- Tool cikti hijyeni korunur
- Prompt injection ve secret hijyeni dikkate alinir

---

## Uzmanlik Alanlari

### FPGA / RTL

- `02-fpga.md` ile VHDL, Verilog, SystemVerilog, FSM, CDC, timing, AXI ve XDC kurallari uygulanir

### Embedded / Vitis

- `03-vitis.md` ile Xilinx driver API, interrupt, ISR, DMA, cache, BSP ve linker disiplini uygulanir

### C# / .NET

- `04-csharp.md` ile modern C#, async/await, WPF MVVM, DI ve EF Core odagi uygulanir

### Python

- `05-python.md` ile type hints, pathlib, asyncio, pytest, packaging ve Pydantic v2 odagi uygulanir

### Dokumantasyon

- `06-documentation.md` ile README, CHANGELOG, API ve teknik dokumanlarda icerik koruma ve hedefli guncelleme uygulanir

### Git

- `10-git.md` ile durum analizi, commit hijyeni, branch guvenligi, conflict ve veri kaybi riski odagi manuel olarak eklenebilir

### Gorsel ve Uzun Context

- `Vision-Engineer` sematik, blok diyagram, tablo, pin mapping ve uzun baglam gerektiren islerde kullanilir

---

## Rule Hiyerarsisi

```text
.continue/rules/
|- 00-core.md                -> Temel agent protokolu
|- 01-general.md             -> Genel muhendislik kurallari
|- 02-fpga.md                -> FPGA / RTL kurallari
|- 03-vitis.md               -> Vitis / Embedded C kurallari
|- 04-csharp.md              -> C# / .NET kurallari
|- 05-python.md              -> Python kurallari
|- 06-documentation.md       -> Dokumantasyon kurallari
|- 07-reasoning.md           -> Gelismis akil yurutme
|- 08-agent-workflow.md      -> Agent tool workflow
|- 09-production-quality.md  -> Mimari, guvenlik ve verification kalitesi
'- 10-git.md                 -> Git durum, commit ve branch guvenligi
```

`alwaysApply` katmaninda ozellikle sunlar zorunlu hale getirilir:
- mevcut dosya degisikliginde `single_find_and_replace`
- yeni dosyada `create_new_file`
- isi bitirdikten sonra kanita dayali en fazla 3 iyilestirme onerisi
- tool cikti hijyeni
- verification ve mimari etki kontrolu

---

## Kurulum

1. Continue eklentisini VS Code'a kurun.
2. Bu repository'yi klonlayin:

   ```bash
   git clone https://github.com/Seradorr/continue.git
   ```

3. `config.yaml` icindeki API ayarlarini kendi ortaminiza gore duzenleyin:

   ```yaml
   api_config: &api_config
     provider: openai
     apiBase: "http://your-api-base:8000/v1"
     apiKey: "your-api-key"
   ```

4. `.continue/rules/` klasorunu workspace icinde kullanin.

---

## vLLM Referans Sunucu Ayarlari

### GLM-4.7

```bash
vllm serve GLM-4.7-FP8 \
  --tool-call-parser glm47 \
  --reasoning-parser glm45 \
  --enable-auto-tool-choice
```

### Kimi-K2.5

```bash
vllm serve Kimi-K2.5 \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --enable-auto-tool-choice
```

### Qwen Ailesi

```text
Qwen3.5-397B-A17B-FP8
Qwen3.5-35B-A3B
```

Kullanim ayrimi:
- `Hard-Engineer` -> kalite odakli, daha derin muhakeme
- `Soft-Engineer` -> hizli ve hedefli degisiklik

---

## Sampling ve Context Ayarlari

### GLM-4.7

| Parametre | Deger |
|-----------|-------|
| `temperature` | 0.7 |
| `frequency_penalty` | 0.2 |
| `presence_penalty` | 0.1 |
| `top_p` | 0.95 |
| `repetition_penalty` | 1.1 |
| `maxTokens` | 32768 |
| `contextLength` | 131072 |
| `reasoning` | true |
| `reasoningBudgetTokens` | 8192 |
| `truncate_prompt_tokens` | 98000 |

### Kimi-K2.5

| Parametre | Deger |
|-----------|-------|
| `temperature` | 1.0 |
| `top_p` | 0.95 |
| `maxTokens` | 32768 |
| `contextLength` | 131072 |
| `truncate_prompt_tokens` | 98000 |

### Hard-Engineer

| Parametre | Deger |
|-----------|-------|
| `temperature` | 0.7 |
| `top_p` | 0.95 |
| `maxTokens` | 32768 |
| `contextLength` | 131072 |
| `reasoning` | true |
| `reasoningBudgetTokens` | 8192 |
| `truncate_prompt_tokens` | 98000 |

### Soft-Engineer

| Parametre | Deger |
|-----------|-------|
| `temperature` | 0.0 |
| `top_p` | 0.95 |
| `maxTokens` | 32768 |
| `contextLength` | 131072 |
| `truncate_prompt_tokens` | 98000 |

### Neden Daha Genis Headroom Birakildi

- System prompt, rules, tool schema ve reasoning cikisi icin ek bosluk gerekir
- Asiri dar `truncate_prompt_tokens` ayarlari tool calling ve uzun context stabilitesini dusurebilir
- Bu nedenle marjlar production kullanima gore daha guvenli hale getirildi

---

## Kritik Davranislar

### Testbench Ayrimi

- Sadece RTL istendiyse sadece RTL dosyasi olustur
- Testbench acikca istendiyse ayri dosyada olustur

### Dokumantasyon Koruma

- README ve CHANGELOG gibi dosyalarda mevcut icerik korunur
- Yeni girisler hedefli eklenir
- Format bozulmaz

### Tool Hijyeni

- `single_find_and_replace` mevcut dosya degisikliginde ana arac olarak hedeflenir
- `create_new_file` yeni dosya icin kullanilir
- Ham tool marker veya parser tokenlari kullanici gorunumlu ciktiya tasinmaz

### Verification

- Bug fix, refactor veya davranis degisimi varsa uygun build veya test adimi belirtilir
- Verification onerileri teknolojiye ve degisen alana uygun olmali

---

## Bilinen Operasyonel Notlar

### Kimi-K2.5 Tool Token Leakage

Belirti:
- Raw tool tokenlari bazen ciktiya sizabilir
- Thinking -> tool gecisinde gorulebilir

Pratik yaklasim:
- vLLM parser davranisini kontrol et
- Tool cagrilarini hedefli ve tekil tut
- Buyuk yeniden yazmalar yerine hedefli degisiklik akisini koru

### GLM-4.7

- `GLM-Hard-Engineer`, hard-engine davranisini GLM tarafinda korur
- Tool parser ve reasoning parser ayarlari referans olarak belgelenmistir
- Sampling tarafinda `frequency_penalty: 0.2` ve `presence_penalty: 0.1` korunur

---

## Dosya Yapisi

```text
.
|- config.yaml
|- .continue/
|  '- rules/
|     |- 00-core.md
|     |- 01-general.md
|     |- 02-fpga.md
|     |- 03-vitis.md
|     |- 04-csharp.md
|     |- 05-python.md
|     |- 06-documentation.md
|     |- 07-reasoning.md
|     |- 08-agent-workflow.md
|     |- 09-production-quality.md
|     '- 10-git.md
'- README.md
```

---

## Versiyon Gecmisi

### v7.2.0 - 2026-03-07

- Rule katmani model-secim tavsiyesi veren dilden arindirildi
- `single_find_and_replace` ve proaktif kalite davranisi butun uzman ailelere standart hale getirildi
- Production kalite, security ve verification odakli yeni `09-production-quality.md` eklendi
- `truncate_prompt_tokens` degerleri tool/rule/reasoning headroom'u icin guvenli seviyeye cekildi
- Python, C# ve FPGA domain rule iceriklerinde modern best practice guncellemeleri yapildi

### v7.1.2 - 2026-03-07

- GLM-5 uzman modelleri model listesine geri eklendi
- GLM ve Kimi uzman aileleri birlikte korunacak sekilde README ve config duzenlendi
- `single_find_and_replace` ve proaktif kalite davranislari GLM tarafina da tasindi

### v7.1.0 - 2026-03-07

- `single_find_and_replace` kural seviyesinde acik ve zorunlu hale getirildi
- `apply` kullanmama tercihi dokumante edildi
- Rule sistemi korunup daha pratik kullanima gore guncellendi
- Istenen isi tamamladiktan sonra en fazla 3 kanita dayali iyilestirme onerisi davranisi eklendi

### v7.0.0 - 2026-02-24

- GLM-5 gecis hazirligi
- Kimi-K2.5 uzman modelleri
- Schematic-Engineer ve Git-Expert eklemeleri

---

## Lisans

Bu proje MIT lisansi altinda sunulmaktadir.

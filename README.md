# Continue AI Configuration - Professional Local LLM Setup

**Version**: 7.0.0
**Last Updated**: 2026-02-24

Lokal olarak çalışan LLM'leri (GLM-5, Kimi-K2.5, Qwen3) GitHub Copilot seviyesinde kod üretimi, debugging, refactoring ve dokümantasyon için optimize eden Continue IDE eklentisi yapılandırması.

---

## 🎯 Amaç

Bu proje, **lokal LLM'lerin profesyonel yazılım geliştirme asistanı olarak kullanılmasını** sağlar:

| Hedef | Açıklama |
|-------|----------|
| **GitHub Copilot Seviyesi** | Kod tamamlama, debugging, refactoring kalitesi |
| **Uzmanlık Alanları** | FPGA/RTL, Embedded C, C#/.NET, Python |
| **Mimari Danışmanlık** | Proje analizi, pattern tespiti, iyileştirme önerileri |
| **Dokümantasyon** | README, Changelog, API docs oluşturma/güncelleme |
| **Şematik/Görsel Analiz** | Devre şeması, blok diyagram, Excel pin mapping okuma |
| **Versiyon Kontrol** | Git operasyonları, commit analizi, branch yönetimi |

---

## 🏗️ Mimari

### Model Yapılandırması

| # | Model | Rol | Context | Kullanım Alanı |
|---|-------|-----|---------|----------------|
| 1-4 | **GLM-5-FP8** | Uzmanlık modelleri | 131K | FPGA, Vitis, C#, Python |
| 5-9 | **Kimi-K2.5** | Uzmanlık modelleri | 128K | FPGA, Vitis, C#, Python, Docs |
| 10 | **Kimi-K2.5** | Schematic-Engineer | 128K | Şematik/görsel okuma (image_input) |
| 11 | **Kimi-K2.5** | Git-Expert | 128K | Versiyon kontrol yönetimi |
| 12 | **Qwen3-Next-80B** | Quick-Engineer | 262K | Hızlı temel işler (3B aktif MoE) |
| 13 | **Qwen3-Coder-480B** | Rerank | 131K | Arama sonucu sıralama |
| 14 | **Qwen3-Embedding-8B** | Embed | - | Vektörleştirme |

### GLM-5-FP8 Bilgileri

| Özellik | Değer |
|---------|-------|
| Toplam Parametre | 744B |
| Aktif Parametre | 40B (MoE, 256 expert) |
| Mimari | GlmMoeDsaForCausalLM (MoE + Sparse MLA / DSA) |
| Pre-training | 28.5T token |
| vLLM Parser | `--tool-call-parser glm47 --reasoning-parser glm45` |
| GPU Gereksinimi | H100/H200+ (sm90+, A100 desteklenmiyor) |
| MTP (Speculative) | Tool calling ile uyumsuz — kapalı tutun |

### Kimi-K2.5 Bilgileri

| Özellik | Değer |
|---------|-------|
| Thinking Mode | AKTIF (varsayılan, kapatmayın — model kalitesi düşer) |
| Temperature | 1.0 (Moonshot resmi önerisi, thinking ON) |
| vLLM Parser | `--tool-call-parser kimi_k2 --reasoning-parser kimi_k2` |
| reasoning_effort | Desteklenmiyor (binary ON/OFF) |
| Bilinen Sorun | Tool token leakage (vLLM PR #34955 bekliyor) |

### Rules Hiyerarşisi

```
.continue/rules/
├── 00-core.md          # Temel protokol (alwaysApply)
├── 01-general.md       # Genel mühendislik (alwaysApply)
├── 02-fpga.md          # VHDL/Verilog (glob: *.vhd, *.v, *.sv)
├── 03-vitis.md         # Embedded C (glob: *.c, *.h)
├── 04-csharp.md        # C#/.NET (glob: *.cs, *.xaml)
├── 05-python.md        # Python (glob: *.py)
├── 06-documentation.md # Markdown (glob: *.md)
└── 07-reasoning.md     # Gelişmiş akıl yürütme (alwaysApply)
```

---

## 🔑 Temel Özellikler

### 1. Uzmanlık Modelleri
Her alan için optimize edilmiş system message ve agent prompt:
- **FPGA-RTL-Engineer**: FSM, sentez, timing, CDC, AXI, Vivado
- **Embedded-C-Cpp-Vitis**: Zynq, BSP, DMA, ISR, cache
- **CSharp-DotNet-Engineer**: async/await, MVVM, WPF, SOLID
- **Python-Engineer**: PEP 8, type hints, pathlib, pytest
- **Schematic-Engineer**: Devre şeması, blok diyagram, Excel, image_input
- **Git-Expert**: Commit, branch, merge, rebase, conflict resolution

### 2. Testbench Ayrımı
RTL dosyalarına testbench otomatik **eklenmez**:
- Sadece `module.vhd` istenirse → sadece `module.vhd`
- Testbench istenirse → ayrı `module_tb.vhd` dosyası

### 3. Dokümantasyon Koruması
Markdown dosyalarında mevcut içerik **silinmez**:
- "Ekle" = mevcut + yeni
- Changelog yeni giriş = en üste
- Format (başlık, liste, tablo) korunur

### 4. Tekrar Yasağı ve Loop Önleme
- Aynı cümle farklı kelimelerle **yasak**
- Dolgu kelimeleri yasak (Hmm, Let me think, Wait)
- Adaptif düşünme: basit istek 3-5 adım, derin analiz 8-12 adım
- Atomik çıktı: tüm değişiklikler tek yanıtta

### 5. Error Recovery
İşlem başarısız olursa:
- Başarılı adımı koru
- Alternatif strateji dene
- Kullanıcıya net bilgi ver

---

## 📁 Dosya Yapısı

```
.
├── config.yaml              # Ana yapılandırma dosyası
├── .continue/
│   └── rules/
│       ├── 00-core.md       # Temel agent protokolü
│       ├── 01-general.md    # Genel mühendislik kuralları
│       ├── 02-fpga.md       # FPGA/RTL kuralları
│       ├── 03-vitis.md      # Vitis/Embedded C kuralları
│       ├── 04-csharp.md     # C#/.NET kuralları
│       ├── 05-python.md     # Python kuralları
│       ├── 06-documentation.md # Dokümantasyon kuralları
│       └── 07-reasoning.md  # Gelişmiş akıl yürütme
└── README.md                # Bu dosya
```

---

## ⚙️ Kurulum

1. **Continue eklentisini** VS Code'a kurun
2. Bu repository'yi klonlayın:
   ```bash
   git clone https://github.com/Seradorr/continue.git
   ```
3. `config.yaml` içindeki API ayarlarını düzenleyin:
   ```yaml
   api_config: &api_config
     provider: openai
     apiBase: "http://your-api-base:8000/v1"
     apiKey: "your-api-key"
   ```
4. `.continue/rules/` klasörünü VS Code workspace'inize kopyalayın

### vLLM Sunucu Yapılandırması

**GLM-5-FP8:**
```bash
vllm serve GLM-5-FP8 \
  --tensor-parallel-size 8 \
  --gpu-memory-utilization 0.85 \
  --tool-call-parser glm47 \
  --reasoning-parser glm45 \
  --enable-auto-tool-choice
# NOT: --speculative-config (MTP) tool calling ile uyumsuz, eklemeyin
```

**Kimi-K2.5:**
```bash
vllm serve Kimi-K2.5 \
  --tool-call-parser kimi_k2 \
  --reasoning-parser kimi_k2 \
  --enable-auto-tool-choice
# Thinking mode varsayılan AÇIK, kapatmayın
```

---

## 🔧 Yapılandırma Seçenekleri

### Temperature Ayarları

| Model | Temperature | Neden |
|-------|-------------|-------|
| GLM-5 (Tüm uzmanlıklar) | 0.7 | Z.ai resmi tool-calling önerisi |
| Kimi-K2.5 (Thinking ON) | 1.0 | Moonshot resmi önerisi |
| Quick-Engineer (Qwen3) | 0.5 | Hızlı, dengeli çıktı |
| Rerank | 0.1 | Tutarlı sıralama |

### Sampling Parametreleri

**GLM-5 (vLLM):**

| Parametre | Değer | Açıklama |
|-----------|-------|----------|
| `frequency_penalty` | 0.1 | Tekrar eden token'lara hafif ceza |
| `presence_penalty` | 0.1 | Yeni token çeşitliliği |
| `top_p` | 0.95 | Neredeyse tüm tokenler dahil |
| `repetition_penalty` | 1.1 | Loop önleme |
| `truncate_prompt_tokens` | 98000 | Context overflow önleme |

**Kimi-K2.5:**

| Parametre | Değer | Açıklama |
|-----------|-------|----------|
| `top_p` | 0.95 | Moonshot resmi önerisi |
| `truncate_prompt_tokens` | 97000 | 131K - 32K = ~98K, güvenli 97K |

### Token Limitleri

| Model | contextLength | maxTokens | truncate |
|-------|---------------|-----------|----------|
| GLM-5-FP8 | 131072 | 32768 | 98000 |
| Kimi-K2.5 | 131072 | 32768 | 97000 |
| Qwen3-Next-80B | 262144 | 32768 | 229000 |

> **Context Overflow Koruması:** `truncate_prompt_tokens` ile input otomatik kesilir, session düşmesi önlenir.

---

## 🐛 Yaşanan Sorunlar ve Çözümler

### GLM-4.7 → GLM-5 Geçişi

GLM-5 (744B/40B MoE) Z.ai tarafından yayınlandı. Önemli notlar:
- vLLM v0.16+ gerekli
- Parser isimleri geriye uyumlu: `glm47`, `glm45`
- MTP (speculative decode) tool calling ile **uyumsuz** — kapalı tutun
- H100/H200+ GPU gerekli (sm90+)

### GLM Sonsuz Düşünme Döngüsü (Infinite Thinking Loop)

**Sorun:** GLM modeli thinking modunda sonsuz döngüye giriyordu.

**Çözüm:**
- `repetition_penalty: 1.1` — loop önleme
- `frequency_penalty: 0.1` — tekrar eden token cezası
- Prompt kuralları: dolgu kelimeleri yasak, tekrar yasağı
- Adaptif düşünme limitleri (basit: 3-5, derin: 8-12 adım)

### Kimi K2.5 Tool Token Leakage

**Sorun:** `<|tool_call_begin|> ... <|tool_call_end|>` gibi özel tokenler çıktıya sızıyor.

**Durum:**
- PR #28543 (Kasım 2025, merged): Streaming mode sızıntısı düzeltildi
- PR #34955 (Şubat 2026, açık): KimiK25ReasoningParser — thinking→tool geçişi
- PR #34968 (Şubat 2026, açık): 8K buffer limiti kaldırma

**Workaround:** vLLM'i en güncel sürüme güncelleyin. Thinking modu **kapatmayın** — model kalitesi düşer.

### Premature Close (Bağlantı Kopması)

**Sorun:** API bağlantısı erken kapanıyordu.

**Sonuç:** Backend kaynaklı sorun olarak tespit edilip çözüldü.

---

## 📋 Versiyon Geçmişi

### v7.0.0 (2026-02-24)
- **BREAKING:** GLM-4.7-FP8 → **GLM-5-FP8** geçişi (744B/40B MoE, DSA)
- **Kimi-K2.5 Context:** 262K → **128K** (sunucu konfigürasyonu)
- **Kimi-K2.5 Thinking:** temperature=1.0 (Moonshot resmi), thinking AÇIK
- **Kimi-K2.5 maxTokens:** 65536 → **32768**
- **Yeni Model:** Schematic-Engineer (Kimi-K2.5, image_input, şematik/görsel okuma)
- **Yeni Model:** Git-Expert (Kimi-K2.5, versiyon kontrol uzmanı)
- **Rule Güncellemeleri:**
  - 00-core.md: GLM-5 referansları, loop önleme referans sadeleştirme
  - 01-general.md: Model seçim tablosu güncellendi
  - 07-reasoning.md: GLM-5 ve Kimi-K2.5 referansları
- **Güvenlik:** Kurumsal referanslar jenerikleştirildi
- **Premature Close:** Çözüldü (backend kaynaklı)

### v6.2.0 (2026-02-20)
- Premature close debug parametreleri eklendi
- Kimi-K2.5 thinking mode varsayılan AÇIK
- Temperature optimizasyonu (thinking=1.0)
- Request defaults anchor yapısı

### v4.4.0 (2026-02-05)
- Atomik Çıktı Protokolü eklendi
- Loop önleme parametreleri güncellendi

### v4.3.0 (2026-02-04)
- İlk açık kaynak yapısı
- Rules hiyerarşisi, 8 rule dosyası

---

## 📄 Lisans

Bu proje MIT lisansı altında sunulmaktadır.

---

## 🙏 Teşekkürler

- [Continue](https://continue.dev) - VS Code AI asistan eklentisi
- [Z.ai](https://z.ai) - GLM-5 modeli
- [Moonshot AI](https://github.com/MoonshotAI) - Kimi-K2.5 modeli
- [vLLM](https://github.com/vllm-project/vllm) - Inference engine
- Lokal LLM toplulukları (Reddit r/LocalLLaMA, GitHub)

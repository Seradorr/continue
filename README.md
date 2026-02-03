# Continue AI Configuration - Professional Local LLM Setup

**Version**: 4.3.0  
**Last Updated**: 2026-02-04

Lokal olarak çalışan LLM'leri (GLM-4.7, Qwen3) GitHub Copilot seviyesinde kod üretimi, debugging, refactoring ve dokümantasyon için optimize eden Continue IDE eklentisi yapılandırması.

---

## 🎯 Amaç

Bu proje, **lokal LLM'lerin profesyonel yazılım geliştirme asistanı olarak kullanılmasını** sağlar:

| Hedef | Açıklama |
|-------|----------|
| **GitHub Copilot Seviyesi** | Kod tamamlama, debugging, refactoring kalitesi |
| **Uzmanlık Alanları** | FPGA/RTL, Embedded C, C#/.NET, Python |
| **Mimari Danışmanlık** | Proje analizi, pattern tespiti, iyileştirme önerileri |
| **Dokümantasyon** | README, Changelog, API docs oluşturma/güncelleme |
| **Tam Dosya Çıktısı** | Kısmi diff yerine tam dosya yazımı (API modelleri için) |

---

## 🏗️ Mimari

### Model Yapılandırması

| Model | Rol | Context | Kullanım Alanı |
|-------|-----|---------|----------------|
| **GLM-4.7-FP8** | Tüm uzmanlık modelleri | 131K | FPGA, Vitis, C#, Python, Docs, Advisor |
| **Qwen3-Next-80B** | Apply-Model | 262K | Deterministik kod birleştirme |
| **Qwen3-Coder-480B** | Rerank | 131K | Arama sonucu sıralama |
| **Qwen3-Embedding-8B** | Embed | - | Vektörleştirme |

> **Not:** Kimi-K2.5 tool token leakage sorunu nedeniyle şu an kullanılmıyor. vLLM PR #28543 merge edildi, güncelleme sonrası tekrar denenecek.

### Rules Hiyerarşisi

```
.continue/rules/
├── 00-core.md          # Temel protokol (alwaysApply)
├── 01-general.md       # Genel mühendislik (alwaysApply)
├── 02-fpga.md          # VHDL/Verilog (glob: *.vhd, *.v)
├── 03-vitis.md         # Embedded C (glob: *.c, *.h)
├── 04-csharp.md        # C#/.NET (glob: *.cs, *.xaml)
├── 05-python.md        # Python (glob: *.py)
├── 06-documentation.md # Markdown (glob: *.md)
└── 07-reasoning.md     # Gelişmiş akıl yürütme (alwaysApply)
```

---

## 🔑 Temel Özellikler

### 1. Tam Dosya Yazımı
API tabanlı modeller kısmi diff'leri güvenilir uygulayamaz. Bu yapılandırma:
- Her değişiklikte **dosyanın tamamını** yazar
- `...` veya `// existing code` gibi kısaltmaları **yasaklar**
- Apply-Model ile deterministik birleştirme sağlar

### 2. Uzmanlık Modelleri
Her alan için optimize edilmiş system message:
- **FPGA-RTL-Engineer**: FSM, sentez, timing, Vivado
- **Embedded-C-Cpp-Vitis**: Zynq, BSP, DMA, ISR
- **CSharp-DotNet-Engineer**: async/await, MVVM, WPF
- **Python-Engineer**: PEP 8, type hints, pathlib

### 3. Testbench Ayrımı
RTL dosyalarına testbench otomatik **eklenmez**:
- Sadece `module.vhd` istenirse → sadece `module.vhd`
- Testbench istenirse → ayrı `module_tb.vhd` dosyası

### 4. Dokümantasyon Koruması
Markdown dosyalarında mevcut içerik **silinmez**:
- "Ekle" = mevcut + yeni
- Changelog yeni giriş = en üste
- Format (başlık, liste, tablo) korunur

### 5. Tekrar Yasağı
Model çıktısında tekrar önleme:
- Aynı cümle farklı kelimelerle **yasak**
- Loop tespiti ve otomatik durdurma
- ANALIZ → SONUÇ → KOD formatı

### 6. Error Recovery
İşlem başarısız olursa:
- Başarılı adımı koru
- Alternatif strateji dene
- Kullanıcıya net bilgi ver

---

## 📁 Dosya Yapısı

```
continue-main/
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
   git clone <repo-url>
   ```
3. `config.yaml` içindeki API ayarlarını düzenleyin:
   ```yaml
   api_config: &api_config
     provider: openai
     apiBase: <your-api-base>
     apiKey: <your-api-key>
   ```
4. `.continue/rules/` klasörünü VS Code workspace'inize kopyalayın

---

## 🔧 Yapılandırma Seçenekleri

### Temperature Ayarları

| Model | Temperature | Neden |
|-------|-------------|-------|
| GLM-4.7 (Tüm uzmanlıklar) | 0.3 | Loop önleme + yaratıcılık dengesi |
| Apply-Model | 0.0 | Deterministik birleştirme |
| Rerank | 0.1 | Tutarlı sıralama |

### vLLM Extra Parametreleri (Loop Önleme)

| Parametre | Değer | Açıklama |
|-----------|-------|----------|
| `repetition_penalty` | 1.0 | KAPALI - GLM loop önleme için zorunlu |
| `min_p` | 0.01 | Düşük eşik = çeşitlilik |
| `top_k` | 20 | Stabil çıktı |
| `frequency_penalty` | 0.0 | Kod için sıfır |
| `presence_penalty` | 0.0 | Kod için sıfır |
| `top_p` | 1.0 | Tüm tokenler dahil |

### Timeout Ayarları

| Model | Timeout | Neden |
|-------|---------|-------|
| Advisor | 600s | Büyük proje analizi |
| FPGA/Vitis/C#/Python/Docs | 400s | Orta karmaşıklık |
| Kodlama-Uzmani | 300s | Hızlı kod üretimi |
| Apply-Model | 900s | Büyük dosya birleştirme |

---

## � Yaşanan Sorunlar ve Çözümler

### GLM-4.7 Sonsuz Düşünme Döngüsü (Infinite Thinking Loop)

**Sorun:** GLM-4.7 modeli thinking modunda sonsuz döngüye giriyordu. "Hmm, let me think..." gibi dolgu kelimeleri sürekli tekrar ediyordu.

**Araştırma Kaynakları:**
- [Reddit r/LocalLLaMA](https://reddit.com/r/LocalLLaMA) - GLM model deneyimleri
- [Unsloth GitHub Issues](https://github.com/unslothai/unsloth) - Fine-tuning sorunları
- [Z.ai Community](https://z.ai) - GLM-4 optimizasyon tartışmaları

**Topluluk Bulgular:**
| Kullanıcı | Öneri | Sonuç |
|-----------|-------|-------|
| u/AfterAte | `repetition_penalty=1.0` (KAPALI) | ✅ Loop durdu |
| u/PANIC_EXCEPTION | `temperature=0.2-0.3` | ✅ Stabil çıktı |
| Unsloth docs | `min_p=0.01` (default 0.05 loop yapıyor) | ✅ Çeşitlilik arttı |
| Z.ai forum | `top_k=20` + `top_p=1.0` | ✅ Deterministik ama flexible |

**Çözüm (vLLM Backend için):**
```yaml
kodlama_extra: &kodlama_extra
  repetition_penalty: 1.0   # KAPALI - #1 loop sebebi
  min_p: 0.01               # Düşük eşik
  top_k: 20                 # Stabil çıktı
  frequency_penalty: 0.0
  presence_penalty: 0.0
  top_p: 1.0
```

**Prompt Kuralları:**
```
THINKING KURALLARI (LOOP ONLEME):
- Ayni fikri TEKRAR ETME - bir kez soyle, sonuca gec
- Dolgu kelimeleri KULLANMA (Hmm, Let me think, Wait, I see)
- Cikmazda kalirsan DURDUR ve kullaniciya sor
```

---

### Kimi K2.5 Tool Token Sızıntısı (Tool Token Leakage)

**Sorun:** Kimi K2.5 modeli tool call yaparken `<|tool▁calls▁begin|>` gibi özel tokenler çıktıya sızıyordu.

**Araştırma Kaynakları:**
- [vLLM GitHub Issues](https://github.com/vllm-project/vllm/issues)
- [vLLM Pull Requests](https://github.com/vllm-project/vllm/pulls)

**Bulunan Çözüm:**
- **PR #28543** (Kasım 2025 - MERGED): `KimiK2ToolParser` state machine implementasyonu
- Token leakage'ı önleyen düzgün parsing eklendi

**Durum:** vLLM'i güncel versiyona yükseltmek gerekiyor. Şu an Kimi K2.5 yerine GLM-4.7 kullanılıyor.

---

### maxTokens vs contextLength Karmaşası

**Sorun:** `maxTokens` ve `contextLength` parametrelerinin farkı net değildi.

**Açıklama:**
| Parametre | Anlam | Örnek |
|-----------|-------|-------|
| `contextLength` | Modelin görebildiği TOPLAM token | 131K (GLM-4.7) |
| `maxTokens` | Modelin ÜRETEBİLECEĞİ maksimum token | 32K |

**Formül:** `contextLength = input_tokens + maxTokens`

**Seçim:** 32K maxTokens = ~3000 satır kod kapasitesi. Loop önleme ile büyük dosya dengesi.

---

### Apply Model Dokümantasyon Sorunu

**Sorun:** Apply-Model doküman dosyalarında (*.md) düzgün çalışmıyordu. Markdown syntax'ını bozuyordu.

**Çözüm:** Docs-Writer için Apply-Model kullanmak yerine **markdown code block** çıktısı:
```markdown
\`\`\`markdown
# Doküman İçeriği
Tam içerik buraya...
\`\`\`
```

Kullanıcı kopyalayıp dosyaya yapıştırıyor. Bu yöntem daha güvenilir.

---

### single_find_replace Güvenilirlik Sorunu

**Sorun:** `single_find_replace` tool'u bazen yanlış yere yazıyordu veya hiç çalışmıyordu.

**Çözüm:** `edit_existing_file` tercih edildi (TAM DOSYA yazımı). Kodlama-Uzmani prompt'undan `single_find_replace` kaldırıldı.

---

## 📋 Versiyon Geçmişi

### v4.3.0 (2026-02-04)
- `kodlama_params` kaldırıldı (kullanılmıyordu, kafa karıştırıyordu)
- **00-core.md** ANALIZ/SONUÇ/KOD formatına dokümantasyon muafiyeti eklendi
- README ve config.yaml model/timeout tutarlılığı sağlandı
- General-Engineer referansları kaldırıldı (mevcut değil)

### v4.2.0 (2026-02-03)
- **BREAKING:** GLM-4.7 loop prevention parametreleri
  - `repetition_penalty=1.0` (KAPALI)
  - `min_p=0.01`, `top_k=20`
- **THINKING KURALLARI** güncellendi
  - "3 adım limiti" → "tekrar yasağı" (model zekasını kısıtlamaz)
  - Dolgu kelimeleri listesi genişletildi
- **Docs-Writer** Qwen3-30B → GLM-4.7-FP8 geçişi
- **07-reasoning.md** loop önleme protokolü eklendi
- **06-documentation.md** markdown code block yöntemi

### v4.1.0 (2026-02-02)
- maxTokens 65K → 32K (loop önleme dengesi)
- single_find_replace Kodlama-Uzmani'dan kaldırıldı
- Git history temizlendi (orphan branch)

### v3.0.3 (2026-02-02)
- Temperature optimizasyonu (GLM-4.7 → 0.15)
- K2.5 büyük proje analizi eklendi
- Output format kuralları (ANALIZ-SONUÇ-KOD)
- Error recovery protokolü
- maxTokens 32K'ya optimize edildi

### v3.0.2 (2026-02-01)
- Tekrar yasağı (TEKRAR YASAGI) eklendi
- Vitis output format kuralları

### v3.0.1 (2026-02-01)
- Apply model silme operasyonu desteği
- [DELETE], [REMOVE], [SİL] markers

### v3.0.0 (2026-01-26)
- Tam yeniden yapılandırma
- Rules hiyerarşisi oluşturuldu
- Uzmanlık modelleri tanımlandı

---

## 🤝 Katkıda Bulunma

1. Fork edin
2. Feature branch oluşturun (`git checkout -b feature/amazing-feature`)
3. Değişikliklerinizi commit edin (`git commit -m 'Add amazing feature'`)
4. Branch'i push edin (`git push origin feature/amazing-feature`)
5. Pull Request açın

---

## 📄 Lisans

Bu proje MIT lisansı altında sunulmaktadır.

---

## 🙏 Teşekkürler

- [Continue](https://continue.dev) - VS Code AI asistan eklentisi
- Lokal LLM toplulukları

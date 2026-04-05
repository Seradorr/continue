# Continue Kapali Ag Home Paketi

Bu repo dogrudan `~/.continue/` icerigi olarak kopyalanmak uzere duzenlenmistir. Kopyalama sonrasi goreli yerlesim su sekilde olmalidir:

```text
~/.continue/
|- config.yaml
|- rules/
|  |- 00-core.md
|  |- 01-general.md
|  |- 02-fpga.md
|  |- 03-vitis.md
|  |- 04-csharp.md
|  |- 05-python.md
|  |- 06-documentation.md
|  |- 07-reasoning.md
|  |- 08-agent-workflow.md
|  |- 09-production-quality.md
|  '- 10-git.md
'- docs/
   |- continue-model-analysis.md
   '- archive/
```

## Hedef

Bu paket kapali ag veya intranet ortaminda Continue'u:

- guvenilir `chat` ve `edit`
- rule-driven domain uzmanligi
- acik ve bakimi kolay model routing
- minimum runtime yuzeyi

ile calistirmak icin hazirlandi.

## Model Seti

| Model | Backend | Roller | Not |
|---|---|---|---|
| `Hard-Engineer` | `Qwen3.5-397B-A17B-FP8` | `chat`, `edit` | Varsayilan kalite odakli ana model |
| `Soft-Engineer` | `Qwen3.5-35B-A3B-FP8` | `chat`, `edit` | Hizli ama hala tool-aware alternatif |
| `Vision-Engineer` | `Kimi-K2.5` | `chat`, `edit` | Gorsel ve uzun baglam gorevleri |
| `Kimi-Hard-Engineer` | `Kimi-K2.5` | `chat`, `edit` | Alternatif hard-engine rotasi |
| `GLM-Hard-Engineer` | `GLM-4.7-FP8` | `chat`, `edit` | Alternatif hard-engine rotasi |
| `Rerank-Model` | `bge-reranker-v2-m3` | `rerank` | Ileride retrieval acilirsa kullanilabilir |
| `Embedding-Model` | `Qwen3-Embedding-8B` | `embed` | Ileride indexing acilirsa kullanilabilir |

`autocomplete` ve `apply` bu pakette tanimli degildir.

## Kullanim

1. Bu repo icerigini `~/.continue/` altina kopyalayin.
2. `config.yaml` dosyasinin ust kisimdaki ortak anchor alaninda endpoint ve key degerlerini guncelleyin:

```yaml
api_config: &api_config
  provider: openai
  apiBase: https://api.sirket.com.tr/v1
  apiKey: key
```

3. Gerekirse sertifika yolunu guncelleyin:

```yaml
caBundlePath: C:\Sertifika\sirketadi-ca.crt
```

4. Continue'u yeniden yukleyin veya IDE pencerenizi reload edin.
5. Chat veya Agent icinde uygun aktif modeli secin.

## Runtime Yapisi

### `config.yaml`

- Resmi ve guncel format `config.yaml`
- OpenAI-compatible intranet endpoint icin `provider: openai` + `apiBase`
- Ust kisimdaki ortak anchor bloklari tum modeller tarafindan reuse edilir
- Sertifika, timeout ve retry ayarlari global `requestOptions` altinda tek yerde tutulur
- Model bazli `requestOptions` yalnizca gerekli farklar icin kullanilir
- Slash promptlar config icinde tanimlidir:
  - `/review`
  - `/root-cause`
  - `/handoff`

### `rules/`

Rule dosyalari runtime davranisini domain'e gore sekillendirir:

- `00-core.md`: cekirdek agent protokolu
- `01-general.md`: genel muhendislik ve verification
- `02-fpga.md`: FPGA / HDL / Vivado / XDC / Tcl
- `03-vitis.md`: Xilinx / Vitis / bare-metal C/C++
- `04-csharp.md`: C# / .NET
- `05-python.md`: Python
- `06-documentation.md`: dokumantasyon ve config duzenleme
- `07-reasoning.md`: kanit temelli akil yurutme
- `08-agent-workflow.md`: incele -> degistir -> dogrula akisi
- `09-production-quality.md`: production kalite ve guvenlik
- `10-git.md`: manuel git guvenligi

Kural stratejisi hibrittir: core kurallar kisa ve sert tutulur, domain kurallari ise karar verdiren checklist, anti-pattern, failure mode ve verification mantigi ile daha zengin tutulur.

## `embed`, `rerank` ve `docs:`

### `embed` ve `rerank` neden kaldi

Bu roller gelecekte retrieval veya indexing acildiginda faydalidir. Varsayilan chat/edit akisina kritik bagimlilik olarak konumlandirilmadilar, ama config'ten de cikartilmadilar.

### `docs:` nedir

`docs:` alani Continue'un dokumantasyon sitelerini crawl/index etmesi icindir.

Artisi:

- dis veya ic dokumanlardan retrieval yapabilir
- framework veya kurum ici wiki sorularinda yardimci olabilir

Eksisi:

- ek index/crawl maliyeti
- kapali agda erisim ve sertifika bakimi
- stale veya yanlis kaynak riski

Bu nedenle varsayilan runtime'da `docs:` bloklari acik degildir.

## Kapali Ag Tasarimi

Bu paket varsayilan olarak sunlari yapmaz:

- internet bagimli Mission Control referansi kullanmaz
- aktif `mcpServers` tanimlamaz
- `docs:` indexing acmaz
- `autocomplete` veya `apply` rolu acmaz

Boylece gorunen yuzey kucuk, davranis daha tahmin edilebilir ve bakim daha kolay kalir.

## Bakim Dosyalari

- `docs/continue-model-analysis.md`
  - Resmi Continue dokumanina dayali karar kaydi
  - Runtime'a otomatik enjekte edilmez
- `docs/archive/*`
  - Onceki refactor notlari ve tarihsel calisma belgeleri

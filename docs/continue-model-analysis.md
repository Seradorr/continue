# Continue Model Analysis

Bu dosya runtime kurali degildir. Bakim ve karar kaydi amaciyla tutulur.

## 1. Resmi Dayanaklar

- `config.yaml` guncel ve resmi formattir; `config.json` deprecated kabul edilir.
  - Kaynak: https://docs.continue.dev/reference
  - Kaynak: https://docs.continue.dev/reference/yaml-migration
- OpenAI-compatible self-hosted endpointler `provider: openai` ve `apiBase` ile tanimlanir.
  - Kaynak: https://docs.continue.dev/customize/model-providers/top-level/openai
- Model rolleri explicit tanimlanabilir; `roles` verilmezse varsayilan roller icinde `apply` da vardir.
  - Kaynak: https://docs.continue.dev/reference
- Agent mode icin `tool_use` capability gereklidir; custom deploymentlerde explicit yazmak daha guvenlidir.
  - Kaynak: https://docs.continue.dev/reference
  - Kaynak: https://docs.continue.dev/faqs
- Rules, Agent/Chat/Edit isteklerinin system message'ina concatenate edilir.
  - Kaynak: https://docs.continue.dev/reference
- Prompts, `prompts:` alanindan slash command olarak cagrilabilir.
  - Kaynak: https://docs.continue.dev/reference
  - Kaynak: https://docs.continue.dev/customize/deep-dives/prompts
- `docs:` alani dokumantasyon sitelerini crawl/index etmek icindir.
  - Kaynak: https://docs.continue.dev/reference
- `mcpServers:` opsiyoneldir; agent mode'a ek tool verir ama varsayilan kapali ag paketinde zorunlu degildir.
  - Kaynak: https://docs.continue.dev/reference
  - Kaynak: https://docs.continue.dev/reference/continue-mcp

## 2. Bu Paketteki Kararlar

### `config.yaml` korunuyor

Paket, kullanicinin endpoint ve API key'i dogrudan dosyanin ustunde girmeye devam ettigi yapida tutuldu. Bu sayede kapali agda `.env`, Mission Control veya harici onboarding adimi gerekmiyor.

### YAML `|` ve anchor mantigi

`|` YAML literal block scalar'dir. Cok satirli prompt ve system message metinlerini satir yapisini koruyarak tanimlamak icin dogru yontemdir. Continue'a ozel degil, YAML'in standart ozelligidir.

Bu pakette anchor kullanimi mevcut, calisan deployment duzenine gore korunmustur. `api_config`, parametre bloklari ve global `requestOptions` yapisi compatibility nedeniyle ayni tutuldu. Bu revizyonda amac, calisan kapali ag kurulumunu bozmadan mantiksal ve promptsal iyilestirme yapmaktir.

### Ana runtime sadece `chat` ve `edit`

`autocomplete` ve `apply` hic tanimlanmadi. Sebep:

- Kullanici bu rolleri istemiyor.
- Resmi dokumana gore `roles` verilmezse `apply` varsayilan olarak gelebilir; bu nedenle ana modellerde roller explicit yazildi.

### Neden `Hard-Engineer` varsayilan

Continue resmi model onerileri chat/edit icin coder-family modellere egilim gosterebilir, ancak bu paket bilincli olarak mevcut operasyonel tercihi korur:

- Varsayilan ana model `Qwen3.5-397B-A17B-FP8`
- Hizli alternatif `Qwen3.5-35B-A3B-FP8`
- Alternatif hard-engine rotalari `Kimi-K2.5` ve `GLM-4.7-FP8`

Bu tercih resmi benchmark iddiasi degil, kapali agdaki mevcut model kalitesi ve kullanici tercihi odakli bir routing karari olarak ele alinmali.

### Neden software-centric persona degil

`Production-grade genel yazilim muhendisi` dili, FPGA/RTL, firmware, linker, interrupt ve timing gibi alanlarda gereksiz yere dar bir cerceve sunuyordu. Bu nedenle persona domain-notr bir muhendislik asistani diline cekildi.

Amac:

- software-centric kokuyu azaltmak
- ayni model setiyle yazilim, firmware ve sayisal tasarim baglamlarini kapsamak
- uzmanligi prompt yerine aktif rule katmanindan almak

### Neden `tool_use` explicit

Self-hosted OpenAI-compatible deploymentlerde Continue capability autodetection bazen eksik veya tutarsiz olabilir. Bu nedenle agent kullanilacak modellerde `capabilities: [tool_use]` explicit yazildi. `Vision-Engineer` icin ek olarak `image_input` verildi.

### Neden `embed` ve `rerank` korunuyor

Continue resmi olarak embed ve rerank rollerini destekliyor. Bunlar indexing ve vector-search akislari icin anlamli olabilir.

Ancak resmi rehberlerde yeni codebase awareness yaklasimi su yone kaymis durumda:

- built-in file/search tools
- project rules
- gerekirse MCP veya custom RAG

Kaynak:

- https://docs.continue.dev/guides/codebase-documentation-awareness
- https://docs.continue.dev/reference

Bu nedenle `Embedding-Model` ve `Rerank-Model` config'te tutuldu, fakat varsayilan runtime'a `docs:` indexing veya aktif `mcpServers:` eklenmedi.

### Neden domain rules yeniden buyutuldu

Kalite hedefi prompt kisaltmak degil, daha dogru ve daha profesyonel yonlendirme saglamaktir. Bu nedenle:

- `alwaysApply` core rules kisa tutuldu
- domain rules yeniden zenginlestirildi
- tekrar eden buyuk tablo ve ornek kodlar geri getirilmedi
- bunun yerine anti-pattern, failure mode, verification ve entegrasyon etkileri daha acik yazildi

### Neden `docs:` varsayilan degil

`docs:` Continue'un dis veya ic dokumantasyon sitelerini crawl/index etmesini saglar.

Artisi:

- Dokumantasyon retrieval kalitesini artirabilir
- Framework veya ic wiki sorularinda yardimci olabilir

Eksisi:

- Ek crawl ve index maliyeti
- Kapali agda erisim ve sertifika bakimi
- Stale veya yanlis index riski

Kapali ag minimal runtime icin default disi birakildi.

## 3. Runtime ve Non-Runtime Dosyalari

Runtime tarafinda Continue'un aktif kullandigi dosyalar:

- `config.yaml`
- `rules/*.md`
- `config.yaml` icindeki `prompts:` bloklari

Runtime'a otomatik enjekte edilmeyen dosyalar:

- `docs/continue-model-analysis.md`
- `README.md`
- `docs/archive/*`

Bu dosyalar insan bakimi, karar kaydi ve tasarim notlari icindir.

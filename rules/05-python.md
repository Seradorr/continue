---
name: Python Rules
description: |
  Python uygulama, otomasyon ve kutuphane gelistirme icin typing, packaging ve test odakli kurallar.
globs:
  - "**/*.py"
  - "**/requirements.txt"
  - "**/pyproject.toml"
  - "**/setup.py"
  - "**/setup.cfg"
  - "**/Pipfile"
  - "**/poetry.lock"
  - "**/tox.ini"
  - "**/pytest.ini"
  - "**/.flake8"
alwaysApply: false
---

# PYTHON RULES

- Mevcut paket yapisi, import stili ve toolchain tercihini koru.
- Yeni veya degisen public API'larda type hint kullan; mevcut typing stiline uy.
- Mevcut kod zaten `pathlib` kullaniyorsa onu surdur; farkli stil varsa gereksiz buyuk donusum yapma.
- Import-time side effect ve global mutable state'ten kacin.
- Module import'u sirasinda IO, network veya config mutasyonu yapan davranislari gereksiz yere ekleme.
- `asyncio` yalnizca proje zaten async ise veya gerekliligi netse kullan.
- Async ve sync API'lari ayni modulde karistirirken cagrim sozlesmesini bozma.
- Veri modeli tarafinda mevcut stil neyse ona uy: `dataclass`, `pydantic`, `TypedDict` veya klasik class.
- Packaging, CLI entrypoint, config loading ve environment etkilerini birlikte dusun.
- Packaging, entrypoint ve environment davranisini etkileyen degisikliklerde `pyproject.toml` ve import yolunu birlikte dusun.
- Anti-pattern olarak su davranislardan kacin:
  - import-time calisan yan etkiler
  - gevsek exception yutma
  - typing ve gercek davranisin ayrismasi
  - packaging etkisini dusunmeden modul tasima
- Verification onerisi olarak en az ilgili `pytest` veya mevcut proje komutunu belirt.

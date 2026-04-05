---
name: Documentation And Config Rules
description: |
  Markdown, changelog ve config dosyalarini yapisini bozmadan guncellemek icin kurallar.
globs:
  - "**/*.md"
  - "**/README*"
  - "**/CHANGELOG*"
  - "**/docs/**/*"
  - "**/*.rst"
  - "**/*.txt"
  - "**/*.yaml"
  - "**/*.yml"
  - "**/*.json"
alwaysApply: false
---

# DOCUMENTATION AND CONFIG RULES

- Kullanici acikca istemedikce mevcut icerigi silme; hedefli guncelleme yap.
- Baslik hiyerarsisi, tablo, liste, kod blogu ve link yapisini koru.
- Changelog girdilerini mevcut siraya ve formata uygun ekle.
- Config dosyalarinda calisan anahtarlari keyfi yeniden adlandirma veya sirf estetik icin oynatma.
- Placeholder, eksik bolum veya sahte ornek birakma.
- Dokumantasyon guncellemesinde teknik iddia varsa config veya kod ile tutarli olmasini kontrol et.
- README, config ve analiz dokumanlari arasinda celiski olusturma.
- Yanlis sadeleştirme yapma: bilgi kaybi yaratan kisaltma kabul edilmez.

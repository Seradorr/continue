---
name: General Engineering Rules
description: |
  Tum diller ve proje tipleri icin genel muhendislik, debugging ve verification kurallari.
alwaysApply: true
---

# GENERAL ENGINEERING RULES

- Once belirtiyi ve beklenen davranisi netlestir; sonra kok neden ara.
- Hata analizi yaparken log, stack trace, diff, config ve caller etkisini birlikte dusun.
- Minimal ama yeterli degisiklik yap; yan etkili buyuk rewrite'tan kacin.
- Multi-file degisiklikte bagimlilik sirasini koru ve interface uyumunu kontrol et.
- Public API, paylasilan utility veya config degisiyorsa backward-compat etkisini dusun.
- Verification onerisi somut olmali: build, test, lint, sentez veya ilgili komut.
- Belirsiz isteklerde uydurma uzmanlik taslama; hangi bilginin eksik oldugunu yaz.

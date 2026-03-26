---
name: Git Workflow Rules
description: |
  Git durum analizi, commit hijyeni, branch guvenligi,
  conflict resolution ve geri alinmasi zor islemler icin kurallar.
alwaysApply: false
---

# GIT WORKFLOW RULES

Bu kurallar Git ve repository yonetimi odakli islerde uygulanir.

---

## 1. DURUM ANALIZI ONCE GELIR

- Aksiyon onermeden once mevcut durumu anlamaya calis
- Gerekirse status, diff, log ve branch/upstream bilgisini incele
- Belirsiz durumda varsayim yapma; mevcut repo durumuna gore konus

---

## 2. GUVENLIK KURALLARI

- `push --force`, `reset --hard`, `checkout --`, paylasilan branch rebase gibi islemlerde once riskleri belirt
- Geri alinmasi zor veya veri kaybi yaratabilecek adimlarda kullanicidan acik onay iste
- Secret, token, credential veya `.env` benzeri dosyalari fark edersen commit etme; riski acikla

---

## 3. COMMIT DISIPLINI

- Yalnizca gorevle ilgili degisiklikleri stage et
- Commit mesaji repo stiline ve degisikligin amacina uygun olsun
- Commit mesaji ne degistiginden cok neden yapildigini anlatsin
- Commit oncesi staged degisikliklerle mesajin uyumlu oldugunu kontrol et

---

## 4. CONFLICT VE HISTORY YONETIMI

- Conflict durumunda once catisan niyetleri acikla, sonra en guvenli cozumu oner
- Rebase, cherry-pick, revert ve bisect islemlerinde gecmis etkisini dusun
- Shared branch akisini bozabilecek history rewrite islemlerinde ekstra dikkatli ol

---

## 5. CIKTI BICIMI

- Git komut ciktilarini oldugu gibi yigma; onemli satirlari ozetle
- Durum, risk ve onerilen adimi net ayir
- Uygulama sonrasi branch durumu ve kalan risk varsa kisa belirt

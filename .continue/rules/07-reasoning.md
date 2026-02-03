---
name: Advanced Reasoning Protocol
description: |
  Gelismis dusunme, analiz ve karar verme protokolleri.
  GLM-4.7 ve Kimi K2.5 icin loop onleme kurallari dahil.
alwaysApply: true
---

# GELISMIS AKIL YURUTME PROTOKOLU

Bu kurallar daha kaliteli ve tutarli cikti icin uygulanir.

---

## 0. LOOP ONLEME (GLM-4.7 / Kimi K2.5 ICIN KRITIK)

> ⚠️ Reddit, GitHub ve Unsloth topluluk arastirmasindan elde edilen kurallar.

### Thinking (Dusunme) Limitleri

| KURAL | ACIKLAMA |
|-------|----------|
| Maksimum 3 adim | Dusunme surecinde 3 adimdan fazla KULLANMA |
| Tekrar YASAK | Ayni fikri ikinci kez yaziyorsan → HEMEN dur |
| Dolgu kelimesi YASAK | "Hmm", "Let me think", "Wait", "好的" KULLANMA |
| 500 token limiti | Dusunme 500 tokeni gecerse → CEVAP VER |

### Loop Tespit ve Mudahale

```
EGER ayni cumle/fikir 2. kez yaziliyorsa:
  → HEMEN dur
  → Mevcut bilgiyle CEVAP ver
  → Eksik kalsa bile devam ETME

EGER dusunme 500 tokeni gecerse:
  → "Yeterli analiz yapildi" de
  → CEVAP ver
```

### Sampling Parametreleri (Backend)

Bu kurallar config.yaml'da ayarlanmis olsa da, referans icin:
- `repetition_penalty: 1.0` (KAPALI - loop'un #1 sebebi)
- `min_p: 0.01` (dusuk esik = cesitlilik = loop onler)
- `top_k: 20` (makul kisitlama)
- `temperature: 0.2-0.3` (kod icin)

---

## 1. ISTEK ANALIZI (Her Istekte)

Her kullanici isteginde once su analizi yap:

```
1. NE ISTENIYOR?
   - Kod mu, dokumantasyon mu, analiz mi?
   - Yeni olusturma mi, degisiklik mi?
   - Tek dosya mi, coklu dosya mi?

2. NE ISTENMIYOR?
   - Testbench/test istenmis mi? (hayirsa ekleme)
   - Ornek istenmis mi? (hayirsa ekleme)
   - Dokumantasyon istenmis mi? (hayirsa ekleme)

3. BELIRSIZLIK VAR MI?
   - Parametreler net mi?
   - Hedef dosya/konum net mi?
   - Teknoloji/versiyon net mi?
```

---

## 2. KARAR AGACI

### Yeni Dosya mi Degisiklik mi?

```
Istek icinde mevcut dosya referansi var mi?
├─ EVET → Dosyayi OKU, TAM DOSYA olarak yeniden YAZ
└─ HAYIR → Yeni dosya OLUSTUR
```

### Testbench/Test Dahil mi?

```
Istekte "testbench", "tb", "test", "simulasyon" gecti mi?
├─ EVET → AYRI DOSYA olarak olustur
└─ HAYIR → EKLEME, sadece ana kodu ver
```

### Dokumantasyon mu Kod mu?

```
Hedef dosya .md, .rst, .txt mi?
├─ EVET → KORUMA modunda calis (silme yok)
└─ HAYIR → Normal kod modunda calis
```

---

## 3. CIKTI ONCESI KONTROL LISTESI

Cikti vermeden once su kontrolleri yap:

### Kod Ciktisi

- [ ] Dosya adi acikca belirtildi mi?
- [ ] Kod TAMAM mi (bastan sona)?
- [ ] Kisaltma (`...`, `existing code`) var mi? → YASAK
- [ ] Syntax hatalari var mi? → DUZELT
- [ ] Istenmeyen icerik (tb, test, ornek) eklendi mi? → CIKAR

### Dokumantasyon Ciktisi

- [ ] Mevcut icerik korundu mu?
- [ ] Yapi (baslik, liste) bozuldu mu? → DUZELT
- [ ] Eski girisler silindi mi? → GERI EKLE

### Degisiklik Ciktisi

- [ ] TAM DOSYA olarak mi yazildi?
- [ ] Degismeyen kisimlar korundu mu?
- [ ] Bosluklar/girintiler bozuldu mu? → DUZELT

---

## 4. KALITE STANDARTLARI

### Kod Kalitesi

| OLCUM | BEKLENTI |
|-------|----------|
| Syntax | Hatasiz, derlenebilir/calisabilir |
| Isimlendirme | Tutarli, anlasilir |
| Yapilandirma | Modular, okunabilir |
| Yorumlar | Sadece istenirse, anlasilir |

### Cikti Kalitesi

| OLCUM | BEKLENTI |
|-------|----------|
| Tamllik | Dosyanin TAMAMI mevcut |
| Dogruluk | Istenen degisiklik uygulanmis |
| Koruma | Degismeyen kisimlar aynen |
| Format | Girinti, bosluk korunmus |

---

## 5. HATA ONLEME

### Yaygin Hatalar ve Onleme

| HATA | ONLEME |
|------|--------|
| Kismi cikti | Her zaman TAM DOSYA yaz |
| Testbench ekleme | Acikca istenmemisse EKLEME |
| Icerik silme | Dokumantasyonda KORUMA modu |
| Varsayim | Belirsizlikte SORU SOR |
| Iteratif islem | Tek seferde TAMAMLA |

### Kendin Kontrol Et

```
CIKTI VERMEDEN ONCE:
1. "Bu cikti kullanicinin istegini tam karsiLiyor mu?"
2. "Istenmeyen bir sey ekledim mi?"
3. "Bir seyi silip kaybettim mi?"
4. "Kisaltma veya placeholder kullandim mi?"
```

---

## 6. EDGE CASE YONETIMI

### Bos Dosya

```
Dosya bos veya mevcut degil mi?
→ Yeni dosya olarak olustur
→ Uygun template/iskelet kullan
```

### Cok Buyuk Dosya

```
Dosya 1000+ satir mi?
→ Kullaniciya bilgi ver
→ Yine de TAM DOSYA olarak isle
→ Performans uyarisi ver (opsiyonel)
```

### Cakisan Istekler

```
Istekler birbiriyle celisiyor mu?
→ Varsayim YAPMA
→ Celiskiyi belirt ve soru sor
```

### Belirsiz Konum

```
Degisiklik nereye yapilacak belli degil mi?
→ Muhtemel konumlari listele
→ Kullaniciya sor
```

---

## 7. PERFORMANS OPTIMIZASYONU

### Context Verimli Kullanimi

- Gereksiz dosya okuma YAPMA
- Ilgili dosyalari ONCE tani
- Buyuk dosyalarda stratejik okuma

### Cikti Optimizasyonu

- Gereksiz aciklama EKLEME
- Onemli bilgiyi ONE CIKAR
- Kod ve aciklamayi AYIR

---

## 8. SUREKLI IYILESTIRME

### Her Istekten Ogrenme

```
1. Kullanici istegi net miydi?
   → Degilse, bir dahaki sefere soru sor

2. Cikti beklentiyi karsiladi mi?
   → Karsilamadiysa, neden analiz et

3. Hata veya duzeltme istendi mi?
   → Hatanin kokunu anla, tekrarlama
```

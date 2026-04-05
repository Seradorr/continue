---
name: Vitis Embedded C Rules
description: |
  Xilinx Vitis, bare-metal C/C++, BSP, DMA, interrupt ve platform entegrasyonu icin kurallar.
globs:
  - "**/*.c"
  - "**/*.h"
  - "**/*.cpp"
  - "**/*.hpp"
  - "**/src/**/*.c"
  - "**/src/**/*.h"
  - "**/src/**/*.cpp"
  - "**/src/**/*.hpp"
  - "**/bsp/**/*"
  - "**/platform/**/*"
  - "**/ps7_cortexa9_0/**/*"
  - "**/psu_cortexa53_0/**/*"
  - "**/psu_cortexr5_0/**/*"
  - "**/Makefile"
  - "**/CMakeLists.txt"
  - "**/lscript.ld"
alwaysApply: false
---

# VITIS EMBEDDED C RULES

- BSP tarafinda uretilen `xparameters.h` ve ilgili driver macro'larini birincil referans kabul et; magic address veya magic IRQ ID yazma.
- MMIO erisiminde `volatile`, register width, ordering ve ack sirasi kritik kabul edilir.
- MMIO yaz/oku sirasinin cihaz protokolunu etkiledigi durumlarda barrier veya driver semantigini bozma.
- ISR icinde minimum isi yap: status oku, gerekli ack/clear adimini uygula, kritik flag veya indeksleri guncelle, agir isi deferred path'e birak.
- ISR icinde heap, uzun loop, bloklayici bekleme ve buyuk kopyalardan kacin.
- ISR ile main loop veya worker context arasinda paylasilan state icin sahiplik ve gorunurluk kurallarini acik tut.
- Top-half/bottom-half ayrimi olan akislarin sorumluluk sinirlarini karistirma.
- Shared interrupt veya chained source yapilarinda source temizligi ile controller ack sirasini karistirma.
- DMA akisinda buffer ownership net olmali; CPU yazmadan sonra flush, DMA tamamlaninca CPU okumadan once invalidate uygula.
- DMA buffer'larinda alignment, cache line boyutu, OCM/DDR yerlesimi ve non-cacheable alan ihtiyacini dusun.
- Descriptor tabanli veya ring tabanli DMA kullaniliyorsa descriptor ownership, completion sirasi ve recycle akisi acik olmali.
- Head-tail ring buffer yapilarinda `head`, `tail`, bos/dolu kosulu ve wrap-around kurallari acik olmali; ISR ile main loop arasinda race olusturma.
- Producer/consumer yapilarinda indeks guncellemelerini atomik ve tek sahiplik mantigiyla tasarla.
- Ring yapilarinda su sorulari net cevapla:
  - bosluk ve doluluk nasil tanimlaniyor
  - head ne zaman ilerliyor
  - tail ne zaman ilerliyor
  - wrap-around sonrasi veri butunlugu nasil korunuyor
- Cache'li sistemlerde ring descriptor ve payload alanlarinin ayni memory policy altinda olup olmadigini kontrol et.
- Linker script, memory section, stack/heap boyutu ve interrupt vector degisiklikleri sistem seviye risk kabul edilir.
- `lscript.ld`, interrupt routing, DMA descriptor alani ve platform init akisi degisiyorsa etkileri acikca kontrol et.
- Platform init, clock setup, cache enable, MMU policy ve exception/interrupt init sirasini bozma.
- Anti-pattern olarak su davranislardan kacin:
  - ISR icinde polling veya uzun memcpy
  - cache flush/invalidate eksigi olan DMA akis
  - shared interrupt source'larda eksik source clear
  - linker script degisikliklerini map output incelemeden birakmak
  - ring buffer bos/dolu semantigini belirsiz birakmak
- Verification onerisi olarak goreve uygun sekilde sunlari belirt:
  - build sonucu ve warning kontrolu
  - map/linker output incelemesi
  - interrupt akisi ve ack sirasi dogrulamasi
  - DMA/cache davranisi ve gerekiyorsa ring descriptor akisinin gozden gecirilmesi

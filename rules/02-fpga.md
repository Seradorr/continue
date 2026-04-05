---
name: FPGA RTL Rules
description: |
  VHDL, Verilog, SystemVerilog, XDC ve Tcl icin HDL, CDC ve timing odakli kurallar.
globs:
  - "**/*.vhd"
  - "**/*.vhdl"
  - "**/*.v"
  - "**/*.sv"
  - "**/*.svh"
  - "**/*.xdc"
  - "**/*.tcl"
  - "**/hdl/**/*"
  - "**/rtl/**/*"
  - "**/ip/**/*"
  - "**/constraints/**/*"
alwaysApply: false
---

# FPGA RTL RULES

- RTL ve testbench ayri dosyalarda tutulur; testbench yalnizca istendiyse eklenir.
- Clock ve reset naming tutarli olmali; reset polarity tasarim boyunca acik olmalidir.
- Top-level saat ve reset kaynaklari sistem davranisini belirler; bu alanlarda degisiklik yaparken tum bagli domainleri dusun.
- CDC icin dogrudan sinyal gecisi yapma; single-bit icin synchronizer, veri akisi icin async FIFO veya handshake kullan.
- CDC kararinda sinyal tipi, pulse width, veri butunlugu ve geri basin ihtiyacini ayir.
- Reset domain crossing durumunda async assert ve sync deassert mantigini dusun.
- Birden fazla reset domain varsa reset release sirasi ve initialization bagimliliklarini kontrol et.
- FSM tasariminda default atamalar, deterministik gecisler ve acik state davranisi korunsun.
- Combinational loop, eksik default atama veya tool-dependent latch davranisi olusturma.
- FSM output ve next-state mantiginda sentez-sonrasi okunabilirlik ve timing etkisini birlikte dusun.
- Timing problemi olan veri yolunda pipeline, register balancing ve fan-out etkisini dusun.
- Uzun combinational yol, genis mux, kontrolsuz priority chain ve uygunsuz clock enable kullanimi timing riskidir.
- XDC degisikliklerinde create_clock, async group, IO timing ve false-path istisnalarini kanitla; gelisiguzel exception yazma.
- False-path, multicycle ve clock group istisnalari yalnizca acik mimari gerekce varsa eklenmeli; warning susturmak icin kullanma.
- CDC synchronizer register'lari, generated clock ve IO timing tanimlari bilincli ve izlenebilir olmali.
- AXI4-Lite ve AXI4-Stream tasariminda handshake kurallarini bozma; valid/ready ve reset davranisini standarda gore koru.
- AXI tarafinda backpressure, burst siniri, sideband sinyalleri ve reset sonrasi ilk transaction davranisini dusun.
- Vivado IP, top-level port, clocking ve reset routing degisikliklerinde sistem etkisini ayri dusun.
- Mixed-clock tasarimlarda clock crossing'i combinational decode ile birlestirme; domain sinirlari acik kalmali.
- Top-level entity, package, constraint ve IP integrator degisiklikleri birlikte dusunulmeden tamam sayilmaz.
- Anti-pattern olarak su davranislardan kacin:
  - dogrudan combinational CDC
  - belirsiz reset polarity veya ayni modulde karisik reset semantigi
  - warning susturmak icin rastgele timing exception
  - gercek generated clock yerine data-path'i saat gibi yorumlama
  - testbench mantigini RTL'e sizdirmak
- Verification onerisi olarak goreve uygun sekilde sunlari belirt:
  - synthesis sonucu ve kritik warning kontrolu
  - timing summary ve kritik path inceleme
  - gerekiyorsa CDC report veya DRC kontrolu
  - top-level ve constraint degisikliklerinde clock/reset baglantilarinin yeniden gozden gecirilmesi

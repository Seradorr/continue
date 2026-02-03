---
name: FPGA RTL Rules
description: |
  VHDL, Verilog, SystemVerilog ve Vivado icin 
  kod stili, anti-pattern ve testbench kurallari.
globs:
  - "**/*.vhd"
  - "**/*.vhdl"
  - "**/*.v"
  - "**/*.sv"
  - "**/*.xdc"
  - "**/*.tcl"
  - "**/hdl/**/*"
  - "**/rtl/**/*"
  - "**/ip/**/*"
alwaysApply: false
---

# FPGA RTL KURALLARI

Bu kurallar VHDL, Verilog, SystemVerilog ve constraint dosyalari icin gecerlidir.

---

## 1. TESTBENCH AYIRIMI (KRITIK)

> ⚠️ Bu kural en yuksek oncelikte uygulanir.

### Kural: Testbench SADECE Istenirse ve AYRI Dosyada

| DURUM | EYLEM |
|-------|-------|
| "debounce kodu yaz" | SADECE debounce.vhd |
| "testbench de ekle" | debounce.vhd + debounce_tb.vhd (AYRI) |
| "simulasyon kodu" | Testbench AYRI dosya |
| "nasil test edilir" | Aciklama ver, testbench YAZMA |

### Testbench Istemeyen Ifadeler

- "ornek kullanim"
- "nasil calisir"
- "aciklama ekle"
- Sadece modul/entity adi

### Testbench Isteyen Ifadeler

- "testbench yaz/ekle"
- "tb olustur"
- "simulasyon kodu"
- "test dosyasi"

### Dosya Isimlendirme

```
TASARIM:    module_name.vhd     veya  module_name.v
TESTBENCH:  module_name_tb.vhd  veya  module_name_tb.v
```

---

## 2. RTL SILME/DEGISTIRME KURALLARI

> ⚠️ Silme veya değiştirme işlemlerinde dikkatli ol.

### Silme Istegi

```
Kullanici "X'i sil" dediginde:

1. X'in NE oldugunu net tanimla (signal, process, port, yorum?)
2. X'i ciktidan CIKAR - "sildim" deyip birakma
3. Geri kalan kodu TAMAMEN yaz
4. Bagimlilikları kontrol et (X kullanan baska yer var mi?)
```

### RTL Silme Ornekleri

| ISTEK | EYLEM |
|-------|-------|
| "debug sinyallerini sil" | debug_* sinyalleri yok, diger kodlar tam |
| "bu yorumu kaldir" | Yorum yok, kod ayni |
| "test process'ini cikar" | Process yok, diger process'ler tam |
| "kullanilmayan portlari sil" | Portlar yok, entity/module guncel |

### Yorum Silme

```
"Yorumlari sil" veya "aciklamalari kaldir" istegi:
→ -- ile baslayan satirlari cikar (VHDL)
→ // ile baslayan satirlari cikar (Verilog)
→ Kod satirlari AYNEN kalir
```

---

## 3. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Entity/Module | snake_case | axi_lite_controller |
| Port | snake_case | data_valid, clk_100mhz |
| Signal | snake_case | state_reg, next_state |
| Constant/Generic | UPPER_SNAKE_CASE | DATA_WIDTH, CLK_FREQ |
| Type | snake_case_t | state_t, data_array_t |
| FSM State | ST_PREFIX | ST_IDLE, ST_PROCESS |

### Port Sirasi

1. Clock sinyalleri (`clk_*`)
2. Reset sinyalleri (`rst_*`, `rstn_*`)
3. Enable/control sinyalleri
4. Input data sinyalleri
5. Output data sinyalleri
6. Status/flag sinyalleri

### Format

- Girinti: 2 veya 4 space (tutarli)
- Tab KULLANMA
- Satir uzunlugu: max 120 karakter
- Her process/always basinda yorum

---

## 4. TASARIM KALIPLARI

### FSM - Two-Process (VHDL)

```vhdl
-- State register (sequential)
process(clk)
begin
    if rising_edge(clk) then
        if rst = '1' then
            state_reg <= ST_IDLE;
        else
            state_reg <= state_next;
        end if;
    end if;
end process;

-- Next state logic (combinational)
process(state_reg, inputs)
begin
    state_next <= state_reg;  -- Default: hold
    case state_reg is
        when ST_IDLE =>
            if start = '1' then
                state_next <= ST_PROCESS;
            end if;
        when others =>
            state_next <= ST_IDLE;
    end case;
end process;
```

### Synchronous Reset Tercihi

- Aktif-high (`rst`) veya aktif-low (`rstn`) - TUTARLI ol
- Asenkron reset sadece ozel durumlar (BRAM, DSP)

### Pipeline Register

```vhdl
process(clk)
begin
    if rising_edge(clk) then
        if ce = '1' then
            stage1 <= input_data;
            stage2 <= stage1;
            stage3 <= stage2;
        end if;
    end if;
end process;
```

---

## 5. ANTI-PATTERNS

### Latch Olusturan Kod

```vhdl
-- YANLIS: else yok = LATCH
process(sel, a, b)
begin
    if sel = '1' then
        output <= a;
    end if;
end process;

-- DOGRU
process(sel, a, b)
begin
    if sel = '1' then
        output <= a;
    else
        output <= b;
    end if;
end process;
```

### Eksik Sensitivity List

```vhdl
-- YANLIS
process(a)  -- b eksik!
begin
    output <= a and b;
end process;

-- DOGRU (VHDL-2008)
process(all)
begin
    output <= a and b;
end process;
```

### CDC Ihlali

```vhdl
-- YANLIS: Direkt baglanti
signal_clkB <= signal_clkA;

-- DOGRU: 2-stage synchronizer
process(clk_b)
begin
    if rising_edge(clk_b) then
        sync1 <= signal_clkA;
        sync2 <= sync1;
    end if;
end process;
signal_clkB_safe <= sync2;
```

### Blocking vs Non-Blocking (Verilog)

```verilog
// YANLIS: Sequential'da blocking
always @(posedge clk) begin
    a = b;   // YANLIS
    c = a;
end

// DOGRU: Non-blocking
always @(posedge clk) begin
    a <= b;
    c <= a;
end
```

---

## 6. VIVADO KURALLARI

### Constraint Dosyasi (.xdc)

- Clock tanimlari en ustte
- False path/multicycle path yorumla
- Pin atamalari gruplu sirala

### Sentez Uyarilari

| UYARI | ANLAM |
|-------|-------|
| Inferred latch | Latch olusmus - DUZELT |
| Multi-driven net | Ayni sinyale birden fazla atama |
| Timing not met | Timing constraint saglanmamis |
| DSP inference failed | DSP cikarilamamis |

---

## 7. TESTBENCH YAPISI (Sadece Istendiginde)

```vhdl
-- Clock generation
clk_process: process
begin
    clk <= '0'; wait for CLK_PERIOD/2;
    clk <= '1'; wait for CLK_PERIOD/2;
end process;

-- Reset generation
rst_process: process
begin
    rst <= '1';
    wait for CLK_PERIOD * 10;
    rst <= '0';
    wait;
end process;

-- Stimulus
stim_process: process
begin
    wait until rst = '0';
    wait for CLK_PERIOD * 2;
    -- Test cases
    report "Test completed" severity note;
    wait;
end process;
```

---

## 8. KRITIK DOSYALAR

Bu dosyalarda degisiklik yapmadan once **ONAY ISTE**:

- `*.xdc` - Constraint dosyalari
- Top-level entity/module
- Clock/reset modulleri
- AXI interface modulleri

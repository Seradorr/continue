---
name: Vitis Embedded C Rules
description: |
  Xilinx Vitis, bare-metal C/C++ ve BSP gelistirme icin
  kod stili, driver kullanimi ve anti-pattern kurallari.
globs:
  - "**/*.c"
  - "**/*.h"
  - "**/src/**/*.c"
  - "**/src/**/*.h"
  - "**/bsp/**/*"
  - "**/platform/**/*"
  - "**/ps7_cortexa9_0/**/*"
  - "**/psu_cortexa53_0/**/*"
alwaysApply: false
---

# VITIS EMBEDDED C KURALLARI

Bu kurallar Xilinx Vitis IDE, bare-metal C ve BSP gelistirme icin gecerlidir.

---

## 0. CIKTI FORMATI VE TEKRAR YASAGI (EN YUKSEK ONCELIK)

> ⚠️ Bu kurallar diger tum kurallardan ONCE uygulanir.

### Tekrar Yasagi

| YASAK | ORNEK |
|-------|-------|
| Ayni cumleyi tekrarlama | "X cakisiyor" → "X yine cakisiyor" → "X hala cakisiyor" |
| Donus yapma | Analiz → Sonuc → Tekrar Analiz (YASAK) |
| Farkli kelime ayni anlam | "overlap", "cakisma", "ust uste binme" art arda |

### Dogru Cikti Yapisi

```
1. TESPIT (tek paragraf):
   - Problem nedir?
   - Hangi satirlar/register'lar?
   - Tek cumlede ozet

2. SONUC (tek liste):
   - Ne yapilmali? (actionable)
   - Her madde FARKLI bir eylem

3. KOD (gerekirse):
   - Sadece degisen kisim
   - Tam dosya istenirse tam dosya
```

### Loop Tespit ve Durdurma

```
EGER son 2 paragrafta ayni konuyu islemissan:
  → DUR
  → Son paragrafi sil
  → "Ozet: [tek cumle]" yaz
  → Sonraki konuya gec veya bitir
```

---

## 1. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Fonksiyon | snake_case | init_dma, read_register |
| Degisken | snake_case | data_buffer, byte_count |
| Macro/Define | UPPER_SNAKE_CASE | MAX_BUFFER_SIZE |
| Struct | PascalCase veya _t | DmaConfig, dma_config_t |
| Global | g_ prefix | g_dma_instance |
| Static | s_ prefix | s_init_done |

### Header Guard

```c
#ifndef PROJECT_MODULE_H
#define PROJECT_MODULE_H

// Content

#endif /* PROJECT_MODULE_H */
```

### Fonksiyon Dokumantasyonu

```c
/**
 * @brief Kisa aciklama.
 * @param param1 Parametre aciklamasi.
 * @return Donus degeri aciklamasi.
 */
int function_name(int param1);
```

---

## 2. XILINX DRIVER PATTERN

### Initialization

```c
#include "xgpio.h"
#include "xparameters.h"

XGpio gpio_instance;

int init_gpio(void)
{
    XGpio_Config *config;
    
    config = XGpio_LookupConfig(XPAR_AXI_GPIO_0_DEVICE_ID);
    if (config == NULL) {
        return XST_FAILURE;
    }
    
    if (XGpio_CfgInitialize(&gpio_instance, config, 
                            config->BaseAddress) != XST_SUCCESS) {
        return XST_FAILURE;
    }
    
    XGpio_SetDataDirection(&gpio_instance, 1, 0x00);
    return XST_SUCCESS;
}
```

### Interrupt Setup

```c
#include "xscugic.h"
#include "xil_exception.h"

XScuGic gic_instance;

int setup_interrupts(void)
{
    XScuGic_Config *config;
    
    config = XScuGic_LookupConfig(XPAR_SCUGIC_SINGLE_DEVICE_ID);
    XScuGic_CfgInitialize(&gic_instance, config, config->CpuBaseAddress);
    
    Xil_ExceptionRegisterHandler(XIL_EXCEPTION_ID_INT,
        (Xil_ExceptionHandler)XScuGic_InterruptHandler, &gic_instance);
    Xil_ExceptionEnable();
    
    return XST_SUCCESS;
}
```

---

## 3. ANTI-PATTERNS

### ISR Icinde Uzun Islem

```c
// YANLIS
void my_isr(void *callback)
{
    process_large_buffer();  // YANLIS!
}

// DOGRU: Flag kullan
volatile int data_ready = 0;

void my_isr(void *callback)
{
    data_ready = 1;
}

void main_loop(void)
{
    if (data_ready) {
        data_ready = 0;
        process_large_buffer();
    }
}
```

### Hata Kontrolsuz Cagri

```c
// YANLIS
XGpio_Initialize(&gpio, DEVICE_ID);

// DOGRU
if (XGpio_Initialize(&gpio, DEVICE_ID) != XST_SUCCESS) {
    xil_printf("Init failed\n");
    return XST_FAILURE;
}
```

### Magic Number

```c
// YANLIS
Xil_Out32(0x43C00000, 0x1F);

// DOGRU
#define MY_IP_BASE  XPAR_MY_IP_0_BASEADDR
#define REG_CTRL    0x00
#define CTRL_EN     0x1F

Xil_Out32(MY_IP_BASE + REG_CTRL, CTRL_EN);
```

### Volatile Eksikligi

```c
// YANLIS
int flag = 0;
void isr(void *cb) { flag = 1; }
void main(void) { while (!flag); }  // Optimize edilebilir!

// DOGRU
volatile int flag = 0;
```

### Bare-metal malloc

```c
// YANLIS
char *buf = malloc(1024);

// DOGRU
static char buf[1024];
// veya
char buf[1024] __attribute__((aligned(32)));
```

---

## 4. CACHE ISLEMLERI

```c
// TX: CPU yazdi, DMA okuyacak
Xil_DCacheFlushRange((UINTPTR)tx_buf, len);

// RX: DMA yazacak, CPU okuyacak
Xil_DCacheInvalidateRange((UINTPTR)rx_buf, len);
```

---

## 5. STRUCT KULLANIMI

### Register Map

```c
typedef struct {
    volatile u32 control;   // 0x00
    volatile u32 status;    // 0x04
    volatile u32 data_in;   // 0x08
    volatile u32 data_out;  // 0x0C
} MyIpRegs;

#define MY_IP ((MyIpRegs *)XPAR_MY_IP_0_BASEADDR)

// Kullanim
MY_IP->control = 0x01;
```

---

## 6. KRITIK DOSYALAR

Bu dosyalarda degisiklik yapmadan once **ONAY ISTE**:

- `lscript.ld` - Linker script
- FSBL kaynak dosyalari
- ISR fonksiyonlari
- `Makefile`, `CMakeLists.txt`
- `xparameters.h` (otomatik uretilir!)

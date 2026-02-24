---
name: Vitis Embedded C Rules
description: |
  Xilinx Vitis, bare-metal C/C++ ve BSP gelistirme icin
  kod stili, driver kullanimi, DMA, interrupt, boot ve anti-pattern kurallari.
globs:
  - "**/*.c"
  - "**/*.h"
  - "**/src/**/*.c"
  - "**/src/**/*.h"
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

# VITIS EMBEDDED C KURALLARI

Bu kurallar Xilinx Vitis IDE, bare-metal C ve BSP gelistirme icin gecerlidir.

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
| ISR handler | _isr suffix | dma_rx_isr, gpio_isr |

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
 * @return XST_SUCCESS veya XST_FAILURE.
 */
int function_name(int param1);
```

---

## 2. XILINX DRIVER PATTERN

### Standard Initialization

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

int connect_interrupt(u32 intr_id, Xil_InterruptHandler handler, void *callback)
{
    int status;

    status = XScuGic_Connect(&gic_instance, intr_id, handler, callback);
    if (status != XST_SUCCESS) {
        return XST_FAILURE;
    }

    XScuGic_Enable(&gic_instance, intr_id);
    return XST_SUCCESS;
}
```

---

## 3. DMA TRANSFER PATTERN'LERI

### Simple DMA Transfer

```c
#include "xaxidma.h"

XAxiDma dma_instance;

int init_dma(void)
{
    XAxiDma_Config *config;

    config = XAxiDma_LookupConfig(XPAR_AXIDMA_0_DEVICE_ID);
    if (config == NULL) return XST_FAILURE;

    if (XAxiDma_CfgInitialize(&dma_instance, config) != XST_SUCCESS)
        return XST_FAILURE;

    XAxiDma_IntrDisable(&dma_instance, XAXIDMA_IRQ_ALL_MASK,
                        XAXIDMA_DEVICE_TO_DMA);
    XAxiDma_IntrDisable(&dma_instance, XAXIDMA_IRQ_ALL_MASK,
                        XAXIDMA_DMA_TO_DEVICE);

    return XST_SUCCESS;
}

int dma_transfer(u8 *tx_buf, u8 *rx_buf, u32 len)
{
    /* TX: CPU yazdi, DMA okuyacak → flush */
    Xil_DCacheFlushRange((UINTPTR)tx_buf, len);

    /* RX: DMA yazacak, CPU okuyacak → invalidate */
    Xil_DCacheInvalidateRange((UINTPTR)rx_buf, len);

    /* Transfer baslat */
    XAxiDma_SimpleTransfer(&dma_instance, (UINTPTR)rx_buf, len,
                           XAXIDMA_DEVICE_TO_DMA);
    XAxiDma_SimpleTransfer(&dma_instance, (UINTPTR)tx_buf, len,
                           XAXIDMA_DMA_TO_DEVICE);

    /* Tamamlanmayi bekle */
    while (XAxiDma_Busy(&dma_instance, XAXIDMA_DEVICE_TO_DMA));
    while (XAxiDma_Busy(&dma_instance, XAXIDMA_DMA_TO_DEVICE));

    return XST_SUCCESS;
}
```

### Double Buffering Pattern

```c
#define BUF_SIZE 4096

static u8 buf_a[BUF_SIZE] __attribute__((aligned(32)));
static u8 buf_b[BUF_SIZE] __attribute__((aligned(32)));
static volatile int active_buf = 0;  /* 0 = A, 1 = B */

void dma_rx_isr(void *callback)
{
    XAxiDma *dma = (XAxiDma *)callback;

    /* Interrupt temizle */
    u32 irq_status = XAxiDma_IntrGetIrq(dma, XAXIDMA_DEVICE_TO_DMA);
    XAxiDma_IntrAckIrq(dma, irq_status, XAXIDMA_DEVICE_TO_DMA);

    /* Buffer swap */
    active_buf ^= 1;

    /* Sonraki transfer'i hemen baslat */
    u8 *next_buf = (active_buf == 0) ? buf_a : buf_b;
    Xil_DCacheInvalidateRange((UINTPTR)next_buf, BUF_SIZE);
    XAxiDma_SimpleTransfer(dma, (UINTPTR)next_buf, BUF_SIZE,
                           XAXIDMA_DEVICE_TO_DMA);
}
```

### Scatter-Gather DMA

```c
/* SG DMA icin:
 * 1. BD (Buffer Descriptor) ring olustur
 * 2. Her BD'ye buffer adresi ve boyutu ata
 * 3. BD ring'i DMA'ya bagla
 * 4. Transfer baslat
 *
 * Avantaj: CPU mudahalesi olmadan coklu transfer
 * Kullanim: Video streaming, buyuk veri blokları
 */
```

---

## 4. PS-PL HABERLESME

### Port Tipleri

| PORT | BANT GENISLIGI | KULLANIM |
|------|----------------|----------|
| AXI GP (M_AXI_GP) | Dusuk (32-bit) | Register erisimi, kontrol |
| AXI HP (S_AXI_HP) | Yuksek (64-bit) | DMA, video, buyuk veri |
| AXI ACP | Yuksek + cache coherent | Cache tutarli veri paylasimi |
| EMIO | GPIO/SPI/I2C/UART | Dusuk hiz periferal |

### Interrupt Mapping

```c
/* PL interrupt'lari PS'e baglamak icin:
 * IRQ_F2P[15:0] → GIC SPI interrupt ID 61-68, 84-91
 *
 * Vivado'da: PL → AXI Interrupt Controller veya direkt IRQ_F2P
 * C'de: XScuGic_Connect ile GIC'e bagla
 */
#define PL_INTR_ID  61  /* IRQ_F2P[0] */
```

---

## 5. BOOT SEQUENCE

```
FSBL (First Stage Boot Loader)
  → PS initialization (MIO, DDR, clock)
  → PL bitstream yukleme
  → U-Boot veya Application yukleme

FSBL → U-Boot → Linux (Linux akisi)
FSBL → Application (Bare-metal akisi)
```

### Boot Hatalari

| HATA | OLASI NEDEN |
|------|-------------|
| FSBL basilamiyor | Boot mode pinleri yanlis |
| PL yuklenmiyor | Bitstream boyutu / CRC hatasi |
| App crash | Stack overflow, DDR init hatasi |
| Hang at main | Interrupt/exception yapilandirmasi eksik |

---

## 6. LINKER SCRIPT (lscript.ld)

### Section Yerlestirme

```
MEMORY {
    ps7_ddr_0 : ORIGIN = 0x00100000, LENGTH = 0x1FF00000
    ps7_ram_0 : ORIGIN = 0x00000000, LENGTH = 0x00030000
}

/* Heap ve Stack boyutlandirma */
_HEAP_SIZE  = 0x10000;   /* 64 KB - buyuk buffer'lar icin artir */
_STACK_SIZE = 0x8000;    /* 32 KB - derin recursion varsa artir */
```

### Kurallari

- DMA buffer'lari cache line aligned olmali (32 byte)
- Stack/heap boyutunu projeye gore ayarla
- OCM (On-Chip Memory) kritik ISR verileri icin kullan
- DDR adres araliklari FSBL ile uyumlu olmali

---

## 7. DEBUG STRATEJILERI

### JTAG Debug

- Breakpoint ve watchpoint kullan
- Register penceresi ile periferal durumu kontrol et
- Memory goruntuleme ile buffer iceriklerini incele

### UART Debug Printf

```c
#include "xil_printf.h"

/* Hafif print (printf yerine) */
xil_printf("DMA status: 0x%08x\r\n", XAxiDma_ReadReg(base, offset));

/* Register dump */
void dump_regs(u32 base, int count)
{
    for (int i = 0; i < count; i++) {
        xil_printf("REG[0x%02x] = 0x%08x\r\n",
                   i * 4, Xil_In32(base + i * 4));
    }
}
```

### Semihosting

- printf ciktisini JTAG uzerinden host PC'ye yonlendirir
- Debug build'de aktif, release'de KAPALI tutmali
- Performance etkisi yuksek — production'da KULLANMA

---

## 8. ANTI-PATTERNS

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
    xil_printf("GPIO init failed\r\n");
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
// YANLIS (derleyici optimize edebilir)
int flag = 0;
void isr(void *cb) { flag = 1; }
void main(void) { while (!flag); }

// DOGRU
volatile int flag = 0;
```

### Bare-metal malloc

```c
// YANLIS (heap fragmentation riski)
char *buf = malloc(1024);

// DOGRU (statik allocation)
static char buf[1024] __attribute__((aligned(32)));
```

---

## 9. CACHE ISLEMLERI

```c
/* TX: CPU yazdi, DMA okuyacak → FLUSH */
Xil_DCacheFlushRange((UINTPTR)tx_buf, len);

/* RX: DMA yazacak, CPU okuyacak → INVALIDATE */
Xil_DCacheInvalidateRange((UINTPTR)rx_buf, len);

/* KURAL: Buffer'lar cache line aligned olmali (32 byte) */
static u8 buf[SIZE] __attribute__((aligned(32)));
```

---

## 10. STRUCT KULLANIMI

### Register Map

```c
typedef struct {
    volatile u32 control;   /* 0x00 */
    volatile u32 status;    /* 0x04 */
    volatile u32 data_in;   /* 0x08 */
    volatile u32 data_out;  /* 0x0C */
} MyIpRegs;

#define MY_IP ((MyIpRegs *)XPAR_MY_IP_0_BASEADDR)

/* Kullanim */
MY_IP->control = 0x01;
u32 status = MY_IP->status;
```

---

## 11. MULTI-CORE (AMP)

### Asymmetric Multi-Processing

```
Zynq-7000: Cortex-A9 x2
Zynq UltraScale+: Cortex-A53 x4 + Cortex-R5 x2

AMP Pattern:
- CPU0: Ana uygulama (bare-metal veya Linux)
- CPU1: Gercek zamanli islem (bare-metal)
- Paylasilan bellek ile haberlesme (OCM veya DDR region)
- IPI (Inter-Processor Interrupt) ile senkronizasyon
```

### Paylasilan Bellek

```c
/* Ortak bellek adresi (linker script'te tanimli) */
#define SHARED_MEM_BASE  0xFFFF0000  /* OCM */

typedef struct {
    volatile u32 flag;
    volatile u32 data[256];
} SharedMem;

#define SHARED ((SharedMem *)SHARED_MEM_BASE)
```

---

## 12. KRITIK DOSYALAR

Bu dosyalarda degisiklik yapmadan once **ONAY ISTE**:

- `lscript.ld` — Linker script
- FSBL kaynak dosyalari
- ISR fonksiyonlari
- `Makefile`, `CMakeLists.txt`
- `xparameters.h` (otomatik uretilir — MANUAL DEGISTIRME!)

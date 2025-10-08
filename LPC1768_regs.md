# 📘 Registros del LPC1768 - Guía Completa

Resumen de los registros más importantes del microcontrolador **LPC1768 (ARM Cortex-M3)**.

---

## 🔹 System Control

### Power Control
| Registro | Dirección | Función |
|-----------|-----------|---------|
| **PCON** | 0x400FC0C0 | Control de energía |
| **PCONP** | 0x400FC0C4 | Control de energía de periféricos |

#### PCONP - Bits importantes:
- Bit 1: PCTIM0 (Timer 0)
- Bit 2: PCTIM1 (Timer 1)
- Bit 3: PCUART0 (UART0)
- Bit 4: PCUART1 (UART1)
- Bit 6: PCPWM1 (PWM1)
- Bit 12: PCADC (ADC)
- Bit 15: PCGPIO (GPIO)

### Clock Control
| Registro | Dirección | Función |
|-----------|-----------|---------|
| **CCLKCFG** | 0x400FC104 | Divisor del CPU clock |
| **PCLKSEL0** | 0x400FC1A8 | Divisor de clock para periféricos (0-15) |
| **PCLKSEL1** | 0x400FC1AC | Divisor de clock para periféricos (16-31) |

#### Valores de PCLKSEL:
- 00: PCLK = CCLK/4
- 01: PCLK = CCLK
- 10: PCLK = CCLK/2
- 11: PCLK = CCLK/8

---

## 🔹 GPIO (General Purpose I/O)

### Pin Function Select
| Registro | Dirección | Rango de Pines |
|-----------|-----------|----------------|
| **PINSEL0** | 0x4002C000 | P0[15:0] |
| **PINSEL1** | 0x4002C004 | P0[31:16] |
| **PINSEL2** | 0x4002C008 | P1[15:0] |
| **PINSEL3** | 0x4002C00C | P1[31:16] |
| **PINSEL4** | 0x4002C010 | P2[15:0] |

#### Valores PINSEL (2 bits por pin):
- 00: GPIO
- 01: Función alternativa 1
- 10: Función alternativa 2
- 11: Función alternativa 3

### Pin Mode (Pull-up/Pull-down)
| Registro | Dirección | Función |
|-----------|-----------|---------|
| **PINMODE0-9** | 0x4002C040-0x4002C064 | Modo de pull para cada pin |

#### Valores PINMODE:
- 00: Pull-up habilitado
- 01: Repetidor
- 10: Sin pull
- 11: Pull-down habilitado

### Fast GPIO (Puertos 0-4)
| Registro | Dirección Base | Función |
|-----------|----------------|---------|
| **FIO0DIR** | 0x2009C000 | Dirección Puerto 0 (1=salida, 0=entrada) |
| **FIO0PIN** | 0x2009C014 | Valor actual Puerto 0 |
| **FIO0SET** | 0x2009C018 | Set bits Puerto 0 |
| **FIO0CLR** | 0x2009C01C | Clear bits Puerto 0 |

#### Para otros puertos:
- Puerto 1: Base + 0x20 (0x2009C020)
- Puerto 2: Base + 0x40 (0x2009C040)
- Puerto 3: Base + 0x60 (0x2009C060)
- Puerto 4: Base + 0x80 (0x2009C080)

**Ejemplo de uso:**
```c
FIO0DIR |= (1 << 22);    // P0.22 como salida
FIO0SET |= (1 << 22);    // P0.22 = HIGH
FIO0CLR |= (1 << 22);    // P0.22 = LOW
```

---

## 🔹 Timers (Timer 0-3)

### Registros Base
| Timer | Dirección Base |
|-------|----------------|
| Timer 0 | 0x40004000 |
| Timer 1 | 0x40008000 |
| Timer 2 | 0x40090000 |
| Timer 3 | 0x40094000 |

### Registros de Control
| Registro | Offset | Función |
|-----------|--------|---------|
| **TCR** | +0x04 | Timer Control Register |
| **TC** | +0x08 | Timer Counter |
| **PR** | +0x0C | Prescale Register |
| **PC** | +0x10 | Prescale Counter |
| **MCR** | +0x14 | Match Control Register |
| **MR0-MR3** | +0x18-0x24 | Match Registers |
| **CCR** | +0x28 | Capture Control Register |
| **CR0-CR3** | +0x2C-0x38 | Capture Registers |
| **EMR** | +0x3C | External Match Register |

#### TCR - Bits importantes:
- Bit 0: Counter Enable (1=enable, 0=disable)
- Bit 1: Counter Reset (1=reset)

#### MCR - Match Control:
- Bits 0-2: MR0 (Interrupt, Reset, Stop)
- Bits 3-5: MR1 (Interrupt, Reset, Stop)
- Bits 6-8: MR2 (Interrupt, Reset, Stop)
- Bits 9-11: MR3 (Interrupt, Reset, Stop)

**Ejemplo Timer básico:**
```c
T0TCR = 0x02;      // Reset timer
T0PR = 24999;      // Prescaler para 1ms @ 25MHz
T0MR0 = 1000;      // Match en 1 segundo
T0MCR = 0x03;      // Interrupt y reset en MR0
T0TCR = 0x01;      // Start timer
```

---

## 🔹 UART (0-3)

### Registros Base
| UART | Dirección Base |
|------|----------------|
| UART0 | 0x4000C000 |
| UART1 | 0x40010000 |
| UART2 | 0x40098000 |
| UART3 | 0x4009C000 |

### Registros Principales
| Registro | Offset | Función |
|-----------|--------|---------|
| **RBR** | +0x00 | Receiver Buffer (solo lectura) |
| **THR** | +0x00 | Transmit Holding (solo escritura) |
| **DLL** | +0x00 | Divisor Latch LSB (cuando DLAB=1) |
| **DLM** | +0x04 | Divisor Latch MSB (cuando DLAB=1) |
| **IER** | +0x04 | Interrupt Enable Register |
| **IIR** | +0x08 | Interrupt ID Register |
| **FCR** | +0x08 | FIFO Control Register |
| **LCR** | +0x0C | Line Control Register |
| **LSR** | +0x14 | Line Status Register |

#### LCR - Line Control:
- Bits 0-1: Word Length (00=5bits, 01=6bits, 10=7bits, 11=8bits)
- Bit 2: Stop bits (0=1bit, 1=2bits)
- Bit 3: Parity Enable
- Bit 4: Parity Select (0=odd, 1=even)
- Bit 7: DLAB (Divisor Latch Access Bit)

#### LSR - Line Status:
- Bit 0: RDR (Receiver Data Ready)
- Bit 5: THRE (Transmitter Holding Register Empty)
- Bit 6: TEMT (Transmitter Empty)

**Configuración UART básica:**
```c
// UART0 @ 9600 baud, 25MHz PCLK
U0LCR = 0x83;        // 8N1, DLAB=1
U0DLL = 162;         // Baudrate = 9600
U0DLM = 0;
U0LCR = 0x03;        // DLAB=0
U0FCR = 0x07;        // Enable FIFO
```

---

## 🔹 PWM1

### Dirección Base: 0x40018000

### Registros de Control
| Registro | Offset | Función |
|-----------|--------|---------|
| **TCR** | +0x04 | Timer Control Register |
| **TC** | +0x08 | Timer Counter |
| **PR** | +0x0C | Prescale Register |
| **MR0** | +0x18 | Match Register 0 (Periodo) |
| **MR1-MR6** | +0x1C-0x30 | Match Registers 1-6 (Duty Cycle) |
| **LER** | +0x50 | Load Enable Register |
| **PCR** | +0x4C | PWM Control Register |

#### PCR - PWM Control:
- Bits 9-14: PWMENA1-PWMENA6 (Enable PWM outputs)
- Bit 2: PWMSEL2 (Single/Double edge)
- Bit 3: PWMSEL3
- Bit 4: PWMSEL4
- Bit 5: PWMSEL5
- Bit 6: PWMSEL6

#### LER - Load Enable:
- Bit 0: Load MR0
- Bits 1-6: Load MR1-MR6

**Ejemplo PWM:**
```c
PWM1TCR = 0x02;      // Reset PWM
PWM1PR = 0;          // Sin prescaler
PWM1MR0 = 1000;      // Periodo = 1000 ciclos
PWM1MR1 = 500;       // Duty = 50%
PWM1LER = 0x03;      // Load MR0 y MR1
PWM1PCR = 0x0200;    // Enable PWM1.1
PWM1TCR = 0x09;      // Enable PWM y Counter
```

---

## 🔹 ADC

### Dirección Base: 0x40034000

### Registros Principales
| Registro | Offset | Función |
|-----------|--------|---------|
| **ADCR** | +0x00 | Control Register |
| **ADGDR** | +0x04 | Global Data Register |
| **ADINTEN** | +0x0C | Interrupt Enable |
| **ADDR0-ADDR7** | +0x10-0x2C | Data Registers (canales 0-7) |
| **ADSTAT** | +0x30 | Status Register |

#### ADCR - Control Register:
- Bits 0-7: SEL (Selección de canal)
- Bits 8-15: CLKDIV (Divisor de clock)
- Bit 16: BURST (Modo burst)
- Bits 17-19: CLKS (Número de clocks)
- Bit 21: PDN (Power Down)
- Bits 24-26: START (Control de inicio)
- Bit 27: EDGE (Flanco para inicio externo)

#### Bits de resultado (ADGDR/ADDRx):
- Bits 4-15: RESULT (Resultado 12-bit)
- Bits 24-26: CHN (Canal que generó el resultado)
- Bit 30: OVERRUN (Sobrescritura)
- Bit 31: DONE (Conversión completa)

**Configuración ADC:**
```c
ADCR = 0x00200401;   // Canal 0, CLKDIV=4, PDN=1
ADCR |= 0x01000000;  // START=001 (inicio inmediato)
while(!(ADGDR & 0x80000000)); // Esperar DONE
result = (ADGDR >> 4) & 0xFFF;
```

---

## 🔹 Interrupciones (NVIC)

### Vector Interrupt Controller
| Registro | Dirección | Función |
|-----------|-----------|---------|
| **ISER0** | 0xE000E100 | Interrupt Set-Enable (0-31) |
| **ISER1** | 0xE000E104 | Interrupt Set-Enable (32-63) |
| **ICER0** | 0xE000E180 | Interrupt Clear-Enable (0-31) |
| **ICER1** | 0xE000E184 | Interrupt Clear-Enable (32-63) |
| **ISPR0** | 0xE000E200 | Interrupt Set-Pending (0-31) |
| **ICPR0** | 0xE000E280 | Interrupt Clear-Pending (0-31) |

### Interrupciones Principales del LPC1768
| IRQ# | Nombre | Descripción |
|------|--------|-------------|
| 1 | TIMER0_IRQn | Timer 0 |
| 2 | TIMER1_IRQn | Timer 1 |
| 3 | TIMER2_IRQn | Timer 2 |
| 4 | TIMER3_IRQn | Timer 3 |
| 5 | UART0_IRQn | UART0 |
| 6 | UART1_IRQn | UART1 |
| 7 | UART2_IRQn | UART2 |
| 8 | UART3_IRQn | UART3 |
| 9 | PWM1_IRQn | PWM1 |
| 22 | ADC_IRQn | ADC |

**Habilitar interrupción:**
```c
NVIC_EnableIRQ(TIMER0_IRQn);  // Habilitar Timer0
```

---

## 🔹 SysTick Timer

### Registros (Base: 0xE000E010)
| Registro | Offset | Función |
|-----------|--------|---------|
| **CTRL** | +0x00 | Control y Status |
| **LOAD** | +0x04 | Reload Value |
| **VAL** | +0x08 | Current Value |
| **CALIB** | +0x0C | Calibration |

#### CTRL bits:
- Bit 0: ENABLE (Habilitar contador)
- Bit 1: TICKINT (Habilitar interrupción)
- Bit 2: CLKSOURCE (1=CPU clock, 0=external)
- Bit 16: COUNTFLAG (Se puso en 0)

**SysTick a 1ms:**
```c
SysTick->LOAD = 24999;    // 1ms @ 25MHz
SysTick->CTRL = 0x07;     // Enable, Interrupt, CPU clock
```

---

## 🔹 Watchdog Timer

### Dirección Base: 0x40000000

| Registro | Offset | Función |
|-----------|--------|---------|
| **WDMOD** | +0x00 | Mode Register |
| **WDTC** | +0x04 | Timer Constant |
| **WDFEED** | +0x08 | Feed Sequence |
| **WDTV** | +0x0C | Timer Value |

#### WDMOD bits:
- Bit 0: WDEN (Watchdog enable)
- Bit 1: WDRESET (Reset enable)

**Feed del Watchdog:**
```c
WDFEED = 0xAA;
WDFEED = 0x55;
```

---

## 🔹 I2C (I2C0, I2C1, I2C2)

### Registros Base
| I2C | Dirección Base |
|-----|----------------|
| I2C0 | 0x4001C000 |
| I2C1 | 0x4005C000 |
| I2C2 | 0x400A0000 |

### Registros Principales
| Registro | Offset | Función |
|-----------|--------|---------|
| **CONSET** | +0x00 | Control Set |
| **STAT** | +0x04 | Status |
| **DAT** | +0x08 | Data |
| **ADR0** | +0x0C | Slave Address 0 |
| **SCLH** | +0x10 | SCL High Duty Cycle |
| **SCLL** | +0x14 | SCL Low Duty Cycle |
| **CONCLR** | +0x18 | Control Clear |

---

## 🔹 SPI (SSP0, SSP1)

### Registros Base
| SPI | Dirección Base |
|-----|----------------|
| SSP0 | 0x40088000 |
| SSP1 | 0x40030000 |

### Registros Principales
| Registro | Offset | Función |
|-----------|--------|---------|
| **CR0** | +0x00 | Control Register 0 |
| **CR1** | +0x04 | Control Register 1 |
| **DR** | +0x08 | Data Register |
| **SR** | +0x0C | Status Register |
| **CPSR** | +0x10 | Clock Prescale Register |

---

## 📖 Referencias y Ejemplos

### Configuración de Pins Común
```c
// UART0 en P0.2 (TXD) y P0.3 (RXD)
PINSEL0 |= (1 << 4) | (1 << 6);  // Función 01 para ambos pins

// PWM1.1 en P2.0
PINSEL4 |= (1 << 0);  // Función 01

// ADC0 en P0.23
PINSEL1 |= (1 << 14);  // Función 01
```

### Cálculo de Baudrate UART
**Fórmula:** `Baudrate = PCLK / (16 × (256 × DLM + DLL))`

Para 9600 baud con PCLK = 25MHz:
- DLL = 162, DLM = 0

### Cálculo de Frecuencia PWM
**Fórmula:** `Frecuencia = PCLK / MR0`

Para 1kHz con PCLK = 25MHz:
- MR0 = 25000

---

**Referencias:**
- UM10360 – LPC176x/5x User Manual (NXP Semiconductors)
- ARM Cortex-M3 Technical Reference Manual
- LPC1768 Datasheet


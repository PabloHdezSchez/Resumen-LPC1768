# 📘 Registros del LPC1768 - Guía Completa

Resumen de los registros más importantes del microcontrolador **LPC1768 (ARM Cortex-M3)**.

---

## 🔹 System Control

### Power Control

| Registro | Dirección | Función |
|-----------|-----------|---------|
| **PCON** | 0x400FC0C0 | Control de energía |
| **PCONP** | 0x400FC0C4 | Control de energía de periféricos |

#### PCONP - Bits importantes

- Bit 1: PCTIM0 (Timer 0)
- Bit 2: PCTIM1 (Timer 1)
- Bit 3: PCUART0 (UART0)
- Bit 4: PCUART1 (UART1)
- Bit 6: PCPWM1 (PWM1)
- Bit 12: PCADC (ADC)
- Bit 15: PCGPIO (GPIO)
- Bit 22: PCTIM2 (Timer 2)
- Bit 23: PCTIM3 (Timer 3)
- Bit 24: PCUART2 (UART2)
- Bit 25: PCUART3 (UART3)

### Clock Control

| Registro | Dirección | Función |
|-----------|-----------|---------|
| **CCLKCFG** | 0x400FC104 | Divisor del CPU clock |
| **PCLKSEL0** | 0x400FC1A8 | Divisor de clock para periféricos (0-15) |
| **PCLKSEL1** | 0x400FC1AC | Divisor de clock para periféricos (16-31) |

#### Valores de PCLKSEL

- 00: PCLK = CCLK/4
- 01: PCLK = CCLK
- 10: PCLK = CCLK/2
- 11: PCLK = CCLK/8

##### Algunos bits importantes

- Bit 1:0: PCLK_WDT (Peripheral clock selection for WDT.)
- Bit 3:2: PCLK_TIMER0
- Bit 5:4: PCLK_TIMER1
- Bit 7:6: PCLK_UART0
- Bit 9:8: PCLK_UART1

```c
// PCLK_TIMER0 = CCLK / 2
// PCLK_UART0  = CCLK

LPC_SC->PCLKSEL0 &= ~((0x3 << 2) | (0x3 << 6));  // Limpia Timer0 y UART0
LPC_SC->PCLKSEL0 |=  ((0x2 << 2) | (0x1 << 6));  // Timer0=CCLK/2, UART0=CCLK
```

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

#### Valores PINSEL (2 bits por pin)

- 00: GPIO
- 01: Función alternativa 1
- 10: Función alternativa 2
- 11: Función alternativa 3

```c
// P0.2=TXD0 (Func1), P0.3=RXD0 (Func1), P0.10=GPIO (00)
LPC_PINCON->PINSEL0 &= ~((3u<<(2*2)) | (3u<<(2*3)) | (3u<<(2*10)));  // limpia campos
LPC_PINCON->PINSEL0 |=  ((1u<<(2*2)) | (1u<<(2*3)));                // 01 = Función 1 (UART0)
// (opcional) usar P0.10 como salida GPIO:
LPC_GPIO0->FIODIR   |=  (1u<<10);

// NOTA: TXD0 y RXD0 son transmision y recepcion del UART0
```

### Pin Mode (Pull-up/Pull-down)

| Registro | Dirección | Función |
|-----------|-----------|---------|
| **PINMODE0-9** | 0x4002C040-0x4002C064 | Modo de pull para cada pin |

#### Valores PINMODE

- 00: Pull-up habilitado
- 01: Repetidor
- 10: Sin pull
- 11: Pull-down habilitado

### Fast GPIO (Puertos 0-4)

| Registro | Dirección Base | Función |
|-----------|----------------|---------|
| **FIO0DIR** | 0x2009C000 | Dirección Puerto 0 (1=salida, 0=entrada) |
| **FIO0MASK** | 0x2009C010 | Es la máscara del Puerto 0 para las operaciones con FIO0PIN |
| **FIO0PIN** | 0x2009C014 | Valor actual Puerto 0 |
| **FIO0SET** | 0x2009C018 | Set bits Puerto 0 |
| **FIO0CLR** | 0x2009C01C | Clear bits Puerto 0 |

#### Para otros puertos

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

### Intro

Los Timer 0,1,2 y 3 son periféricos configurados usando los siguientes registros:

1. Power: Mediante el registro PCONP (Al hacer reset, los registros 0 y 1 están activados. Los otros NO)
2. Peripheral Clock: Hay que ajustar el valor de PCLKSEL0 para Timers 0-1 y PCLKSEL1 para Timers 2-3
3. Pins: Seleccionar los pines de timer mediante el registro PINSEL y PINMODE
4. Interrupt: Ver los registros T0/1/2/3MCR y T0/1/2/3CCR para match y capture. Las interrupciones son habilitadas en el NVIC usando el registro de interrupcion adecuado.

```c
NVIC_EnableIRQ(TIMER0_IRQn);  // habilita interrupción de Timer0
```

5. DMA: Hasta dos de esos eventos de “match” pueden configurarse para disparar una solicitud DMA (Direct Memory Access).

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
| **IR** | +0x00 | Interrupt Register - Muestra qué evento produjo la interrupción (Se limpia escribiendo 1) |
| **TCR** | +0x04 | Timer Control Register - Arranque y reset del contador |
| **TC** | +0x08 | Timer Counter - Contador principal de 32 bits |
| **PR** | +0x0C | Prescale Register - Valor del divisor de frecuencia |
| **PC** | +0x10 | Prescale Counter - Contador del prescaler, se reinicia al alcanzar PR |
| **MCR** | +0x14 | Match Control Register - Define acciones al coincidir TC=MRx |
| **MR0-MR3** | +0x18-0x24 | Match Registers - Valores de comparación |
| **CCR** | +0x28 | Capture Control Register - Configuración del flancos de captura y habilitación de interrupciones |
| **CR0-CR3** | +0x2C-0x38 | Capture Registers - Almacenan el valor de TC al capturar |
| **EMR** | +0x3C | External Match Register - Controla las salidas MATx.y (Toggle, set, clear) |
| **CTCR** | +0x70 | Count Control Register - Selecciona modo Timer o Counter y fuente externa |
| **PWMC** | +0x74 | PWM Control - Habilita salida PWM simple en coincidencias MRx |

#### TCR - Bits importantes

- Bit 0: Counter Enable (1=enable, 0=disable)
- Bit 1: Counter Reset (1=reset)

#### MCR - Match Control

- Bits 0-2: MR0 (Interrupt, Reset, Stop)
- Bits 3-5: MR1 (Interrupt, Reset, Stop)
- Bits 6-8: MR2 (Interrupt, Reset, Stop)
- Bits 9-11: MR3 (Interrupt, Reset, Stop)

#### CCR - Capture Control Register

| Bits | Función                                  |
| ---- | ---------------------------------------- |
| 0    | Captura en flanco ascendente en CAPn.0   |
| 1    | Captura en flanco descendente en CAPn.0  |
| 2    | Habilita interrupción por captura CAPn.0 |
| 3–5  | Igual para CAPn.1                        |
| 6–8  | Igual para CAPn.2                        |
| 9–11 | Igual para CAPn.3                        |

#### EMR - External Match Register

Permite que las salidas MATn.0-MATn.3 cambien de estado automáticamente al producirse una coincidencia.

| Bits | Campo     | Descripción                                                            |
| ---- | --------- | ---------------------------------------------------------------------- |
| 0–3  | EM0–EM3   | Estado actual de las salidas externas (lectura/escritura)              |
| 4–11 | EMC0–EMC3 | 00 = Sin acción, 01 = Clear, 10 = Set, 11 = Toggle en coincidencia MRx |

#### CTCR - Count Control Register

| Bits | Campo            | Descripción                                                                                                                          |
| ---- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 1:0  | **Mode**         | 00 = Timer (usa PCLK)<br>01 = Counter por flanco ascendente<br>10 = Counter por flanco descendente<br>11 = Counter por ambos flancos |
| 3:2  | **Input Select** | Selecciona cuál entrada CAPn.x se usa como fuente de conteo externo                                                                  |

#### PWMC - PWM Control Register

Permite generar PWM básico con el propio temporizador, sin usar el periférico PWM1 completo.

| Bits | Campo  | Descripción                  |
| ---- | ------ | ---------------------------- |
| 0–3  | PWMENx | 1 = habilita modo PWM en MRx |

**Ejemplo Timer básico:**

```c
// Supone PCLK_TIMER0 = 25 MHz
LPC_SC->PCONP |= (1 << 1);        // Alimenta TIMER0
LPC_SC->PCLKSEL0 &= ~(3 << 2);    // PCLK_TIMER0 = CCLK/4 (25 MHz si CPU=100 MHz)

LPC_TIM0->TCR = 0x02;             // Reset timer
LPC_TIM0->PR  = 24999;            // Prescaler -> incrementa TC cada 1 ms (25 MHz / 25,000)
LPC_TIM0->MR0 = 1000;             // Match cada 1 segundo
LPC_TIM0->MCR = (1<<0) | (1<<1);  // MR0 genera interrupción y reset
LPC_TIM0->TCR = 0x01;             // Inicia el temporizador

NVIC_EnableIRQ(TIMER0_IRQn);      // Habilita IRQ en el NVIC

void TIMER0_IRQHandler(void) {
  LPC_TIM0->IR = 1;               // Limpia bandera MR0
  LPC_GPIO0->FIOPIN ^= (1<<10);   // Ejemplo: conmutar LED en P0.10
}
```

### Formula

- Formula base:
`f = PCLK / ((PR + 1) * (MR + 1))`

---

## 🔹 UART (0-3)

### Intro

Las UART0, UART2 y UART3 del LPC1768 son interfaces de comunicación serie asíncrona con FIFO de 16 bytes en TX y RX, generador de baudios programable y funcionamiento por interrupciones. La UART1 es muy parecida pero añade señales de módem y soporte RS-485.

Para usar cualquier UART hay que seguir siempre esta secuencia:

1. Power: Mediante el registro PCONP. Ajustar los bits PCUART0/2/3.
2. Peripheral Clock.
3. Baud rate. En los registros U0/2/3LCR. Poner el bit DLAB = 1. Esto habilita el acceso a los registros DLL y DLM para ajustar el baudrate. De ser necesario, ajusta el baudrate fracional en el registro divisor fraccional.
4. UART FIFO.
5. PINS. Seleciona los pines UART a traves de los registros PINSEL y PINMODE. **Importante no poner resistencias pull-down**
6. Interrupciones. Habilitar las interrupciones UART poner el bit DLAB = 0 en el registro U0/2/3LCR. Esto habilita el acceso a U0/2/3IER. No olvidar habilitar las interrupciones con NVIC.
7. DMA: UART0/2/3 las operaciones de transmitir y recibir pueden operar con el controlador GPMA.


### Registros Base

| UART | Dirección Base |
|------|----------------|
| UART0 | 0x4000C000 |
| UART1 | 0x40010000 |
| UART2 | 0x40098000 |
| UART3 | 0x4009C000 |

### Registros Principales

| Registro            | Offset                        | Función                                                                                                                            |
| ------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **RBR / THR / DLL** | +0x00                         | RBR: dato recibido (solo lectura). THR: dato a transmitir (solo escritura). DLL: byte bajo del divisor de baudios (cuando DLAB=1). |
| **DLM / IER**       | +0x04                         | DLM: byte alto del divisor (DLAB=1). IER: habilitación de interrupciones (DLAB=0).                                                 |
| **IIR / FCR**       | +0x08                         | IIR: identifica la causa de la interrupción. FCR: habilita y configura FIFO.                                                       |
| **LCR**             | +0x0C                         | Line Control: formato de la trama y DLAB.                                                                                          |
| **LSR**             | +0x14                         | Line Status: estado de RX/TX y errores.                                                                                            |
| **FDR**             | +0x28 aprox.                  | Fractional Divider Register: mejora la precisión del baud rate.                                                                    |
| **TER**             | (UART0/2/3 no siempre se usa) | Transmit Enable.                                                                                                                   |

#### LCR - Line Control

- Bits 0-1: Word Length (00=5bits, 01=6bits, 10=7bits, 11=8bits)
- Bit 2: Stop bits (0=1bit, 1=2bits)
- Bit 3: Parity Enable
- Bits 5:4 – Parity Select: 00=par, 01=impar, 10=forzado ‘1’, 11=forzado ‘0’.
- Bit 6 – Break Control: fuerza TX a ‘0’.
- Bit 7 – DLAB: 1 = acceso a DLL/DLM, 0 = acceso normal a THR/RBR/IER.

Esto es clave: para poner el baud rate pones DLAB=1, y para enviar/recibir lo bajas otra vez a 0.

#### LSR - Line Status

- Bit 0 (RDR): hay dato en RX (FIFO no vacía).
- Bit 1 (OE): overrun (llegó otro byte y no habías leído el anterior).
- Bit 2 (PE): error de paridad.
- Bit 3 (FE): framing error (bit de stop incorrecto).
- Bit 4 (BI): break.
- Bit 5 (THRE): THR vacío → puedes escribir otro byte.
- Bit 6 (TEMT): transmisor completamente vacío (THR y shift).
- Bit 7: indica que ha ocurrido algún error de línea.

Estos bits son los que miras en la ISR cuando IIR te dice “RLS”.

#### FCR - FIFO Control

- Bit 0 – FIFO Enable: 1 = activa FIFO RX/TX (siempre ponerlo).
- Bit 1 – RX FIFO Reset: 1 = vacía FIFO RX.
- Bit 2 – TX FIFO Reset: 1 = vacía FIFO TX.
- Bit 3 – DMA mode: 1 = peticiones DMA.
- Bits 7:6 – RX Trigger Level: 00=1 byte, 01=4, 10=8, 11=14 → cuándo quiero interrupción de RX.

Esto permite, por ejemplo, que no te interrumpa por cada byte, sino cuando haya 4 u 8.

Uso típico:
```c
LPC_UART0->FCR = (1<<0) | (1<<1) | (1<<2);   // FIFO on + reset RX + reset TX
// (luego puedes volver a escribir sin los reset si quieres configurar el trigger)
```

#### IER / IIR – Interrupciones

##### IER (Interrupt Enable Register), con DLAB=0:

- Bit 0 – RDA: dato recibido disponible.
- Bit 1 – THRE: transmisor vacío.
- Bit 2 – RLS: error de línea.
- Bits 8 y 9 – ABEO / ABTO (auto baud).

##### IIR (Interrupt Identification Register) te dice qué ha pasado:

- 0x04 → RDA (hay dato en RX)
- 0x02 → THRE (puedes mandar más)
- 0x06 → RLS (error de línea)
- 0x0C → CTI (Character Timeout). Para saberlo, se suele hacer switch(U0IIR & 0x0E).

No olvidar habilitar también en NVIC la IRQ concreta: UART0_IRQn, UART1_IRQn, UART2_IRQn, UART3_IRQn.

### Generador de Bauidios
1. La UART recibe un reloj: PCLK_UARTn
2. Ese reloj se divide primero por 16 internamente.
3. Luego se aplica el divisor de 16 bits(DLL:DLM)
4. Opcionalmente se afina con el divisor fraccional FDR (DIVADDVAL, MULVAL);

La fórmula completa es la siguiente:

`Baud = PCLK / (16 * DLL_DLM * (1 + DIVADDVAL/MULVAL))`

Siendo:
- DLL_DLM = 256 * DLM + DLL
- DIVADDVAL = FDR[3:0]
- MULVAL = FDR[7:4]

Reglas del FDR:
- 1 <= MULVAL <= 15
- 0 <= DIVADDVAL <= 14

### Ejemplo

```c
// Supongamos: PCLK_UART0 = 25 MHz y quiero 9600 baudios
// 1) Formato + DLAB=1
LPC_UART0->LCR = 0x83;      // 8 bits, sin paridad, 1 stop, DLAB=1

// 2) Divisor entero (sin fraccional)
LPC_UART0->DLM = 0;
LPC_UART0->DLL = 162;       // 25 MHz / (16 * 162) ≈ 9603 baudios

// 3) Opcional: FDR por defecto --> mejor no tocar
LPC_UART0->FDR = (1<<4) | 0;  // MULVAL=1, DIVADDVAL=0

// 4) Volver a modo normal (acceso a THR/RBR/IER)
LPC_UART0->LCR = 0x03;      // 8N1, DLAB=0

// 5) Activar FIFO
LPC_UART0->FCR = 0x07;      // FIFO en RX/TX y reset

// 6) (Opcional) Interrupciones
LPC_UART0->IER = (1<<0);    // interrupción por dato recibido
NVIC_EnableIRQ(UART0_IRQn);
```

---

## 🔹 PWM1

### Intro

El PWM se configura usando los siguientes registros

1. Power: En el registro PCONP, poner el bit PCPWM1 (En reset ya está a 1)
2. Peripheral Clock: En el registro PCLKSEL0, seleccionar PCLK_PWM1.
3. PINS: Seleccionar los pines PWM mediante los registros PINSEL. Seleccionar los pin modes con las funciones PWM1 mediante PINMODE.
4. Interrupts: Ver los registros PWM1MCR y PWM1CCR. Habilitar las interrupciones NVIC.

```c
NVIC_EnableIRQ(PWM1_IRQn);
```

### Dirección Base

| Periférico | Dirección base |
| ---------- | -------------- |
| **PWM1**   | `0x4001 8000`  |

### Registros de Control

| Registro | Offset | Descripción |
| ----------- | ---------- | ------------------------------------------------------------------------------------------- |
| **IR**      | +0x00      | Interrupt Register – bandera de interrupciones por match/capture (se limpia escribiendo 1). |
| **TCR**     | +0x04      | Timer Control Register – arranque, reset y modo PWM.                                        |
| **TC**      | +0x08      | Timer Counter – contador principal.                                                         |
| **PR**      | +0x0C      | Prescale Register – divisor del reloj base.                                                 |
| **MCR**     | +0x14      | Match Control Register – acciones al coincidir MRx (interrumpir, resetear, parar).          |
| **MR0–MR6** | +0x18–0x30 | Match Registers – MR0 define el periodo, MR1–MR6 el duty cycle.                             |
| **CCR**     | +0x28      | Capture Control Register – captura de TC por flancos de entrada CAPn.x.                     |
| **LER**     | +0x50      | Load Enable Register – actualiza MRx en el siguiente ciclo.                                 |
| **PCR**     | +0x4C      | PWM Control Register – activa salidas PWM y modo single/double edge.                        |
| **CTCR**    | +0x70      | Count Control Register – modo Timer o Counter.                                              |
| **PWMC**    | +0x74      | PWM Mode Control – activa modo PWM general. *(En LPC1768 ya se gestiona desde TCR)*         |

### Detalles Registros

#### TCR - Timer Control Register

| Bit | Nombre                  | Función                                        |
| :-: | :---------------------- | :--------------------------------------------- |
|  0  | **Counter Enable (CE)** | 1 = habilita contador                          |
|  1  | **Counter Reset (CR)**  | 1 = resetea TC y PC                            |
|  3  | **PWM Enable (PWMEN)**  | 1 = habilita modo PWM (TC activo + PWM activo) |

En modo PWM, se suele escribir TCR = 0x09 (bits 3 y 0 activos).

#### MCR - Match Control Register

|  Bits | Función            |
| :---: | :----------------- |
| [0–2] | MR0I / MR0R / MR0S |
| [3–5] | MR1I / MR1R / MR1S |
|  ...  | ...                |

#### PCR - PWM Control

|  Bit | Nombre              | Descripción                      |
| :--: | :------------------ | :------------------------------- |
| 9–14 | **PWMENA1–PWMENA6** | 1 = activa cada salida PWM1.x    |
|  2–6 | **PWMSELx**         | 0 = single edge, 1 = double edge |

#### LER - Load Enable

Permite actualizar MRx de forma segura

| Bit | Significado   |
| --- | ------------- |
| 0   | Carga MR0     |
| 1–6 | Carga MR1–MR6 |

Hay que escribir 1 en el bit correspondiente después de modificar MRx, para que el nuevo valor se aplique al final del ciclo PWM actual.

#### IR - Interrupt Register

| Bit | Evento               |
| --- | -------------------- |
| 0–6 | MR0–MR6 coincidencia |
| 8   | Captura 0            |
| 9   | Captura 1            |

Se limpia escribiendo un 1 en el bit correspondiente.

#### CCR - Capture Control Register

Usado si se emplea la función de captura (CAP1.0, CAP1.1):

| Bit | Descripción                |
| --- | -------------------------- |
| 0   | Captura flanco ascendente  |
| 1   | Captura flanco descendente |
| 2   | Interrupción por captura   |
| 3–5 | Igual para CAP1.1          |

**Ejemplo PWM:**

```c
#include "LPC17xx.h"

void PWM1_init(void) {
  LPC_SC->PCONP |= (1 << 6);               // Power PWM1
  LPC_SC->PCLKSEL0 &= ~(3 << 12);          // PCLK_PWM1 = CCLK/4
  LPC_PINCON->PINSEL4 |= (1 << 0);         // P2.0 -> PWM1.1
  LPC_PINCON->PINMODE4 &= ~(3 << 0);       // Pull-up por defecto

  LPC_PWM1->PR  = 0;                       // Sin prescaler
  LPC_PWM1->MR0 = 1000;                    // Periodo = 1000 ticks
  LPC_PWM1->MR1 = 500;                     // 50% duty
  LPC_PWM1->MCR = (1 << 1);                // Reset TC on MR0
  LPC_PWM1->LER = (1 << 0) | (1 << 1);     // Cargar MR0 y MR1
  LPC_PWM1->PCR = (1 << 9);                // Enable PWM1.1
  LPC_PWM1->TCR = (1 << 3) | (1 << 0);     // Enable PWM + counter
}

void PWM1_IRQHandler(void) {
  if (LPC_PWM1->IR & (1 << 0)) {           // MR0 interrupt?
    LPC_PWM1->IR = (1 << 0);               // Limpiar bandera MR0
    // Aquí tu código periódico, p. ej. ajustar duty dinámicamente
  }
}

int main(void) {
  PWM1_init();
  NVIC_EnableIRQ(PWM1_IRQn);
  while(1);
}
```

**Ejemplo Timer Servomotor**
Produce un tiempo en alto entre 0.3 y 2.1 (1.8 + 0.3).

```c
#define fpCLK 25e6
#define Tpwm 15e-3

void config_pwm2(void){
  LPC_PINCON->PINSEL3 |= (2<<8);
  LPC_SC->PCOMP |= (1<<6);
  LPC_PWM1->MR0 = Fpclk*Tpwm-1;
  LPC_PWM1->PCR |= (1<<10);
  LPC_PWM1->MCR |= (1<<1);
  LPC_PWM1->TCR |= (1<<0) | (1<<3);
}

void set_servo(float grados){
  LPC_PWM1->MR2=(Fpclk*0.3e-3 + Fpclk*1.8e-3*grados/180);
  LPC_PWM1->LER |= (1 << 2) | (1 << 0);
}
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

#### ADCR - Control Register

- Bits 0-7: SEL (Selección de canal)
- Bits 8-15: CLKDIV (Divisor de clock)
- Bit 16: BURST (Modo burst)
- Bits 17-19: CLKS (Número de clocks)
- Bit 21: PDN (Power Down)
- Bits 24-26: START (Control de inicio)
- Bit 27: EDGE (Flanco para inicio externo)

#### Bits de resultado (ADGDR/ADDRx)

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

#### CTRL bits

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

#### WDMOD bits

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

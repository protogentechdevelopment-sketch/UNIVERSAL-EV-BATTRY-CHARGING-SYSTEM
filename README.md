# ⚡UNIVERSAL-EV-BATTRY-CHARGING-SYSTEM

> A high-frequency isolated DC–DC power conversion system for universal EV battery charging, integrating a MOSFET full-bridge inverter, step-up transformer, STM32-based PWM control, and multi-stage rectification and filtering.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Working Principle](#working-principle)
- [PWM & Switching Calculations](#pwm--switching-calculations)
- [Results](#results)
- [Limitations & Future Scope](#limitations--future-scope)

---

## Overview

The rapid growth of electric vehicles, renewable energy systems, and portable electronics has significantly increased demand for efficient, compact, and high-power-density battery charging systems. Conventional charging systems suffer from low efficiency, bulky size, poor voltage adaptability, and high switching losses.

This project presents the design and implementation of a **Multi-Stage High-Frequency Isolated DC–DC Power Conversion System** that addresses these limitations. The system converts a regulated low-voltage DC input into a higher regulated DC output using:

- **MOSFET Full-Bridge Inverter** for DC-to-AC conversion
- **Step-Up Transformer** for voltage amplification and galvanic isolation
- **Bridge Rectifier + Capacitor Filter** for clean DC output
- **STM32F103C6T6 Microcontroller** for precision PWM-based switching control

The system demonstrates high efficiency, compact form factor, and reliable performance — making it suitable for EV charging, renewable energy storage, and embedded power supply applications.

---

## Features

- ✅ High-frequency switching at ~30 kHz for compact passive component sizing
- ✅ MOSFET full-bridge inverter (H-bridge) with IR2110 gate driver
- ✅ Step-up transformer with 1:3 turns ratio (8T primary : 24T secondary)
- ✅ STM32F103C6T6 bare-metal PWM control — no HAL overhead
- ✅ LM2596 buck converter for stable 12V → 5V regulated supply
- ✅ KBJ6J bridge rectifier for full-wave rectification
- ✅ Electrolytic capacitor filter for ripple reduction
- ✅ 16×2 LCD display for real-time voltage and status output
- ✅ USB Type-C breakout board for DC power delivery output
- ✅ Saleae Logic Analyser used for PWM signal verification
- ✅ Output voltage ~21–25V DC from 5V DC input

---

## System Architecture

```
+------------------+     +--------------------+     +---------------------+
|  12V DC Supply   | --> |  LM2596 Buck Conv  | --> |  5V Regulated Rail  |
+------------------+     +--------------------+     +----------+----------+
                                                                |
                                                    +-----------v-----------+
                                                    |  STM32F103C6T6 MCU   |
                                                    |  (TIM1 PWM @ 30 kHz) |
                                                    +-----------+-----------+
                                                                |
                                                    +-----------v-----------+
                                                    |  IR2110 Gate Driver   |
                                                    |  (High + Low Side)    |
                                                    +-----------+-----------+
                                                                |
                                          +---------------------v---------------------+
                                          |       IRFZ44N Full-Bridge Inverter        |
                                          |   Q1+Q4 / Q2+Q3 alternating switching    |
                                          +---------------------+---------------------+
                                                                |
                                                    +-----------v-----------+
                                                    |   Step-Up Transformer  |
                                                    |   8T Primary : 24T Sec |
                                                    |   Turns Ratio = 1:3    |
                                                    +-----------+-----------+
                                                                |
                                                    +-----------v-----------+
                                                    |  KBJ6J Bridge Rectif  |
                                                    |  Full-Wave AC → DC    |
                                                    +-----------+-----------+
                                                                |
                                                    +-----------v-----------+
                                                    | Electrolytic Capacitor |
                                                    | Filter (Ripple Reduce) |
                                                    +-----------+-----------+
                                                                |
                                                    +-----------v-----------+
                                                    |  ~21–25V DC Output    |
                                                    |  USB Type-C / Load    |
                                                    +-----------------------+
```

### Block Diagram Summary

```
DC Input → Buck Converter → MCU (PWM) → Gate Driver → Full-Bridge Inverter
        → Step-Up Transformer → Bridge Rectifier → Filter → DC Output
                                     ↑
                              LCD Display (Status + Voltage)
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Main Microcontroller | STM32F103C6T6 | — | ARM Cortex-M3, PWM generation & control |
| Gate Driver IC | IR2110 | GPIO (PWM) | High/Low-side MOSFET driver with bootstrap |
| Power MOSFETs (×4) | IRFZ44N | Gate (PWM) | Full-bridge H-bridge switching |
| Buck Converter | LM2596 | — | 12V → 5V regulated supply |
| Step-Up Transformer | Ferrite Core, 8T:24T | — | Voltage step-up + isolation (ratio 1:3) |
| Bridge Rectifier | KBJ6J Series | — | Full-wave AC → DC rectification |
| Filter Capacitor | Electrolytic 1000µF | — | Ripple reduction, smooth DC output |
| LCD Display | 16×2 HD44780 | GPIO (4-bit) | Voltage and system status display |
| USB Type-C Board | Breakout Board | VBUS/GND | DC power delivery output |
| Logic Analyser | Saleae Logic 2.4.43 | USB | PWM signal verification & debugging |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **STM32CubeIDE** | Firmware development, bare-metal register programming, debugging |
| **STM32CubeProgrammer** | Flashing compiled binary via ST-Link (SWD interface) |
| **Saleae Logic 2.4.43** | PWM waveform capture, frequency and duty cycle verification |

### Libraries & Programming Approach

| Item | Detail |
|---|---|
| **Programming Approach** | Bare-metal (direct register access — no HAL/CMSIS abstraction) |
| **Device Header** | `stm32f1xx.h` |
| **Timer Used** | TIM1 Advanced Timer (4-channel PWM) |
| **GPIO Pins** | PA8, PA9, PA10, PA11 (Alternate Function Push-Pull) |

---

## Working Principle

### 1. Input Stage — Buck Conversion
The 12V DC input is regulated to **5V DC** using the LM2596 switching buck converter. It operates on PWM-based inductor energy storage, maintaining stable 5V output regardless of load variation.

```
Vout = D × Vin
```

### 2. PWM Generation — STM32 TIM1
The STM32 generates 4-channel high-frequency PWM signals via the TIM1 advanced timer, configured in bare-metal mode for maximum precision.

### 3. Gate Driver — IR2110
The IR2110 amplifies the 3.3V MCU PWM signals to the 10–15V gate drive voltage required for MOSFET switching. It uses a **bootstrap capacitor** to drive the high-side MOSFET independently.

### 4. Full-Bridge MOSFET Inverter
Four IRFZ44N MOSFETs are arranged in an H-bridge. Diagonal pairs alternate to generate a **bipolar square wave AC output** at 30 kHz:

| Cycle | Active MOSFETs | Transformer Effect |
|---|---|---|
| Cycle 1 | Q1 + Q4 ON | Positive voltage across primary |
| Cycle 2 | Q2 + Q3 ON | Negative voltage across primary |

```
Vfundamental = 4 × Vdc / π
```

### 5. Step-Up Transformer
The ferrite-core transformer with **8 primary turns and 24 secondary turns** steps up the high-frequency AC voltage by a factor of 3.

```
Vs / Vp = Ns / Np = 24 / 8 = 3
```

### 6. Rectification & Filtering
The KBJ6J bridge rectifier converts AC to pulsating DC. The filter capacitor smooths the output:

```
Vavg = 2 × Vmax / π        (Rectifier average output)
Vr   = I / (f × C)          (Ripple voltage formula)
```

### 7. Output Voltage Control
Output voltage is controlled by adjusting the PWM duty cycle:

```
Vout = D × (Ns / Np) × Vin
```

---

## PWM & Switching Calculations

### Timer Configuration (TIM1)

| Parameter | Value |
|---|---|
| Clock Frequency | 8 MHz |
| Prescaler (PSC) | 0 |
| Auto-Reload Register (ARR) | 265 |
| Capture Compare Register (CCR) | 133–144 |

### Switching Frequency

```
f = fclk / (ARR + 1)
f = 8,000,000 / 266 ≈ 30,075 Hz  (~30 kHz)
```

### Duty Cycle

```
Duty = (CCR / (ARR + 1)) × 100
Duty = (133 / 266) × 100 ≈ 50–54%
```

---

## Program (Core PWM Logic)

```c
#include "stm32f1xx.h"

#define ARR_VAL  265
#define DUTY     133   // ~50%

void PWM_Init(void) {
    // Enable clocks
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN | RCC_APB2ENR_AFIOEN | RCC_APB2ENR_TIM1EN;

    // PA8–PA11: Alternate Function Push-Pull, 50 MHz
    GPIOA->CRH &= ~(0xFFFF);
    GPIOA->CRH |= 0xBBBB;

    // Timer base
    TIM1->PSC = 0;
    TIM1->ARR = ARR_VAL;

    // PWM Mode 1 on all 4 channels
    TIM1->CCMR1 |= (6 << 4) | (1 << 3) | (6 << 12) | (1 << 11);
    TIM1->CCMR2 |= (6 << 4) | (1 << 3) | (6 << 12) | (1 << 11);

    // Enable outputs
    TIM1->CCER |= (1<<0) | (1<<4) | (1<<8) | (1<<12);
    TIM1->BDTR |= (1 << 15);   // Main Output Enable (required for TIM1)
    TIM1->EGR  |= TIM_EGR_UG;
    TIM1->CR1  |= TIM_CR1_CEN;
}

int main(void) {
    PWM_Init();
    while (1) {
        // Cycle 1: Q1 + Q4 ON
        TIM1->CCR1 = DUTY; TIM1->CCR4 = DUTY;
        TIM1->CCR2 = 0;    TIM1->CCR3 = 0;
        delay_ms(10);

        // Dead time
        TIM1->CCR1 = 0; TIM1->CCR4 = 0;
        delay_us(5);

        // Cycle 2: Q2 + Q3 ON
        TIM1->CCR2 = DUTY; TIM1->CCR3 = DUTY;
        TIM1->CCR1 = 0;    TIM1->CCR4 = 0;
        delay_ms(10);

        // Dead time
        TIM1->CCR2 = 0; TIM1->CCR3 = 0;
        delay_us(5);
    }
}
```

---

## Results

| Parameter | Value |
|---|---|
| Input Voltage | 12V DC |
| Regulated Control Supply | 5V DC (LM2596) |
| Switching Frequency | ~30 kHz |
| PWM Duty Cycle | ~50–54% |
| Transformer Turns Ratio | 1:3 (8T : 24T) |
| Final DC Output Voltage | ~21–25V DC |
| Output Ripple | Low (high-frequency operation) |
| Shoot-Through Events | None observed |
| System Status | Stable and verified on hardware |

### Key Observations

- STM32 generated stable 30 kHz PWM with correct duty cycle — verified on Saleae Logic Analyser
- IR2110 gate driver provided clean switching with no shoot-through
- Full-bridge MOSFET inverter produced a consistent bipolar square wave across transformer primary
- Step-up transformer achieved expected 3× voltage gain
- KBJ6J rectifier performed full-wave rectification efficiently
- Filter capacitor reduced output ripple to acceptable levels
- LCD displayed live output voltage and system status during operation

---

## Limitations & Future Scope

### Current Limitations

- System operates in **open-loop mode** — output voltage not auto-regulated under load changes
- KBJ6J rectifier introduces diode conduction losses (designed for 50/60 Hz, used at 30 kHz)
- Switching noise and EMI require careful PCB layout
- Heating in MOSFETs and rectifier components under high load

### Future Enhancements

- **Closed-Loop Control** — ADC feedback from output voltage to auto-adjust PWM duty cycle
- **Sinusoidal PWM (SPWM)** — reduce harmonic distortion in the inverter output
- **Fast Recovery / Schottky Diodes** — reduce rectifier switching losses at high frequency
- **GaN / SiC Devices** — enable MHz-range switching, further reducing passive component size
- **Improved Transformer Design** — optimized ferrite core and winding for lower losses
- **Protection Circuits** — overvoltage, overcurrent, and thermal protection
- **Renewable Energy Integration** — adapt for solar MPPT and battery management systems

---

> *Designed and implemented as a final year B.E. project to contribute toward compact, efficient, and intelligent battery charging solutions for modern electric vehicles and embedded power systems.*

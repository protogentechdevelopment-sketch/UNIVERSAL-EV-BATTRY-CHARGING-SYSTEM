# 🔋 UNIVERSAL EV BATTERY CHARGING SYSTEM

> A high-frequency isolated DC–DC power conversion system for universal EV battery charging using STM32F103C6T6, IRFZ44N MOSFETs, IR2110 Gate Driver, Step-Up Transformer, and Bridge Rectifier.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Communication Protocols](#communication-protocols)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Conventional battery charging systems suffer from high switching losses, bulky size, poor voltage adaptability, and limited efficiency — making them unsuitable for modern EV and renewable energy applications. This project addresses those limitations with a **compact, high-frequency isolated DC–DC power conversion system** built on embedded hardware and power electronics.

The system converts a 12V DC input into a regulated ~21V DC output through a multi-stage pipeline: an LM2596 buck converter regulates the control supply, an STM32F103C6T6 microcontroller generates high-frequency PWM signals at ~30 kHz, an IR2110 gate driver switches four IRFZ44N MOSFETs in a full-bridge configuration, a 1:3 step-up transformer raises and isolates the voltage, and a KBJ6J bridge rectifier with electrolytic filter capacitors produces smooth DC output — all **without complex resonant control or wide-bandgap devices**.

---

## Features

- ✅ High-frequency PWM switching at ~30 kHz via STM32 TIM1 advanced timer
- ✅ Full-bridge MOSFET inverter using IRFZ44N in H-bridge configuration
- ✅ IR2110 gate driver with bootstrap circuit for reliable high-side switching
- ✅ Step-up transformer (1:3 turns ratio, 8T primary / 24T secondary, ferrite core)
- ✅ KBJ6J bridge rectifier with electrolytic capacitor filtering
- ✅ LM2596 buck converter for regulated 5V control supply from 12V input
- ✅ 16×2 LCD display for real-time output voltage and system status
- ✅ Bare-metal STM32 programming for deterministic timing and minimal overhead
- ✅ Shoot-through prevention via dead-time insertion in switching logic
- ✅ Output voltage regulation via duty cycle control (~54% → ~21V DC output)

---

## System Architecture

The system is divided into six conversion and control layers:

```
┌─────────────────────────────────────────────────────────────────┐
│  INPUT LAYER         12V DC Input → LM2596 Buck Converter (5V)  │
├─────────────────────────────────────────────────────────────────┤
│  CONTROL LAYER       STM32F103C6T6 (ARM Cortex-M3)              │
│                      TIM1 Advanced Timer — PWM @ 30 kHz         │
├─────────────────────────────────────────────────────────────────┤
│  SWITCHING LAYER     IR2110 Gate Driver + IRFZ44N MOSFETs       │
│                      Full-Bridge (H-Bridge) Inverter Topology   │
├─────────────────────────────────────────────────────────────────┤
│  CONVERSION LAYER    Step-Up Ferrite Core Transformer (1:3)     │
│                      Square Wave AC → Stepped-Up AC             │
├─────────────────────────────────────────────────────────────────┤
│  RECTIFICATION LAYER KBJ6J Bridge Rectifier                     │
│                      Full-Wave AC → Pulsating DC                │
├─────────────────────────────────────────────────────────────────┤
│  OUTPUT LAYER        Electrolytic Capacitor Filter (1000µF)     │
│                      Smooth ~21V DC → Load / Battery Charger    │
└─────────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
12V DC Input
      │
      ▼
LM2596 Buck Converter (5V regulated)
      │
      ▼
STM32F103C6T6 ──► 16×2 LCD Display (System Status / Output Voltage)
      │
      ▼
IR2110 Gate Driver (Bootstrap High-Side Drive)
      │
      ▼
IRFZ44N Full-Bridge MOSFET Inverter (30 kHz Square Wave AC)
      │
      ▼
Step-Up Transformer (8T:24T — 1:3 ratio, Ferrite Core)
      │
      ▼
KBJ6J Bridge Rectifier (Full-Wave Rectification)
      │
      ▼
Electrolytic Capacitor Filter (1000µF Ripple Reduction)
      │
      ▼
~21V DC Regulated Output → Battery Charger / Load
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Microcontroller | STM32F103C6T6 | — | Central control unit, PWM generation |
| Power MOSFETs | IRFZ44N (×4) | Gate via IR2110 | Full-bridge DC-to-AC inverter switching |
| Gate Driver | IR2110 | HIN / LIN inputs | High-side & low-side MOSFET drive |
| Transformer | Ferrite Core (8T:24T) | AC coupling | Voltage step-up (1:3) and isolation |
| Bridge Rectifier | KBJ6J Series | AC input / DC output | Full-wave AC-to-DC rectification |
| Filter Capacitor | Electrolytic 1000µF | DC bus | Ripple reduction and output smoothing |
| Buck Converter | LM2596 Module | DC-DC | 12V → 5V regulated control supply |
| LCD Display | 16×2 (HD44780) | Parallel GPIO | Real-time system status and voltage display |
| Logic Analyser | Saleae (software) | USB | PWM waveform verification and debugging |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **STM32CubeIDE** | Bare-metal firmware development, compilation, and debugging |
| **STM32CubeProgrammer** | Flashing compiled firmware to MCU via ST-Link (SWD interface) |
| **Saleae Logic 2.4.43** | PWM signal capture, frequency/duty cycle verification, H-bridge analysis |

---

## Circuit & Pin Connections

### STM32 PWM Output Pins → IR2110 Gate Driver Inputs

| STM32 Pin | TIM1 Channel | Role |
|---|---|---|
| PA8 | CH1 | HIN1 — High-side Q1 |
| PA9 | CH2 | LIN2 — Low-side Q2 |
| PA10 | CH3 | HIN2 — High-side Q3 |
| PA11 | CH4 | LIN1 — Low-side Q4 |

### IR2110 Gate Driver → IRFZ44N MOSFETs (Full Bridge)

| IR2110 Pin | Connected To | Description |
|---|---|---|
| HIN | STM32 PA8 / PA10 | High-side PWM input |
| LIN | STM32 PA9 / PA11 | Low-side PWM input |
| HO | Q1 / Q3 Gate | High-side MOSFET gate drive |
| LO | Q2 / Q4 Gate | Low-side MOSFET gate drive |
| VB | Bootstrap capacitor (+) | Floating high-side supply |
| VS | MOSFET switch node | Bootstrap reference |
| VCC | 12V | Driver logic supply |
| VSS | GND | Driver ground |

### Step-Up Transformer → KBJ6J Bridge Rectifier

| Transformer Terminal | Rectifier Terminal | Description |
|---|---|---|
| Secondary Winding A | AC Input (~) | Transformer output AC |
| Secondary Winding B | AC Input (~) | Transformer output AC |
| — | + DC Output | Rectified positive rail |
| — | − DC Output | Rectified negative rail (GND) |

### Power Supply: LM2596 Buck Converter

| LM2596 Pin | Connection | Description |
|---|---|---|
| VIN | 12V DC Input | Primary supply input |
| VOUT | 5V (to STM32 VCC) | Regulated control supply |
| GND | System GND | Common ground |

### LCD Display → STM32 GPIO (4-bit mode)

| LCD Pin | STM32 Pin | Description |
|---|---|---|
| RS | GPIO | Register select |
| EN | GPIO | Enable (data latch) |
| D4–D7 | GPIO | 4-bit data bus |
| VCC | 5V | Power |
| GND | GND | Ground |

---

## How It Works

1. **Power-Up** — 12V DC input feeds the LM2596 buck converter, producing a stable 5V rail for the STM32 and control logic.
2. **PWM Generation** — STM32 TIM1 timer generates four complementary PWM channels at ~30 kHz with ~54% duty cycle using bare-metal register configuration.
3. **Gate Driving** — IR2110 amplifies the 3.3V PWM signals to 10–15V gate drive voltage; bootstrap capacitor enables high-side switching.
4. **Full-Bridge Switching** — MOSFETs alternate in diagonal pairs (Q1+Q4, then Q2+Q3) with a 5µs dead-time interval to produce a bipolar square wave AC signal.
5. **Voltage Step-Up** — The ferrite-core transformer (1:3 turns ratio) magnetically steps up the AC signal threefold and provides galvanic isolation.
6. **Rectification** — KBJ6J bridge rectifier performs full-wave rectification, converting the stepped-up AC into pulsating DC.
7. **Filtering** — 1000µF electrolytic capacitor smooths the pulsating DC into stable ~21V DC output.
8. **Display** — LCD shows system status (IDLE / CONNECTED DEVICE) and output voltage in real time via STM32 ADC feedback.

### Switching Sequence (Full-Bridge)

```
Cycle 1: Q1 (PA8-HO) + Q4 (PA11-LO) ON → Current flows: Left→Transformer→Right
Dead Time: All MOSFETs OFF (5µs)
Cycle 2: Q2 (PA9-LO) + Q3 (PA10-HO) ON → Current flows: Right→Transformer→Left
Dead Time: All MOSFETs OFF (5µs)
```

### Key Calculations

```
Switching Frequency:  f = fclk / (ARR + 1) = 8,000,000 / 266 ≈ 30,075 Hz
Duty Cycle:           D = CCR / (ARR + 1) = 133 / 266 ≈ 50%
Transformer Ratio:    Vs / Vp = Ns / Np = 24 / 8 = 3
Output Voltage:       Vout = D × (Ns/Np) × Vin ≈ 0.54 × 3 × 12 ≈ 21V DC
Ripple Voltage:       Vr = I / (f × C)
```

### Performance Metrics

| Parameter | Value |
|---|---|
| Input Voltage | 12V DC |
| Control Supply | 5V DC (LM2596) |
| Switching Frequency | ~30 kHz |
| PWM Duty Cycle | ~54% |
| Transformer Turns Ratio | 1:3 (8T : 24T) |
| Output Voltage | ~21V DC |
| Filter Capacitance | 1000µF |

---

## Communication Protocols

### Bare-Metal STM32 Register Configuration (TIM1 — PWM)

```c
#include "stm32f1xx.h"

#define ARR_VAL  265
#define DUTY     133   // ~50%

void PWM_Init(void) {
    // Enable clocks
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN | RCC_APB2ENR_AFIOEN | RCC_APB2ENR_TIM1EN;

    // Configure PA8–PA11 as Alternate Function Push-Pull (50 MHz)
    GPIOA->CRH &= ~(0xFFFF);
    GPIOA->CRH |=  0xBBBB;

    // Timer config
    TIM1->PSC = 0;
    TIM1->ARR = ARR_VAL;

    // PWM Mode 1 on all four channels
    TIM1->CCMR1 |= (6 << 4)  | (1 << 3)  | (6 << 12) | (1 << 11);
    TIM1->CCMR2 |= (6 << 4)  | (1 << 3)  | (6 << 12) | (1 << 11);

    // Enable channel outputs
    TIM1->CCER  |= (1 << 0) | (1 << 4) | (1 << 8) | (1 << 12);

    // Main Output Enable (required for TIM1 advanced timer)
    TIM1->BDTR  |= (1 << 15);

    // Update event and start
    TIM1->EGR   |= TIM_EGR_UG;
    TIM1->CR1   |= TIM_CR1_CEN;
}
```

### Main Loop — Full-Bridge Switching with Dead Time

```c
while (1) {
    // Cycle 1: Q1 + Q4 ON
    TIM1->CCR1 = DUTY;   // PA8
    TIM1->CCR4 = DUTY;   // PA11
    TIM1->CCR2 = 0;
    TIM1->CCR3 = 0;
    delay_ms(10);

    // Dead time
    TIM1->CCR1 = 0;
    TIM1->CCR4 = 0;
    delay_us(5);

    // Cycle 2: Q2 + Q3 ON
    TIM1->CCR2 = DUTY;   // PA9
    TIM1->CCR3 = DUTY;   // PA10
    TIM1->CCR1 = 0;
    TIM1->CCR4 = 0;
    delay_ms(10);

    // Dead time
    TIM1->CCR2 = 0;
    TIM1->CCR3 = 0;
    delay_us(5);
}
```

### IR2110 Bootstrap Operation

- **Bootstrap capacitor** charges during low-side conduction (VS ≈ 0V)
- During high-side turn-on, capacitor provides floating gate supply (VB − VS ≈ 12V)
- Gate drive voltage: 10–15V ensures full MOSFET enhancement and low R_DS(on)
- Gate resistors control slew rate and suppress switching ringing

---

## Results

The system was successfully implemented and tested on hardware. Key outcomes:

- The STM32 TIM1 timer reliably generated four-channel complementary PWM at ~30 kHz with ~54% duty cycle, confirmed via Saleae Logic Analyzer waveform capture.
- The IR2110 gate driver successfully drove all four IRFZ44N MOSFETs with no shoot-through conditions observed — indicating correct dead-time insertion and switching synchronization.
- The full-bridge MOSFET inverter produced a stable bipolar square wave AC signal at the transformer primary, verified on an oscilloscope.
- The step-up transformer (1:3, ferrite core) stepped up the inverter output voltage by a factor of 3, with the secondary producing high-frequency AC consistent with the design.
- The KBJ6J bridge rectifier and 1000µF filter capacitor produced a smooth, stable ~21V DC output — closely matching the theoretical prediction (Vout = D × (Ns/Np) × Vin).
- The LCD correctly displayed system states ("EV CHARGER SYSTEM IDLE", "CONNECTED DEVICE SOURCE: 9V / 12V / 24V", "CHARGE: 1.4Ah") in real time.
- High-frequency operation significantly reduced passive component size, with the ferrite core transformer remaining compact compared to equivalent 50 Hz designs.

---

## Future Scope

- **Closed-loop voltage regulation** — Implement ADC-based feedback from output voltage to automatically adjust PWM duty cycle and maintain constant Vout under varying load conditions
- **Sinusoidal PWM (SPWM)** — Replace square-wave modulation with SPWM to reduce harmonic distortion and improve power quality
- **Wide-bandgap devices** — Upgrade to GaN or SiC switches to enable MHz-range switching, reduced losses, and higher power density
- **LLC / CLLC resonant topology** — Transition to soft-switching resonant converter for Zero Voltage Switching (ZVS) and lower EMI
- **Fast recovery / Schottky diodes** — Replace KBJ6J bridge rectifier to reduce reverse-recovery losses at high switching frequencies
- **USB Power Delivery (PD) protocol** — Integrate USB-C PD negotiation for multi-voltage output selection (9V / 12V / 15V / 20V)
- **Protection mechanisms** — Add overvoltage protection (OVP), overcurrent protection (OCP), and thermal shutdown using STM32 ADC and GPIO
- **PCB design** — Design a compact custom PCB with optimized layout to minimize parasitic inductance, switching noise, and EMI
- **Bidirectional V2G support** — Extend topology to support Vehicle-to-Grid energy flow using a bidirectional CLLC stage
- **Solar / renewable integration** — Adapt the converter for MPPT-based DC-DC stage in photovoltaic battery charging systems

---

> *A Single Stage Isolated DC-DC Converter for Universal EV Battery Charging.*

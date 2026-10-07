# Class D Subwoofer Amplifier — STM32

> 1st-year project at CentraleSupélec · June 2024  
> Built for the student association **Sono Barco Centrale Supélec (SBCS)**

## Overview

Design and implementation of a **Class D audio amplifier** capable of driving a 1700 W subwoofer. The amplifier achieves >90% efficiency using PWM-based switching and a full-bridge (H-bridge) MOSFET topology.

The full project report is available in [`Rapport_de_soutenance.pdf`](./Rapport_de_soutenance.pdf).

---

## How It Works

```
Audio Input → Level Shifter → STM32 (PWM gen.) → Pre-amp → MOSFET H-Bridge → Low-pass Filter → Speaker
```

1. **Level Shifter** — shifts the ±1.23 V audio jack signal to the 0–3 V range readable by the STM32
2. **STM32 PWM Generation** — the microcontroller generates two complementary 200 kHz PWM signals (one per H-bridge branch) using Timer channels in STM32CubeIDE
3. **Pre-amplifier** — boosts the PWM signal to the gate threshold of the MOSFETs (TL081CP op-amp, 3.3 kΩ / 1 kΩ resistors)
4. **Full-Bridge (H-Bridge)** — four NTPF082N65S3F MOSFETs switch the high-voltage supply across the load; a dead-time is enforced to prevent shoot-through
5. **Low-pass Filter** — removes the high-frequency switching content and reconstructs the amplified analog audio signal

---

## Hardware

| Component | Part | Key Specs |
|---|---|---|
| Microcontroller | STM32 B-L475E-IOT01A2 | Cortex-M4, FPU, DSP, 80 MHz |
| MOSFETs (×4) | NTPF082N65S3F | 650 V, 40 A, R_DS(on) = 82 mΩ |
| Op-amp (pre-amp) | TL081CP | — |
| Heat sink | CO2245V1 (Farnell) | R_th ≤ 7 °C/W |

### Thermal Design

With four MOSFETs each dissipating up to 10 W, the required heat sink thermal resistance was calculated as:

```
R_da ≤ (T_jmax − T_amb) / P_diss − R_jb = (150 − 45) / 10 − 2.62 ≈ 7.88 °C/W
```

The chosen heat sink (CO2245V1, R_th = 7 °C/W) satisfies this constraint.

---

## Firmware

The firmware was developed in **STM32CubeIDE** (STM32L4 HAL).

Key configuration choices:
- PWM frequency: **200 kHz** (set via `Prescaler` and `CounterPeriod`)
- Duty cycle at rest: **50%** (zero audio input)
- Second PWM channel set to **Mode 2** (hardware complement) to avoid phase delay
- ADC sampling drives real-time duty-cycle modulation via sawtooth comparison

### Build & Flash

1. Open the project in **STM32CubeIDE** (`File → Import → Existing Project`)
2. Connect the B-L475E-IOT01A2 board via USB/ST-Link
3. Build (`Ctrl+B`) and flash (`Run → Debug`)

---

## Repository Structure

```
.
├── Core/
│   ├── Inc/          # Header files (main.h, stm32l4xx_it.h, …)
│   └── Src/          # Application source (main.c, PWM config, …)
├── Drivers/
│   ├── CMSIS/        # ARM CMSIS headers
│   └── STM32L4xx_HAL_Driver/   # ST HAL driver source
├── rapport_ampli_1A.pdf         # Full project report (French)
├── Ampli 3.0.ioc                # STM32CubeMX pin/clock configuration
└── README.md
```

> **Note:** The `Debug/` build output folder is excluded from version control via `.gitignore`.

---

## Authors

| Minka Dubourg | Firmware & system integration | Circuit design

Supervised by **Pietro Maris Ferreira** — CentraleSupélec

---

## Next steps

- Implement a **feedback loop** to reduce harmonic distortion
- Design a **custom PCB** to replace the breadboard prototype
- Use a **switching power supply** for better efficiency and voltage regulation

# Hardware Specifications

## STM32F407 Robot Controller Board - Detailed Hardware Documentation

### Table of Contents
1. [Board Overview](#board-overview)
2. [Component Specifications](#component-specifications)
3. [Schematic Details](#schematic-details)
4. [PCB Layout Guidelines](#pcb-layout-guidelines)
5. [Bill of Materials](#bill-of-materials)
6. [Testing and Validation](#testing-and-validation)

---

## Board Overview

### Design Philosophy
This controller board is designed with the following priorities:
- **Reliability**: Industrial-grade components for 24/7 operation
- **Modularity**: Independent motor channels for flexible configuration
- **Expandability**: Multiple communication interfaces for system integration
- **Safety**: Comprehensive protection circuits and fault detection

### Key Specifications Summary

| Category | Specification |
|----------|---------------|
| Microcontroller | STM32F407VGT6 (LQFP100) |
| Motor Drivers | 6x independent H-bridge channels |
| Max Motor Current | 5A continuous per channel (30A total) |
| Input Voltage | 12-24V DC (wide range input) |
| Operating Temp | -20°C to +70°C (industrial) |
| Board Size | 100mm x 80mm x 20mm (with components) |

---

## Component Specifications

### Main Microcontroller

**STM32F407VGT6**
- Package: LQFP100
- Core: ARM Cortex-M4F @ 168 MHz
- Flash: 1024 KB
- RAM: 192 KB (128KB + 64KB CCM)
- Voltage: 1.8V-3.6V (3.3V nominal)
- Features:
  - Hardware FPU (Floating Point Unit)
  - DSP instructions
  - 12 DMA controllers
  - Real-time clock (RTC)
  - True random number generator (RNG)

### Motor Driver ICs

**DRV8871 H-Bridge Motor Drivers** (x6)
- Max Continuous Current: 5A per channel
- Peak Current: 6.5A
- Operating Voltage: 6.5V-45V
- PWM Frequency: Up to 100 kHz
- Internal current limiting
- Thermal shutdown at 150°C
- Undervoltage lockout (UVLO)
- Package: HSOIC-8 with PowerPAD

**Alternative**: L298N (for lower current applications)
- Max Current: 2A per channel
- Operating Voltage: 5V-35V
- Integrated flyback diodes

### Power Supply Components

**Main Regulator: LM2596S-3.3**
- Input: 12-24V DC
- Output: 3.3V @ 3A
- Efficiency: >85%
- Switching Frequency: 150 kHz
- Protection: Thermal shutdown, short circuit

**Backup LDO: AMS1117-3.3**
- Input: 4.5V-15V
- Output: 3.3V @ 1A
- Dropout Voltage: 1.1V
- Package: SOT-223

**5V Regulator: LM7805 or LM2940-5.0**
- For encoder power supply (if 5V encoders used)
- Output: 5V @ 1A
- With heatsink for thermal management

### Protection Components

**Reverse Polarity Protection**
- P-channel MOSFET: IRF9540N or equivalent
- Max Voltage: 100V
- Max Current: 20A continuous

**Overvoltage Protection**
- Transient Voltage Suppressor (TVS): SMBJ30A
- Breakdown Voltage: 30V
- Peak Power: 600W (10/1000μs)

**Fuse**
- Main Power: 5A automotive blade fuse
- Motor Channels: 6A mini blade fuse (optional per channel)

### Passive Components

**Capacitors**
- Input Bulk: 1000μF 35V electrolytic
- Regulator Output: 470μF 6.3V electrolytic
- Decoupling: 100nF ceramic (X7R) near each IC
- Motor Flyback: 100nF 50V ceramic per channel

**Resistors**
- Pull-up/Pull-down: 10kΩ (0603 SMD)
- Current Sense: 0.1Ω 1% 2W (through-hole)
- LED Current Limiting: 330Ω (0603 SMD)
- CAN Termination: 120Ω 1% (optional, jumper selectable)

**Inductors**
- Buck Converter: 33μH, 3A, DCR < 0.05Ω (DO3316 package)

### Connectors

**Power Input**
- Screw Terminal: 2-pin 5.08mm pitch, rated 15A
- Or: XT60 connector for battery applications

**Motor Outputs**
- Screw Terminal: 2-pin 5.08mm pitch per motor (12 pins total)
- Wire gauge support: 12-22 AWG

**Encoder Inputs**
- JST-XH 4-pin (VCC, GND, A, B) per encoder
- Or: 2.54mm pitch pin headers

**Communication**
- UART: 4-pin header (TX, RX, GND, 3.3V)
- CAN: 3-pin screw terminal (CANH, CANL, GND)
- Ethernet: RJ45 connector with integrated magnetics
- USB: Micro-USB type B connector
- I2C: 4-pin header (SDA, SCL, GND, 3.3V)

**Programming/Debug**
- SWD: 4-pin header (SWDIO, SWCLK, GND, 3.3V)
- Or: 20-pin ARM Cortex debug connector (optional)

### Indicators

**LEDs**
- Power LED: Green, 3mm through-hole or 0805 SMD
- Status LED: Orange, 3mm through-hole or 0805 SMD
- Error LED: Red, 3mm through-hole or 0805 SMD
- CAN Activity: Yellow, 0805 SMD
- ETH Activity: Green, 0805 SMD (or integrated in RJ45)

### Crystal Oscillator

**Main Crystal**
- Frequency: 8 MHz (for 168 MHz system clock via PLL)
- Package: HC-49S or 3225 SMD
- Load Capacitance: 20pF
- Accuracy: ±30 ppm

**RTC Crystal**
- Frequency: 32.768 kHz
- Package: cylindrical 3x8mm or 3215 SMD
- Load Capacitance: 12.5pF
- Accuracy: ±20 ppm

### CAN Transceiver

**TJA1050 or SN65HVD230**
- Supply Voltage: 5V
- Bus speed: Up to 1 Mbps
- ESD protection: ±8kV
- Package: SO-8

### Ethernet PHY

**LAN8720A**
- Interface: RMII to STM32F407
- Speed: 10/100 Mbps
- Supply: 3.3V with 1.2V internal LDO
- Package: QFN-24
- Integrated: Auto-negotiation, auto-MDIX

---

## Schematic Details

### Power Supply Section

```
VIN (12-24V) --> [Reverse Protection] --> [Fuse] --> [TVS] --> [LM2596 Buck] --> 3.3V
                                                            |
                                                            --> [LM7805] --> 5V (Encoders)
                                                            |
                                                            --> Motor Power (direct)
```

**Key Design Points:**
- Separate analog and digital ground planes
- Star ground configuration at power input
- Wide power traces (≥2mm for power, ≥1mm for 3.3V)
- Multiple bypass capacitors near each IC

### Motor Driver Section (Per Channel)

```
STM32 PWM --> [Level Shifter] --> DRV8871 IN1
STM32 DIR --> [Level Shifter] --> DRV8871 IN2
                                     |
                                  OUT1/OUT2 --> Motor
                                     |
                              Current Sense --> ADC
```

**Current Sensing:**
- Shunt resistor: 0.1Ω in series with motor ground
- Op-amp: LM358 or similar for amplification (×10 gain)
- Output to STM32 ADC with 0-3.3V range
- Current calculation: I = V_adc / (0.1Ω × 10)

### Encoder Interface

```
Encoder 5V/3.3V --> [Level Shifter] --> STM32 Timer Input (Quadrature mode)
                         |
                    Pull-up resistors (10kΩ)
                         |
                    ESD protection (TVS diodes)
```

### Communication Interfaces

**CAN Bus:**
```
STM32 CAN_TX --> TJA1050 TXD
STM32 CAN_RX <-- TJA1050 RXD
                    |
               CANH/CANL --> External CAN bus
                    |
            [120Ω Termination] (jumper selectable)
```

**Ethernet:**
```
STM32 RMII --> LAN8720A PHY --> RJ45 with magnetics
                    |
              25MHz Crystal
                    |
              LED indicators
```

---

## PCB Layout Guidelines

### Layer Stack-Up (4-layer recommended)

1. **Top Layer**: Signal routing, component pads
2. **Inner Layer 1 (GND)**: Solid ground plane
3. **Inner Layer 2 (PWR)**: Power distribution (3.3V, 5V areas)
4. **Bottom Layer**: Signal routing, ground fill

### Design Rules

| Parameter | Specification |
|-----------|---------------|
| Trace Width (Power) | ≥2mm for VIN, 1.5mm for motor power |
| Trace Width (Signal) | 0.25mm minimum, 0.4mm typical |
| Trace Width (3.3V) | ≥1mm for main rail |
| Clearance | 0.3mm minimum, 0.5mm for high voltage |
| Via Size | 0.6mm drill, 1.0mm pad (power vias) |
| Via Size | 0.3mm drill, 0.6mm pad (signal vias) |

### Critical Routing

**High-Speed Signals:**
- Keep RMII traces short and equal length (±1mm)
- Route as differential pairs where applicable
- Avoid vias in high-speed paths when possible
- Keep away from switching power supply traces

**Motor Driver Power:**
- Star ground connection for motor drivers
- Wide traces for motor current paths
- Keep motor output traces short to reduce EMI
- Add ground plane under motor drivers for heat dissipation

**Analog Signals:**
- Separate analog ground from digital ground (connect at single point)
- Route current sense signals away from PWM traces
- Shield ADC inputs with ground traces
- Use guard rings around sensitive analog circuits

### Thermal Management

**Heat Sinks Required:**
- Motor driver ICs (DRV8871): 10°C/W heatsink recommended
- Voltage regulators under heavy load
- Consider copper pour for additional cooling

**Thermal Vias:**
- Place thermal vias under PowerPAD of DRV8871 (9+ vias)
- Connect to ground plane for heat spreading
- 0.3mm drill, filled if possible

### EMI Mitigation

- Ferrite beads on motor power lines
- RC snubbers across motor terminals (100Ω + 100nF)
- Bypass capacitors close to IC power pins (<10mm)
- Ground plane coverage >80% on inner layers
- Keep switching nodes small (buck converter)

---

## Bill of Materials (BOM)

### Major Components

| Reference | Description | Quantity | Part Number | Manufacturer |
|-----------|-------------|----------|-------------|--------------|
| U1 | Microcontroller STM32F407VGT6 | 1 | STM32F407VGT6 | STMicroelectronics |
| U2-U7 | Motor Driver H-Bridge | 6 | DRV8871DDAR | Texas Instruments |
| U8 | Buck Regulator 3.3V | 1 | LM2596S-3.3 | Texas Instruments |
| U9 | LDO Regulator 3.3V | 1 | AMS1117-3.3 | AMS |
| U10 | CAN Transceiver | 1 | TJA1050T | NXP |
| U11 | Ethernet PHY | 1 | LAN8720A | Microchip |
| Y1 | Crystal 8MHz | 1 | ABM3-8.000MHZ | Abracon |
| Y2 | Crystal 32.768kHz | 1 | ABS07 | Abracon |

### Passive Components (Select List)

| Reference | Description | Quantity | Package |
|-----------|-------------|----------|---------|
| C1 | Cap 1000μF 35V Electrolytic | 1 | Radial |
| C2-C7 | Cap 470μF 6.3V Electrolytic | 6 | Radial |
| C8-C20 | Cap 100nF Ceramic X7R | 30+ | 0603 |
| C21-C26 | Cap 10μF Ceramic X5R | 6 | 0805 |
| R1-R20 | Res 10kΩ 1% | 20 | 0603 |
| R21-R26 | Res 0.1Ω 1% 2W | 6 | Through-hole |
| L1 | Inductor 33μH 3A | 1 | DO3316 |

### Connectors

| Reference | Description | Quantity |
|-----------|-------------|----------|
| J1 | Power Input Terminal 2-pin | 1 |
| J2-J7 | Motor Output Terminal 2-pin | 6 |
| J8-J13 | Encoder Input JST-XH 4-pin | 6 |
| J14 | RJ45 Ethernet with magnetics | 1 |
| J15 | USB Micro-B | 1 |
| J16 | SWD Header 4-pin | 1 |

**Note**: Complete BOM available in separate spreadsheet format.

---

## Testing and Validation

### Inspection Checklist

**Visual Inspection:**
- [ ] Check for solder bridges, especially on fine-pitch ICs
- [ ] Verify correct component orientation (ICs, diodes, electrolytic caps)
- [ ] Inspect for cold solder joints
- [ ] Check connector mechanical integrity

**Continuity Tests:**
- [ ] Power rail continuity (3.3V, 5V, GND)
- [ ] Motor output continuity
- [ ] Communication line continuity
- [ ] SWD programming interface

### Power-Up Testing Sequence

**Step 1: Initial Power-Up (No Load)**
1. Set power supply to 12V, current limit 500mA
2. Connect power (verify polarity!)
3. Measure 3.3V rail (±5%)
4. Measure 5V rail (±5%) if populated
5. Check for excessive current draw (<200mA idle)
6. Verify power LED illuminates

**Step 2: Microcontroller Programming**
1. Connect SWD programmer (ST-Link)
2. Verify connection in STM32CubeProgrammer
3. Flash test firmware
4. Observe boot messages on UART

**Step 3: Motor Driver Testing**
1. Connect single motor (no load)
2. Test each channel individually:
   - Forward direction, 50% PWM
   - Reverse direction, 50% PWM
   - Monitor current consumption
3. Verify no thermal issues after 1 minute

**Step 4: Encoder Testing**
1. Connect encoder to channel 1
2. Rotate motor manually
3. Verify count increments/decrements
4. Check direction sensing
5. Repeat for all encoder channels

**Step 5: Communication Testing**
- UART: Echo test at 115200 baud
- CAN: Loopback test or external device
- Ethernet: Ping test, link LED verification
- USB: Enumeration test

### Functional Tests

**Motor Control Accuracy:**
- Verify PWM frequency (measure with oscilloscope)
- Test speed steps: 0%, 25%, 50%, 75%, 100%
- Measure linearity of speed vs PWM duty cycle
- Verify direction control

**Current Sensing Calibration:**
- Apply known load (power resistor)
- Measure actual current (multimeter)
- Compare with ADC reading
- Adjust calibration constant if needed

**Thermal Testing:**
- Run all motors at 75% load for 30 minutes
- Monitor IC temperatures (thermal camera or sensor)
- Verify all temperatures < 80°C
- Check heatsink effectiveness

**EMI Testing (if available):**
- Run motors at full load
- Measure radiated emissions
- Verify compliance with limits
- Add additional filtering if needed

### Acceptance Criteria

**Pass Criteria:**
- All power rails within ±5% of nominal
- All motor channels functional in both directions
- All encoder inputs reading correctly
- All communication interfaces operational
- No thermal shutdown under rated load
- Idle current consumption <300mA
- All LEDs functional

**Failure Criteria:**
- Any short circuit or excessive current
- Non-functional motor channel
- Microcontroller programming failure
- Thermal shutdown under normal operation
- Communication interface failure

---

## Revision History

| Revision | Date | Changes |
|----------|------|---------|
| A | 2025-01 | Initial hardware design |

---

## References

- [STM32F407 Datasheet](https://www.st.com/resource/en/datasheet/stm32f407vg.pdf)
- [DRV8871 Datasheet](https://www.ti.com/lit/ds/symlink/drv8871.pdf)
- [LAN8720A Datasheet](https://www.microchip.com/wwwproducts/en/LAN8720A)
- AN4488: Getting Started with STM32F4 Hardware Development

---

*This document is subject to change. Always refer to the latest version on the project repository.*

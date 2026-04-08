# STM32 Robot Controller Board


`stm32` `pcb-design` `robotics` `motor-control` `embedded-systems` `hardware-design` `fusion360` `bts7960` `can-bus` `ethernet`

---

## Overview

This is a custom 2-layer PCB for robotics, built around the **STM32F407VGT6** microcontroller. It integrates high-current motor control, sensor inputs, and multiple communication interfaces onto a single board.

---

## Key Features

* **Design Type:** 2-Layer Modular Carrier Board
* **Core Module:** Integrates a core STM32F407VGT6 compute module via pin headers.
* **Motor Control:** 6x DC Motor (BTS7960) headers, 4x Encoders, 4x Servos
* **Communication:** 6x UART, 2x I2C, 1x CAN Bus, 1x Ethernet
* **I/O:** 5x Digital Inputs, 3x Digital Outputs, 2x LEDs
* **Power:** 12V-24V Input, 5V Regulated Output

---

## Design Files

The original hardware source files are in the `/hardware/source/` folder.

1.  **View Images:**
    * [PCB Layout](docs/images/pcb-layout.png)
    * [Schematic](docs/images/schematic.png)
2.  **Open Source Files:**
    * Install [Autodesk Fusion 360](https://www.autodesk.com/products/fusion-360).
    * Open the `.fsch` (schematic) and `.fbrd` (board) files.

---

## Learning Outcomes

This project provided hands-on experience in:
* **Hardware Design:** Full PCB lifecycle from schematic capture to layout in Fusion 360.
* **Power Electronics:** Integrating high-current BTS7960 motor drivers and designing a stable power distribution network.
* **Mixed-Signal Layout:** Managing a layout with noisy high-current traces (motors) and sensitive high-speed lines (Ethernet, CAN).
* **Design for Manufacturing (DFM):** Creating a 2-layer board with clear silkscreen, component spacing, and routing for assembly.
* **Hardware Validation:** Assembling, testing, and debugging the physical board.

---

## Gallery

#### PCB Layout
![PCB Layout](docs/images/pcb-layout.png)

#### Schematic
![Schematic](docs/images/schematic.png)

#### Assembled Board
![Assembled Board #1](docs/images/assembled-pcb1.jpeg)
![Assembled Board #2](docs/images/assembled-pcb2.jpeg)


---

## Contact & Credits

* **Hardware Design:** Ivander Sugiarta Halim
* **GitHub:** [github.com/ivander11](https://github.com/ivander11)
* **LinkedIn:** [linkedin.com/in/ivanderhalim](https://www.linkedin.com/in/ivanderhalim/)

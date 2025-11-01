# STM32F407 Robot Controller Board

Custom STM32F407-based PCB motor controller board designed for robotics applications, specifically optimized for waste sorting robots with multiple motor control requirements.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Hardware](https://img.shields.io/badge/Hardware-Rev%20A-blue)]()
[![Documentation](https://img.shields.io/badge/Docs-Complete-green)](docs/)

## 📚 Documentation

- **[Quick Start Guide](docs/QUICKSTART.md)** - Get started in 15 minutes
- **[Hardware Specifications](docs/HARDWARE_SPECS.md)** - Detailed hardware design and components
- **[Software Setup Guide](docs/SOFTWARE_SETUP.md)** - Firmware development and programming
- **[API Reference](docs/API_REFERENCE.md)** - Communication protocols and interfaces
- **[Contributing Guidelines](CONTRIBUTING.md)** - How to contribute to this project

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Hardware Specifications](#hardware-specifications)
- [Pin Assignments](#pin-assignments)
- [Getting Started](#getting-started)
- [Software Architecture](#software-architecture)
- [Application: Waste Sorting Robot](#application-waste-sorting-robot)
- [Troubleshooting](#troubleshooting)
- [Safety Considerations](#safety-considerations)
- [Contributing](#contributing)
- [License](#license)

## Overview

This board provides a comprehensive motor control solution featuring:
- **6 independent motor driver channels** with H-bridge circuits
- **Encoder inputs** for precise position and velocity feedback
- **Multiple communication interfaces** (UART, CAN, Ethernet)
- **STM32F407VGT6 microcontroller** (ARM Cortex-M4F @ 168MHz)
- **Robust power management** for reliable operation

## Features

### Motor Control
- **6x Motor Driver Channels**: Independent control for up to 6 DC motors
- **PWM Generation**: Configurable frequency and duty cycle for speed control
- **Direction Control**: Bidirectional operation with H-bridge drivers
- **Current Sensing**: Overcurrent protection and monitoring on each channel
- **Encoder Support**: Quadrature encoder inputs for closed-loop control

### Microcontroller (STM32F407VGT6)
- **ARM Cortex-M4F Core**: 168 MHz with FPU and DSP instructions
- **Memory**: 1MB Flash, 192KB RAM
- **Peripherals**: 
  - 17 timers (including advanced control timers)
  - 3x SPI, 3x I2C, 6x USART/UART
  - 2x CAN, 1x USB OTG FS/HS
  - Ethernet MAC 10/100
  - 3x 12-bit ADCs (24 channels)

### Communication Interfaces
- **UART**: Serial communication for debugging and external device control
- **CAN Bus**: Robust industrial communication (up to 1 Mbps)
- **Ethernet**: 10/100 Mbps for network connectivity and remote monitoring
- **USB**: Device/Host capability for firmware updates and data logging

### Power Supply
- **Input Voltage**: 12-24V DC (configurable)
- **Motor Supply**: Separate power rail for motor drivers
- **Logic Supply**: 3.3V regulated for MCU and peripherals
- **Protection**: Reverse polarity, overvoltage, and overcurrent protection

## Hardware Specifications

### Electrical Characteristics

| Parameter | Specification |
|-----------|--------------|
| Input Voltage (Main) | 12-24V DC |
| Input Voltage (Motor) | 12-24V DC (up to 30A total) |
| Logic Voltage | 3.3V @ 1A |
| Motor Channel Current | Up to 5A per channel (peak) |
| PWM Frequency | 1-20 kHz (configurable) |
| Encoder Input Voltage | 3.3V or 5V compatible |
| Operating Temperature | -20°C to +70°C |

### Board Dimensions
- **Size**: 100mm x 80mm (approximate)
- **Mounting**: 4x M3 mounting holes
- **Connectors**: Screw terminals for power, JST-XH for encoders, pin headers for communication

## Pin Assignments

### Motor Driver Channels

| Channel | PWM Pin | DIR Pin | ENABLE Pin | Current Sense |
|---------|---------|---------|------------|---------------|
| Motor 1 | PE9 (TIM1_CH1) | PD0 | PD1 | PA0 (ADC1_IN0) |
| Motor 2 | PE11 (TIM1_CH2) | PD2 | PD3 | PA1 (ADC1_IN1) |
| Motor 3 | PE13 (TIM1_CH3) | PD4 | PD5 | PA2 (ADC1_IN2) |
| Motor 4 | PE14 (TIM1_CH4) | PD6 | PD7 | PA3 (ADC1_IN3) |
| Motor 5 | PA0 (TIM2_CH1) | PD8 | PD9 | PA4 (ADC1_IN4) |
| Motor 6 | PA1 (TIM2_CH2) | PD10 | PD11 | PA5 (ADC1_IN5) |

### Encoder Inputs

| Encoder | Channel A | Channel B | Notes |
|---------|-----------|-----------|-------|
| Encoder 1 | PA6 (TIM3_CH1) | PA7 (TIM3_CH2) | 5V tolerant |
| Encoder 2 | PB4 (TIM3_CH1) | PB5 (TIM3_CH2) | 5V tolerant |
| Encoder 3 | PC6 (TIM8_CH1) | PC7 (TIM8_CH2) | 5V tolerant |
| Encoder 4 | PC8 (TIM8_CH3) | PC9 (TIM8_CH4) | 5V tolerant |
| Encoder 5 | PA15 (TIM2_CH1) | PB3 (TIM2_CH2) | 5V tolerant |
| Encoder 6 | PB6 (TIM4_CH1) | PB7 (TIM4_CH2) | 5V tolerant |

### Communication Interfaces

| Interface | Pins | Configuration |
|-----------|------|---------------|
| UART1 (Debug) | PA9 (TX), PA10 (RX) | 115200 baud default |
| UART2 (External) | PA2 (TX), PA3 (RX) | Configurable |
| CAN1 | PD0 (RX), PD1 (TX) | 500 kbps default |
| Ethernet | RMII interface | RMII pins (PA1, PA2, PA7, PB11, PB12, PB13, PC1, PC4, PC5) |
| USB OTG FS | PA11 (DM), PA12 (DP) | Device mode |
| I2C1 | PB8 (SCL), PB9 (SDA) | For sensors/peripherals |

### Status LEDs

| LED | Pin | Function |
|-----|-----|----------|
| Power LED | - | Power indicator (green) |
| Status LED | PD12 | System status (orange) |
| Error LED | PD13 | Fault/error indicator (red) |
| CAN LED | PD14 | CAN activity (yellow) |
| ETH LED | PD15 | Ethernet activity (green) |

## Getting Started

### Hardware Setup

1. **Power Connection**
   - Connect 12-24V DC power supply to main power terminals (observe polarity!)
   - Connect motor power supply to motor power terminals
   - Verify power LEDs illuminate

2. **Motor Connections**
   - Connect motors to motor output terminals (M1-M6)
   - Wire encoders to corresponding encoder connectors
   - Ensure proper encoder phasing for correct direction sensing

3. **Communication Setup**
   - Connect UART cable for debugging (3.3V logic levels)
   - Optional: Connect CAN bus with 120Ω termination resistors
   - Optional: Connect Ethernet cable for network access

### Software Setup

1. **Development Environment**
   ```bash
   # Install STM32CubeIDE or ARM GCC toolchain
   # Clone firmware repository
   git clone <firmware-repository-url>
   cd firmware
   ```

2. **Build Firmware**
   ```bash
   # Using STM32CubeIDE: Import project and build
   # Using command line:
   make clean
   make all
   ```

3. **Flash Firmware**
   ```bash
   # Using ST-Link programmer
   st-flash write build/firmware.bin 0x8000000
   
   # Or using OpenOCD
   openocd -f interface/stlink.cfg -f target/stm32f4x.cfg \
           -c "program build/firmware.elf verify reset exit"
   ```

4. **Verify Operation**
   - Connect to UART1 at 115200 baud
   - Observe boot messages and system initialization
   - Test motor channels individually before full operation

## Software Architecture

### Motor Control API

```c
// Initialize motor driver
void Motor_Init(uint8_t channel);

// Set motor speed (-100 to +100)
void Motor_SetSpeed(uint8_t channel, int8_t speed);

// Enable/disable motor
void Motor_Enable(uint8_t channel, bool enable);

// Get encoder position
int32_t Motor_GetEncoderCount(uint8_t channel);

// Reset encoder counter
void Motor_ResetEncoder(uint8_t channel);
```

### Communication Protocols

#### UART Protocol
- **Baud Rate**: 115200 (default)
- **Format**: 8N1 (8 data bits, no parity, 1 stop bit)
- **Command Format**: ASCII with newline termination

Example commands:
```
M1:50\n      # Set motor 1 to 50% speed
M2:-75\n     # Set motor 2 to -75% speed (reverse)
E1\n         # Read encoder 1 position
STATUS\n     # Get system status
```

#### CAN Bus Protocol
- **Bitrate**: 500 kbps (configurable)
- **ID Range**: 0x100-0x1FF for motor commands, 0x200-0x2FF for status
- **Format**: Extended CAN frames

| CAN ID | Data | Description |
|--------|------|-------------|
| 0x100 | [channel, speed, enable] | Motor control command |
| 0x200 | [channel, pos_high, pos_low] | Encoder position |
| 0x300 | [status_byte] | System status |

#### Ethernet/TCP
- **Default IP**: 192.168.1.100 (configurable via DHCP)
- **Port**: 8080 for control, 8081 for monitoring
- **Protocol**: JSON over TCP for commands and status

## Application: Waste Sorting Robot

This controller board is optimized for waste sorting robotics applications:

- **Conveyor Control**: Motors 1-2 for dual conveyor belts
- **Gripper/Actuator**: Motors 3-4 for pick-and-place mechanisms
- **Camera Pan/Tilt**: Motors 5-6 for vision system positioning
- **Networking**: Ethernet connection to vision processing unit
- **CAN Bus**: Integration with PLC or supervisory controller

### Typical System Integration

```
[Vision System] --Ethernet--> [STM32F407 Controller] --Motors--> [Conveyors/Grippers]
                                        |
                                     CAN Bus
                                        |
                                      [PLC]
```

## Troubleshooting

### Common Issues

**Problem**: Motors not responding
- Check power supply voltage and connections
- Verify motor enable signals are HIGH
- Check PWM signal generation with oscilloscope
- Confirm motor driver IC is not in thermal shutdown

**Problem**: Encoder values incorrect
- Verify encoder power supply (3.3V or 5V as required)
- Check encoder cable for proper connections
- Ensure encoder channels A and B are not swapped
- Test encoder with multimeter in quadrature mode

**Problem**: CAN communication fails
- Verify CAN termination resistors (120Ω at each end)
- Check CAN_H and CAN_L wiring (not swapped)
- Confirm bitrate matches other devices on bus
- Use CAN analyzer to debug frame transmission

**Problem**: Ethernet not connecting
- Check physical link LED on RJ45 connector
- Verify IP configuration (static vs DHCP)
- Confirm Ethernet cable is not crossover type
- Test with ping command from host PC

**Problem**: Overheating or thermal shutdown
- Reduce motor load or duty cycle
- Improve board cooling (add heatsinks or fan)
- Check for short circuits in motor wiring
- Verify power supply can handle total current draw

### Diagnostics

Use the built-in diagnostics commands:
```
STATUS\n        # Overall system status
DIAG M1\n       # Detailed diagnostics for motor 1
TEMP\n          # Board temperature readings
VOLTAGE\n       # Supply voltage monitoring
ERROR_LOG\n     # Recent error events
```

## Safety Considerations

⚠️ **Important Safety Information**

- Always disconnect power before making connections or modifications
- Use appropriate fuses or circuit breakers on power inputs
- Ensure motors cannot cause injury if they move unexpectedly
- Keep fingers and loose items away from moving parts
- Follow proper ESD procedures when handling the board
- Do not exceed maximum voltage or current ratings
- Provide adequate ventilation and cooling

## Maintenance

- Regularly inspect connections for looseness or corrosion
- Check motor brushes (if applicable) for wear
- Clean encoder sensors to prevent dust buildup
- Update firmware periodically for bug fixes and improvements
- Back up configuration settings before firmware updates

## Technical Support

For technical questions, firmware updates, or hardware modifications:
- **GitHub Issues**: [Open an issue](https://github.com/ivander11/stm32-robot-controller-board/issues)
- **Documentation**: Comprehensive guides in [`/docs`](docs/) folder
- **Discussions**: [GitHub Discussions](https://github.com/ivander11/stm32-robot-controller-board/discussions)

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to report bugs
- How to suggest features
- Development guidelines
- Code style and standards
- Pull request process

## Project Status

- **Hardware**: Revision A - Production ready
- **Firmware**: Version 1.0 - In development
- **Documentation**: Complete

## Roadmap

- [ ] Release firmware v1.0
- [ ] Add example applications
- [ ] ROS2 integration package
- [ ] Web-based configuration interface
- [ ] Hardware revision B with additional features

## Related Projects

- [Firmware Repository](https://github.com/ivander11/stm32-robot-controller-firmware) - STM32F407 firmware
- [ROS Package](https://github.com/ivander11/stm32-robot-controller-ros) - ROS driver (planned)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- STMicroelectronics for STM32F4 HAL libraries
- FreeRTOS for real-time operating system
- lwIP for TCP/IP stack implementation

## Version History

- **v1.0** (2025-01) - Initial release with 6-motor control
- Hardware revision A - First production run

---

**Note**: This documentation describes the hardware design. Firmware implementation may vary. Refer to the firmware repository for software-specific details.

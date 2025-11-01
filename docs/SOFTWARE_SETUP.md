# Software Setup Guide

## STM32F407 Robot Controller Board - Firmware Development Guide

### Table of Contents
1. [Development Environment Setup](#development-environment-setup)
2. [Firmware Architecture](#firmware-architecture)
3. [Building the Firmware](#building-the-firmware)
4. [Programming and Debugging](#programming-and-debugging)
5. [Configuration](#configuration)
6. [API Reference](#api-reference)
7. [Example Applications](#example-applications)

---

## Development Environment Setup

### Required Software

#### Option 1: STM32CubeIDE (Recommended for Beginners)

**Download and Installation:**
1. Visit [STM32CubeIDE Download Page](https://www.st.com/en/development-tools/stm32cubeide.html)
2. Download version for your OS (Windows, Linux, macOS)
3. Install following the installer instructions
4. Register for STMicroelectronics account (free)

**Features:**
- Integrated Eclipse-based IDE
- Built-in GDB debugger
- STM32CubeMX integration
- No additional toolchain required

#### Option 2: Command Line Development (Advanced)

**Required Tools:**
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install gcc-arm-none-eabi gdb-multiarch
sudo apt-get install openocd stlink-tools
sudo apt-get install make cmake

# macOS (using Homebrew)
brew install gcc-arm-embedded
brew install openocd stlink
brew install cmake

# Windows (using MSYS2)
pacman -S mingw-w64-x86_64-arm-none-eabi-gcc
pacman -S mingw-w64-x86_64-openocd
pacman -S mingw-w64-x86_64-cmake
```

**Toolchain Versions:**
- GCC ARM: 10.3 or later
- OpenOCD: 0.11.0 or later
- STLink Tools: 1.7.0 or later

### Optional Tools

**Serial Terminal:**
- Linux: `screen`, `minicom`, or `picocom`
- Windows: PuTTY, TeraTerm
- Cross-platform: CoolTerm, Serial Studio

**CAN Bus Tools:**
- SocketCAN (Linux)
- PCAN-View (Windows)
- CANalyzer (Professional)

**Ethernet Tools:**
- Wireshark for packet capture
- Hercules for TCP/UDP testing

---

## Firmware Architecture

### Directory Structure

```
firmware/
├── Core/
│   ├── Inc/              # Header files
│   │   ├── main.h
│   │   ├── stm32f4xx_it.h
│   │   └── stm32f4xx_hal_conf.h
│   └── Src/              # Source files
│       ├── main.c
│       ├── stm32f4xx_it.c
│       └── system_stm32f4xx.c
├── Drivers/
│   ├── STM32F4xx_HAL_Driver/  # ST HAL library
│   └── CMSIS/                  # ARM CMSIS
├── Middlewares/
│   ├── FreeRTOS/              # RTOS (optional)
│   └── lwIP/                   # TCP/IP stack
├── App/
│   ├── motor_control/         # Motor control module
│   ├── encoder/               # Encoder interface
│   ├── communication/         # UART, CAN, Ethernet
│   └── config/                # Configuration files
├── Makefile
├── STM32F407VGTx_FLASH.ld     # Linker script
└── README.md
```

### Software Modules

**1. Motor Control Module (`motor_control.c/h`)**
- PWM generation using TIM1, TIM2
- Direction control via GPIO
- Speed setpoint management
- Current monitoring via ADC

**2. Encoder Module (`encoder.c/h`)**
- Quadrature decoder using timer encoder mode
- Position and velocity calculation
- Direction sensing
- Overflow handling

**3. Communication Module (`communication.c/h`)**
- UART command parser
- CAN bus message handling
- Ethernet/TCP server
- Protocol implementation

**4. Safety Module (`safety.c/h`)**
- Overcurrent detection
- Thermal monitoring
- Emergency stop handling
- Fault logging

**5. Configuration Module (`config.c/h`)**
- Parameter storage in flash
- Runtime configuration
- Calibration data
- Network settings

---

## Building the Firmware

### Using STM32CubeIDE

1. **Import Project:**
   - File → Open Projects from File System
   - Navigate to firmware directory
   - Select directory and click Finish

2. **Configure Build:**
   - Right-click project → Properties
   - C/C++ Build → Settings
   - Select Debug or Release configuration

3. **Build:**
   - Project → Build All (Ctrl+B)
   - Check console for errors
   - Output: `Debug/firmware.elf` and `firmware.bin`

### Using Makefile

1. **Configure Toolchain Path:**
   ```bash
   # Edit Makefile if needed
   PREFIX = arm-none-eabi-
   ```

2. **Build Commands:**
   ```bash
   # Clean previous build
   make clean
   
   # Build debug version
   make DEBUG=1
   
   # Build release version (optimized)
   make
   
   # Build with verbose output
   make V=1
   ```

3. **Output Files:**
   - `build/firmware.elf` - Executable with debug symbols
   - `build/firmware.bin` - Binary for flashing
   - `build/firmware.hex` - Intel HEX format
   - `build/firmware.map` - Memory map

### Build Configurations

**Debug Build:**
- Optimization: -O0 (no optimization)
- Debug symbols: -g3
- Assertions enabled
- Larger code size, easier debugging

**Release Build:**
- Optimization: -O2 or -Os (size optimization)
- Debug symbols: minimal or none
- Assertions disabled
- Smaller code size, better performance

---

## Programming and Debugging

### Hardware Connections

**SWD Programming Interface:**
```
ST-Link V2    <-->    Board
---------              -----
3.3V          <-->    3.3V (optional, for powering board)
SWDIO         <-->    SWDIO
SWCLK         <-->    SWCLK
GND           <-->    GND
```

### Programming Methods

#### Method 1: STM32CubeIDE

1. Connect ST-Link to board
2. Run → Debug (F11) or Run (Ctrl+F11)
3. IDE will automatically flash and start debugging

#### Method 2: ST-Link Utility (Windows)

1. Open STM32 ST-Link Utility
2. Target → Connect
3. File → Open File (select .bin or .hex)
4. Target → Program & Verify
5. Start address: 0x08000000 (for .bin)

#### Method 3: Command Line (st-flash)

```bash
# Flash binary file
st-flash write build/firmware.bin 0x08000000

# Verify flash
st-flash read firmware_readback.bin 0x08000000 0x100000

# Erase flash
st-flash erase
```

#### Method 4: OpenOCD

```bash
# Start OpenOCD server
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg

# In another terminal, connect with GDB
arm-none-eabi-gdb build/firmware.elf
(gdb) target remote localhost:3333
(gdb) monitor reset halt
(gdb) load
(gdb) monitor reset run
```

### Debugging

#### Using STM32CubeIDE Debugger

1. Set breakpoints by clicking line numbers
2. Run → Debug (F11)
3. Use debug controls:
   - Step Over (F6)
   - Step Into (F5)
   - Resume (F8)
4. Watch variables in Variables view
5. Monitor peripherals in SFR view

#### Using GDB Command Line

```bash
# Start OpenOCD in one terminal
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg

# In another terminal
arm-none-eabi-gdb build/firmware.elf

(gdb) target remote localhost:3333
(gdb) monitor reset halt
(gdb) load
(gdb) break main
(gdb) continue
(gdb) print motor_speed
(gdb) info registers
(gdb) backtrace
```

#### Serial Debug Output

```c
// In firmware, use printf over UART
printf("Motor 1 Speed: %d\r\n", motor_speed);

// Redirect printf to UART in syscalls.c
int _write(int file, char *ptr, int len) {
    HAL_UART_Transmit(&huart1, (uint8_t*)ptr, len, HAL_MAX_DELAY);
    return len;
}
```

View output:
```bash
# Linux
screen /dev/ttyUSB0 115200

# Or
minicom -D /dev/ttyUSB0 -b 115200

# Windows (PowerShell)
# Use PuTTY or TeraTerm
```

---

## Configuration

### System Clock Configuration

The STM32F407 is configured for maximum performance:

```c
// System Clock: 168 MHz
// AHB Clock: 168 MHz
// APB1 Clock: 42 MHz (for timers: 84 MHz)
// APB2 Clock: 84 MHz (for timers: 168 MHz)

// In SystemClock_Config():
RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
RCC_OscInitStruct.HSEState = RCC_HSE_ON;
RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
RCC_OscInitStruct.PLL.PLLM = 8;   // 8 MHz / 8 = 1 MHz
RCC_OscInitStruct.PLL.PLLN = 336; // 1 MHz * 336 = 336 MHz
RCC_OscInitStruct.PLL.PLLP = 2;   // 336 MHz / 2 = 168 MHz
RCC_OscInitStruct.PLL.PLLQ = 7;   // For USB: 48 MHz
```

### Motor Control Parameters

Edit `App/config/motor_config.h`:

```c
// PWM frequency for motor control (Hz)
#define MOTOR_PWM_FREQUENCY     20000  // 20 kHz

// Maximum motor speed (duty cycle 0-100)
#define MOTOR_MAX_SPEED         100

// Current limit per channel (mA)
#define MOTOR_CURRENT_LIMIT     5000   // 5A

// Encoder counts per revolution
#define ENCODER_CPR             2048   // Adjust per encoder

// PID controller gains (if using closed-loop control)
#define MOTOR_PID_KP            1.0f
#define MOTOR_PID_KI            0.1f
#define MOTOR_PID_KD            0.05f
```

### Communication Settings

Edit `App/config/comm_config.h`:

```c
// UART Configuration
#define UART_DEBUG_BAUDRATE     115200
#define UART_EXT_BAUDRATE       115200

// CAN Configuration
#define CAN_BITRATE             500000  // 500 kbps
#define CAN_BASE_ID             0x100

// Ethernet Configuration
#define ETH_IP_ADDR0            192
#define ETH_IP_ADDR1            168
#define ETH_IP_ADDR2            1
#define ETH_IP_ADDR3            100
#define ETH_NETMASK_ADDR0       255
#define ETH_NETMASK_ADDR1       255
#define ETH_NETMASK_ADDR2       255
#define ETH_NETMASK_ADDR3       0
#define ETH_GATEWAY_ADDR0       192
#define ETH_GATEWAY_ADDR1       168
#define ETH_GATEWAY_ADDR2       1
#define ETH_GATEWAY_ADDR3       1
```

### Storing Configuration in Flash

```c
// Write configuration to flash
void Config_Save(void) {
    HAL_FLASH_Unlock();
    
    // Erase sector
    FLASH_EraseInitTypeDef EraseInit;
    EraseInit.TypeErase = FLASH_TYPEERASE_SECTORS;
    EraseInit.Sector = FLASH_SECTOR_11;  // Last sector
    EraseInit.NbSectors = 1;
    EraseInit.VoltageRange = FLASH_VOLTAGE_RANGE_3;
    
    uint32_t SectorError;
    HAL_FLASHEx_Erase(&EraseInit, &SectorError);
    
    // Write data
    uint32_t addr = 0x080E0000;  // Sector 11 start
    HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, addr, config_data);
    
    HAL_FLASH_Lock();
}

// Read configuration from flash
void Config_Load(void) {
    uint32_t addr = 0x080E0000;
    config_data = *(__IO uint32_t*)addr;
}
```

---

## API Reference

### Motor Control API

```c
// Initialize motor controller
void Motor_Init(void);

// Set motor speed (-100 to +100, negative for reverse)
void Motor_SetSpeed(uint8_t channel, int8_t speed);

// Enable or disable motor
void Motor_Enable(uint8_t channel, bool enable);

// Get current motor speed
int8_t Motor_GetSpeed(uint8_t channel);

// Get motor current (mA)
uint16_t Motor_GetCurrent(uint8_t channel);

// Emergency stop all motors
void Motor_EmergencyStop(void);
```

### Encoder API

```c
// Initialize encoder interface
void Encoder_Init(void);

// Get encoder position (counts)
int32_t Encoder_GetPosition(uint8_t channel);

// Reset encoder counter
void Encoder_Reset(uint8_t channel);

// Get encoder velocity (counts per second)
int32_t Encoder_GetVelocity(uint8_t channel);

// Get encoder direction
int8_t Encoder_GetDirection(uint8_t channel);
```

### Communication API

```c
// UART
void UART_Init(void);
void UART_SendString(const char* str);
uint8_t UART_ReceiveByte(void);

// CAN
void CAN_Init(void);
void CAN_SendMessage(uint32_t id, uint8_t* data, uint8_t len);
bool CAN_ReceiveMessage(CAN_Message_t* msg);

// Ethernet
void ETH_Init(void);
void ETH_SendTCP(uint8_t* data, uint16_t len);
uint16_t ETH_ReceiveTCP(uint8_t* buffer, uint16_t max_len);
```

---

## Example Applications

### Example 1: Simple Motor Control

```c
#include "motor_control.h"

int main(void) {
    HAL_Init();
    SystemClock_Config();
    Motor_Init();
    
    // Enable motor 1
    Motor_Enable(1, true);
    
    // Set speed to 50% forward
    Motor_SetSpeed(1, 50);
    
    while (1) {
        HAL_Delay(1000);
    }
}
```

### Example 2: Position Control with Encoder

```c
#include "motor_control.h"
#include "encoder.h"

#define TARGET_POSITION 10000  // counts

int main(void) {
    HAL_Init();
    SystemClock_Config();
    Motor_Init();
    Encoder_Init();
    
    Motor_Enable(1, true);
    Encoder_Reset(1);
    
    while (1) {
        int32_t position = Encoder_GetPosition(1);
        int32_t error = TARGET_POSITION - position;
        
        // Simple proportional controller
        int8_t speed = (int8_t)(error / 100);
        
        // Limit speed to -100..+100
        if (speed > 100) speed = 100;
        if (speed < -100) speed = -100;
        
        Motor_SetSpeed(1, speed);
        
        HAL_Delay(10);  // 100 Hz control loop
    }
}
```

### Example 3: UART Command Interface

```c
#include "motor_control.h"
#include <stdio.h>
#include <string.h>

void Process_Command(char* cmd) {
    if (strncmp(cmd, "M", 1) == 0) {
        // Motor command: M1:50
        uint8_t channel;
        int8_t speed;
        sscanf(cmd, "M%d:%d", &channel, &speed);
        Motor_SetSpeed(channel, speed);
        printf("Motor %d set to %d%%\r\n", channel, speed);
    }
    else if (strncmp(cmd, "E", 1) == 0) {
        // Encoder query: E1
        uint8_t channel;
        sscanf(cmd, "E%d", &channel);
        int32_t pos = Encoder_GetPosition(channel);
        printf("Encoder %d: %ld\r\n", channel, pos);
    }
    else if (strcmp(cmd, "STOP") == 0) {
        Motor_EmergencyStop();
        printf("Emergency stop!\r\n");
    }
}

int main(void) {
    HAL_Init();
    SystemClock_Config();
    UART_Init();
    Motor_Init();
    Encoder_Init();
    
    printf("Robot Controller Ready\r\n");
    
    char cmd_buffer[64];
    uint8_t idx = 0;
    
    while (1) {
        if (UART_DataAvailable()) {
            uint8_t c = UART_ReceiveByte();
            
            if (c == '\n' || c == '\r') {
                cmd_buffer[idx] = '\0';
                Process_Command(cmd_buffer);
                idx = 0;
            }
            else if (idx < sizeof(cmd_buffer) - 1) {
                cmd_buffer[idx++] = c;
            }
        }
    }
}
```

### Example 4: CAN Bus Communication

```c
#include "motor_control.h"

#define CAN_ID_MOTOR_CMD    0x100
#define CAN_ID_MOTOR_STATUS 0x200

void CAN_RxCallback(CAN_Message_t* msg) {
    if (msg->id == CAN_ID_MOTOR_CMD) {
        uint8_t channel = msg->data[0];
        int8_t speed = (int8_t)msg->data[1];
        Motor_SetSpeed(channel, speed);
    }
}

int main(void) {
    HAL_Init();
    SystemClock_Config();
    CAN_Init();
    Motor_Init();
    
    CAN_RegisterCallback(CAN_RxCallback);
    
    while (1) {
        // Send motor status every 100ms
        CAN_Message_t msg;
        msg.id = CAN_ID_MOTOR_STATUS;
        msg.data[0] = 1;  // Channel
        msg.data[1] = Motor_GetSpeed(1);
        msg.data[2] = Motor_GetCurrent(1) >> 8;
        msg.data[3] = Motor_GetCurrent(1) & 0xFF;
        msg.len = 4;
        
        CAN_SendMessage(&msg);
        
        HAL_Delay(100);
    }
}
```

---

## Troubleshooting

### Build Errors

**Error: "arm-none-eabi-gcc: command not found"**
- Solution: Install ARM GCC toolchain and add to PATH

**Error: "undefined reference to `HAL_xxx`"**
- Solution: Ensure all HAL source files are included in build

**Error: "region `FLASH' overflowed"**
- Solution: Code too large, enable optimization (-Os) or reduce features

### Programming Errors

**Error: "No ST-Link detected"**
- Check USB connection
- Install ST-Link drivers
- Try different USB port

**Error: "Cannot connect to target"**
- Verify SWD connections (SWDIO, SWCLK, GND)
- Check board power supply
- Try lowering SWD clock speed

**Error: "Flash verification failed"**
- Erase flash completely before programming
- Check power supply stability
- Try programming at lower speed

---

## Additional Resources

- [STM32F4 HAL Documentation](https://www.st.com/resource/en/user_manual/dm00105879.pdf)
- [FreeRTOS Documentation](https://www.freertos.org/Documentation/RTOS_book.html)
- [lwIP Documentation](https://www.nongnu.org/lwip/)
- Community Forum: [STM32 Community](https://community.st.com/s/)

---

*For hardware-specific details, refer to [HARDWARE_SPECS.md](HARDWARE_SPECS.md)*

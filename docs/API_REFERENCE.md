# API and Interface Documentation

## STM32F407 Robot Controller Board - Communication Protocols and Interfaces

### Table of Contents
1. [Communication Overview](#communication-overview)
2. [UART Protocol](#uart-protocol)
3. [CAN Bus Protocol](#can-bus-protocol)
4. [Ethernet/TCP Protocol](#ethernet-tcp-protocol)
5. [I2C Interface](#i2c-interface)
6. [Status and Error Codes](#status-and-error-codes)
7. [Integration Examples](#integration-examples)

---

## Communication Overview

The STM32F407 Robot Controller Board supports multiple communication interfaces for flexible integration:

| Interface | Speed | Use Case | Cable Length |
|-----------|-------|----------|--------------|
| UART | 115200 baud (configurable) | Debugging, simple control | <15m |
| CAN Bus | 500 kbps (configurable) | Industrial control, multi-device | <100m |
| Ethernet | 10/100 Mbps | Remote monitoring, high-speed data | <100m (twisted pair) |
| I2C | 100/400 kHz | Sensors, peripherals | <3m |

### Communication Protocol Selection Guide

**Use UART when:**
- Simple point-to-point communication
- Debugging and development
- Human-readable commands required
- Low bandwidth application

**Use CAN when:**
- Multiple devices on shared bus
- Harsh industrial environment
- Deterministic real-time communication
- Safety-critical applications

**Use Ethernet when:**
- High-speed data transfer required
- Remote network access needed
- Integration with IT infrastructure
- Web-based monitoring/control

---

## UART Protocol

### Physical Interface

**Hardware Connections:**
```
Controller    <-->    Host Device
---------              -----------
TX (PA9)      <-->    RX
RX (PA10)     <-->    TX
GND           <-->    GND
```

**Electrical Specifications:**
- Logic Level: 3.3V CMOS
- Baud Rate: 115200 (default), configurable: 9600, 19200, 38400, 57600, 115200, 230400
- Data Format: 8 data bits, No parity, 1 stop bit (8N1)
- Flow Control: None

### Command Protocol

**Command Format:**
```
<COMMAND>[:<PARAMETERS>]\n
```

**Response Format:**
```
OK [<DATA>]\n
or
ERROR <CODE> <MESSAGE>\n
```

### Command Reference

#### Motor Control Commands

**Set Motor Speed**
```
Command:  M<channel>:<speed>
Channel:  1-6
Speed:    -100 to +100 (negative = reverse)
Example:  M1:50       # Motor 1 at 50% forward
          M2:-75      # Motor 2 at 75% reverse
Response: OK M1:50
```

**Enable/Disable Motor**
```
Command:  EN<channel>:<state>
Channel:  1-6
State:    0=disable, 1=enable
Example:  EN1:1       # Enable motor 1
Response: OK EN1:1
```

**Get Motor Status**
```
Command:  MS<channel>
Channel:  1-6
Example:  MS1
Response: OK MS1:50,ENABLED,1234  # Speed:50, Enabled, Current:1234mA
```

#### Encoder Commands

**Get Encoder Position**
```
Command:  EP<channel>
Channel:  1-6
Example:  EP1
Response: OK EP1:12345  # Position in encoder counts
```

**Reset Encoder**
```
Command:  ER<channel>
Channel:  1-6
Example:  ER1
Response: OK ER1
```

**Get Encoder Velocity**
```
Command:  EV<channel>
Channel:  1-6
Example:  EV1
Response: OK EV1:1500  # Velocity in counts/second
```

#### System Commands

**Get System Status**
```
Command:  STATUS
Example:  STATUS
Response: OK STATUS:RUNNING,12.5V,45C,0x0000  # State, Voltage, Temp, Error flags
```

**Get Version Information**
```
Command:  VERSION
Example:  VERSION
Response: OK VERSION:1.0.0,HW:A,2025-01-15
```

**Emergency Stop**
```
Command:  ESTOP
Example:  ESTOP
Response: OK ESTOP
Note:     Immediately stops all motors
```

**Reset Controller**
```
Command:  RESET
Example:  RESET
Response: OK RESET
Note:     Software reset after 1 second
```

#### Configuration Commands

**Set PWM Frequency**
```
Command:  PWM:<frequency>
Freq:     1000-20000 Hz
Example:  PWM:15000
Response: OK PWM:15000
```

**Set Current Limit**
```
Command:  LIMIT<channel>:<current>
Channel:  1-6
Current:  0-5000 mA
Example:  LIMIT1:3000
Response: OK LIMIT1:3000
```

**Save Configuration**
```
Command:  SAVE
Example:  SAVE
Response: OK SAVED
Note:     Saves current settings to flash
```

**Load Configuration**
```
Command:  LOAD
Example:  LOAD
Response: OK LOADED
Note:     Loads settings from flash
```

### Example UART Session

```
> VERSION
OK VERSION:1.0.0,HW:A,2025-01-15

> EN1:1
OK EN1:1

> M1:50
OK M1:50

> MS1
OK MS1:50,ENABLED,1234

> EP1
OK EP1:12345

> M1:0
OK M1:0

> EN1:0
OK EN1:0
```

### Error Responses

```
ERROR 001 Invalid command
ERROR 002 Invalid parameter
ERROR 003 Channel out of range
ERROR 004 Motor disabled
ERROR 005 Overcurrent detected
ERROR 006 Overvoltage
ERROR 007 Undervoltage
ERROR 008 Overtemperature
ERROR 009 Communication timeout
ERROR 010 Flash write error
```

---

## CAN Bus Protocol

### Physical Interface

**Hardware Connections:**
```
Controller CAN    <-->    CAN Bus
--------------              --------
CANH              <-->    CANH
CANL              <-->    CANL
GND               <-->    GND

Note: 120Ω termination resistors required at both ends of bus
```

**Electrical Specifications:**
- Standard: CAN 2.0B
- Bit Rate: 500 kbps (default), configurable: 125k, 250k, 500k, 1000k
- Frame Type: Extended (29-bit) or Standard (11-bit)
- Termination: 120Ω at each end

### CAN Message Format

**Standard Frame (11-bit ID):**
```
|  ID (11-bit)  | DLC |  Data (0-8 bytes)  |
```

**Extended Frame (29-bit ID):**
```
|  ID (29-bit)  | DLC |  Data (0-8 bytes)  |
```

### CAN ID Allocation

**Standard 11-bit IDs:**

| ID Range | Purpose | Direction |
|----------|---------|-----------|
| 0x100-0x10F | Motor Commands | Host → Controller |
| 0x110-0x11F | Encoder Queries | Host → Controller |
| 0x200-0x20F | Motor Status | Controller → Host |
| 0x210-0x21F | Encoder Data | Controller → Host |
| 0x300-0x30F | System Commands | Host → Controller |
| 0x310-0x31F | System Status | Controller → Host |
| 0x400-0x4FF | Error Messages | Controller → Host |

### CAN Message Definitions

#### Motor Control Messages

**Motor Speed Command (0x100 + channel)**
```
ID:   0x100 + (channel - 1)  # Example: 0x100 for motor 1
DLC:  2
Data: [channel, speed]
      channel: 1-6
      speed: -100 to +100 (signed int8_t)

Example: Motor 1 to 50%
  ID: 0x100
  Data: [01 32]  # channel=1, speed=50 (0x32)
```

**Motor Enable Command (0x108)**
```
ID:   0x108
DLC:  2
Data: [channel, enable]
      channel: 1-6
      enable: 0=disable, 1=enable

Example: Enable motor 1
  ID: 0x108
  Data: [01 01]
```

#### Motor Status Messages

**Motor Status Response (0x200 + channel)**
```
ID:   0x200 + (channel - 1)
DLC:  8
Data: [channel, speed, state, current_h, current_l, voltage_h, voltage_l, error]
      channel: 1-6
      speed: -100 to +100 (signed)
      state: bit0=enabled, bit1=fault
      current: uint16_t in mA (big-endian)
      voltage: uint16_t in mV (big-endian)
      error: error code

Example: Motor 1 status
  ID: 0x200
  Data: [01 32 01 04 D2 0C 1C 00]
  # channel=1, speed=50, enabled, current=1234mA, voltage=3100mV, no error
```

#### Encoder Messages

**Encoder Query (0x110)**
```
ID:   0x110
DLC:  1
Data: [channel]
      channel: 1-6

Example: Query encoder 1
  ID: 0x110
  Data: [01]
```

**Encoder Data Response (0x210 + channel)**
```
ID:   0x210 + (channel - 1)
DLC:  8
Data: [channel, position_b3, position_b2, position_b1, position_b0, 
       velocity_h, velocity_l, direction]
      position: int32_t in encoder counts (big-endian)
      velocity: int16_t in counts/sec (big-endian)
      direction: -1, 0, +1

Example: Encoder 1 data
  ID: 0x210
  Data: [01 00 00 30 39 05 DC 01]
  # channel=1, position=12345, velocity=1500, direction=forward
```

#### System Messages

**System Status Broadcast (0x310)**
```
ID:   0x310
DLC:  8
Data: [state, voltage_h, voltage_l, temp, error_h, error_l, reserved, reserved]
      state: 0=INIT, 1=READY, 2=RUNNING, 3=ERROR, 4=ESTOP
      voltage: uint16_t in mV
      temp: int8_t in °C
      error: uint16_t error flags

Broadcast: Every 100ms

Example:
  ID: 0x310
  Data: [02 0C 1C 2D 00 00 00 00]
  # state=RUNNING, voltage=3100mV, temp=45°C, no errors
```

**Emergency Stop (0x300)**
```
ID:   0x300
DLC:  0
Data: []

Effect: Immediately stops all motors
```

### CAN Bus Error Messages

**Error Message (0x400)**
```
ID:   0x400
DLC:  4
Data: [error_code, channel, data1, data2]
      error_code: See error code table
      channel: Affected channel (0 if system-wide)
      data1, data2: Additional error information

Example: Overcurrent on motor 1
  ID: 0x400
  Data: [05 01 13 88 00 00]  # Error 5, channel 1, current=5000mA
```

### CAN Example: Python Integration

```python
import can

# Initialize CAN bus
bus = can.interface.Bus(channel='can0', bustype='socketcan', bitrate=500000)

# Send motor command: Motor 1 to 50%
msg = can.Message(
    arbitration_id=0x100,
    data=[0x01, 0x32],  # channel=1, speed=50
    is_extended_id=False
)
bus.send(msg)

# Receive status
for msg in bus:
    if msg.arbitration_id == 0x200:  # Motor 1 status
        channel = msg.data[0]
        speed = msg.data[1] if msg.data[1] < 128 else msg.data[1] - 256
        current = (msg.data[3] << 8) | msg.data[4]
        print(f"Motor {channel}: Speed={speed}%, Current={current}mA")
```

---

## Ethernet TCP Protocol

### Network Configuration

**Default Settings:**
- IP Address: 192.168.1.100
- Subnet Mask: 255.255.255.0
- Gateway: 192.168.1.1
- DHCP: Disabled (static IP)

**Ports:**
- 8080: Command/Control (TCP)
- 8081: Status Monitoring (TCP)
- 8082: Data Streaming (UDP, optional)

### TCP Command Protocol

**Connection:**
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('192.168.1.100', 8080))
```

**Message Format (JSON):**

**Request:**
```json
{
  "id": 1,
  "command": "motor_set",
  "parameters": {
    "channel": 1,
    "speed": 50
  }
}
```

**Response:**
```json
{
  "id": 1,
  "status": "ok",
  "result": {
    "channel": 1,
    "speed": 50,
    "current": 1234
  }
}
```

### Command Reference (JSON)

#### Motor Control

**Set Motor Speed:**
```json
{
  "command": "motor_set",
  "parameters": {
    "channel": 1,
    "speed": 50
  }
}
```

**Get Motor Status:**
```json
{
  "command": "motor_status",
  "parameters": {
    "channel": 1
  }
}

Response:
{
  "status": "ok",
  "result": {
    "channel": 1,
    "speed": 50,
    "enabled": true,
    "current": 1234,
    "voltage": 12500
  }
}
```

#### Encoder

**Get Encoder Position:**
```json
{
  "command": "encoder_get",
  "parameters": {
    "channel": 1
  }
}

Response:
{
  "status": "ok",
  "result": {
    "channel": 1,
    "position": 12345,
    "velocity": 1500,
    "direction": 1
  }
}
```

#### System

**Get System Status:**
```json
{
  "command": "system_status"
}

Response:
{
  "status": "ok",
  "result": {
    "state": "running",
    "voltage": 12.5,
    "temperature": 45,
    "uptime": 3600,
    "errors": []
  }
}
```

**Batch Command:**
```json
{
  "command": "batch",
  "commands": [
    {"command": "motor_set", "parameters": {"channel": 1, "speed": 50}},
    {"command": "motor_set", "parameters": {"channel": 2, "speed": 50}},
    {"command": "motor_set", "parameters": {"channel": 3, "speed": -30}}
  ]
}
```

### Status Monitoring (Port 8081)

Continuous JSON stream of system status:

```json
{
  "timestamp": 1234567890,
  "motors": [
    {"channel": 1, "speed": 50, "current": 1234},
    {"channel": 2, "speed": 50, "current": 1156},
    {"channel": 3, "speed": -30, "current": 890}
  ],
  "encoders": [
    {"channel": 1, "position": 12345, "velocity": 1500},
    {"channel": 2, "position": 10234, "velocity": 1450}
  ],
  "system": {
    "voltage": 12.5,
    "temperature": 45,
    "state": "running"
  }
}
```

Update rate: 10 Hz (configurable)

### Example: Python Ethernet Control

```python
import socket
import json

class RobotController:
    def __init__(self, ip='192.168.1.100', port=8080):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((ip, port))
        self.msg_id = 0
    
    def send_command(self, command, parameters=None):
        self.msg_id += 1
        msg = {
            "id": self.msg_id,
            "command": command,
            "parameters": parameters or {}
        }
        self.sock.send(json.dumps(msg).encode() + b'\n')
        
        response = self.sock.recv(4096).decode()
        return json.loads(response)
    
    def set_motor(self, channel, speed):
        return self.send_command("motor_set", {
            "channel": channel,
            "speed": speed
        })
    
    def get_encoder(self, channel):
        return self.send_command("encoder_get", {
            "channel": channel
        })

# Usage
robot = RobotController('192.168.1.100')
robot.set_motor(1, 50)
encoder_data = robot.get_encoder(1)
print(f"Position: {encoder_data['result']['position']}")
```

---

## I2C Interface

### Physical Interface

**Hardware Connections:**
```
Controller    <-->    I2C Device
----------              ----------
SCL (PB8)     <-->    SCL
SDA (PB9)     <-->    SDA
GND           <-->    GND
3.3V          <-->    VDD (if required)
```

**Electrical Specifications:**
- Clock Speed: 100 kHz (Standard) or 400 kHz (Fast Mode)
- Pull-ups: 4.7kΩ on SCL and SDA (on-board)
- Voltage: 3.3V

### I2C Devices (Optional Peripherals)

**Supported Devices:**
- Temperature Sensors: LM75, TMP102
- IMU: MPU6050, MPU9250
- EEPROM: AT24C256 (for configuration storage)
- GPIO Expander: PCF8574, MCP23017

### Example: Reading Temperature Sensor (LM75)

```c
#define LM75_ADDR 0x48

uint16_t I2C_ReadTemperature(void) {
    uint8_t data[2];
    HAL_I2C_Mem_Read(&hi2c1, LM75_ADDR << 1, 0x00, 1, data, 2, HAL_MAX_DELAY);
    
    int16_t temp_raw = (data[0] << 8) | data[1];
    temp_raw >>= 5;  // 11-bit data
    
    float temperature = temp_raw * 0.125;
    return (uint16_t)(temperature * 10);  // Return in 0.1°C units
}
```

---

## Status and Error Codes

### System States

| Code | State | Description |
|------|-------|-------------|
| 0x00 | INIT | System initializing |
| 0x01 | READY | Ready for operation |
| 0x02 | RUNNING | Normal operation |
| 0x03 | ERROR | Error state |
| 0x04 | ESTOP | Emergency stop active |

### Error Codes

| Code | Name | Description | Recovery |
|------|------|-------------|----------|
| 0x01 | INVALID_CMD | Unknown command | Resend correct command |
| 0x02 | INVALID_PARAM | Invalid parameter value | Check parameter range |
| 0x03 | CHANNEL_RANGE | Channel out of range (1-6) | Use valid channel |
| 0x04 | MOTOR_DISABLED | Motor is disabled | Enable motor first |
| 0x05 | OVERCURRENT | Current limit exceeded | Reduce load, check wiring |
| 0x06 | OVERVOLTAGE | Supply voltage too high | Check power supply |
| 0x07 | UNDERVOLTAGE | Supply voltage too low | Check power supply |
| 0x08 | OVERTEMP | Temperature limit exceeded | Improve cooling |
| 0x09 | COMM_TIMEOUT | Communication timeout | Check connection |
| 0x0A | FLASH_ERROR | Flash write failed | Retry or RMA |
| 0x0B | ENCODER_FAULT | Encoder not detected | Check encoder wiring |
| 0x0C | CAN_BUS_OFF | CAN bus error | Check CAN wiring |
| 0x0D | ETH_LINK_DOWN | Ethernet link down | Check cable |

### Error Flags (Bitmask)

```c
#define ERROR_NONE          0x0000
#define ERROR_OVERCURRENT_1 0x0001
#define ERROR_OVERCURRENT_2 0x0002
#define ERROR_OVERCURRENT_3 0x0004
#define ERROR_OVERCURRENT_4 0x0008
#define ERROR_OVERCURRENT_5 0x0010
#define ERROR_OVERCURRENT_6 0x0020
#define ERROR_OVERVOLTAGE   0x0040
#define ERROR_UNDERVOLTAGE  0x0080
#define ERROR_OVERTEMP      0x0100
#define ERROR_ESTOP         0x0200
#define ERROR_COMM_FAULT    0x0400
```

---

## Integration Examples

### Example 1: ROS Integration (Python)

```python
import rospy
from std_msgs.msg import Int32
import serial

class STM32MotorDriver:
    def __init__(self, port='/dev/ttyUSB0'):
        self.ser = serial.Serial(port, 115200, timeout=1)
        rospy.init_node('stm32_motor_driver')
        
        rospy.Subscriber('/motor1/cmd', Int32, self.motor1_callback)
        self.encoder_pub = rospy.Publisher('/motor1/encoder', Int32, queue_size=10)
        
        self.timer = rospy.Timer(rospy.Duration(0.1), self.read_encoder)
    
    def motor1_callback(self, msg):
        cmd = f"M1:{msg.data}\n"
        self.ser.write(cmd.encode())
    
    def read_encoder(self, event):
        self.ser.write(b"EP1\n")
        response = self.ser.readline().decode()
        if response.startswith("OK EP1:"):
            position = int(response.split(":")[1])
            self.encoder_pub.publish(position)

if __name__ == '__main__':
    driver = STM32MotorDriver()
    rospy.spin()
```

### Example 2: PLC Integration (Ladder Logic via CAN)

```
// Read encoder position via CAN
// Send query message ID 0x110
CAN_TX_ID := 16#110;
CAN_TX_DATA[0] := 1;  // Channel 1
CAN_TX_LEN := 1;
CAN_SEND;

// Receive encoder data ID 0x210
IF CAN_RX_ID = 16#210 THEN
    Encoder_Position := SHL(CAN_RX_DATA[1], 24) OR 
                        SHL(CAN_RX_DATA[2], 16) OR
                        SHL(CAN_RX_DATA[3], 8) OR
                        CAN_RX_DATA[4];
END_IF;

// Set motor speed
CAN_TX_ID := 16#100;
CAN_TX_DATA[0] := 1;     // Channel
CAN_TX_DATA[1] := Speed;  // Speed value
CAN_TX_LEN := 2;
CAN_SEND;
```

---

*For hardware details, see [HARDWARE_SPECS.md](HARDWARE_SPECS.md)*  
*For software development, see [SOFTWARE_SETUP.md](SOFTWARE_SETUP.md)*

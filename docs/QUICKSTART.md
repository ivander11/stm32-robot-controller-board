# Quick Start Guide

## STM32F407 Robot Controller Board - Getting Started in 15 Minutes

This guide will help you get your STM32F407 Robot Controller Board up and running quickly.

---

## What You Need

### Hardware Required
- [ ] STM32F407 Robot Controller Board
- [ ] 12-24V DC power supply (2A minimum)
- [ ] ST-Link V2 programmer (for firmware upload)
- [ ] USB-to-UART adapter (3.3V logic level)
- [ ] At least one DC motor (6V-24V)
- [ ] Optional: Quadrature encoder for the motor

### Software Required
- [ ] STM32CubeIDE or ARM GCC toolchain
- [ ] Serial terminal (PuTTY, screen, or minicom)
- [ ] Firmware source code (download from repository)

---

## Step 1: Hardware Assembly (5 minutes)

### Power Connection

1. **Verify polarity** on your power supply:
   - Positive (+) → Red terminal
   - Negative (-) → Black terminal

2. **Connect power supply** to main power terminals (J1)
   - Input voltage: 12-24V DC
   - Current capacity: 2A minimum for testing, 5A+ for full operation

3. **Verify power LED** illuminates (green LED should be on)

⚠️ **Warning**: Double-check polarity before connecting power! Reverse polarity can damage the board.

### Motor Connection

1. **Connect your first motor** to Motor 1 terminals (J2):
   - Motor wire 1 → Terminal 1
   - Motor wire 2 → Terminal 2
   - Wire order doesn't matter (direction can be reversed in software)

2. **Optional - Connect encoder** to Encoder 1 connector (J8):
   - Red wire → VCC
   - Black wire → GND
   - Yellow/White wire → Channel A
   - Green/Blue wire → Channel B

### Programming Connection

1. **Connect ST-Link V2** to SWD header (J16):
   ```
   ST-Link    Board
   -------    -----
   3.3V   →   3.3V (pin 1)
   SWDIO  →   SWDIO (pin 2)
   SWCLK  →   SWCLK (pin 3)
   GND    →   GND (pin 4)
   ```

### Serial Connection (for debugging)

1. **Connect USB-to-UART adapter** to UART1 header (J17):
   ```
   USB-UART   Board
   --------   -----
   TX     →   RX (PA10)
   RX     →   TX (PA9)
   GND    →   GND
   ```

---

## Step 2: Install Software (5 minutes)

### Option A: STM32CubeIDE (Recommended for beginners)

1. **Download STM32CubeIDE**:
   - Visit: https://www.st.com/en/development-tools/stm32cubeide.html
   - Select your OS (Windows/Linux/macOS)
   - Install following the prompts

2. **Download firmware**:
   ```bash
   git clone https://github.com/ivander11/stm32-robot-controller-firmware.git
   cd stm32-robot-controller-firmware
   ```

3. **Import project** in STM32CubeIDE:
   - File → Open Projects from File System
   - Browse to firmware directory
   - Click Finish

### Option B: Command Line (Advanced)

```bash
# Install toolchain (Ubuntu/Debian)
sudo apt-get install gcc-arm-none-eabi openocd

# Download firmware
git clone https://github.com/ivander11/stm32-robot-controller-firmware.git
cd stm32-robot-controller-firmware

# Build
make
```

---

## Step 3: Flash Firmware (2 minutes)

### Using STM32CubeIDE

1. Click **Run** → **Debug** (or press F11)
2. Wait for firmware to upload
3. IDE should show "Download verified successfully"

### Using Command Line

```bash
# Flash the firmware
st-flash write build/firmware.bin 0x08000000

# You should see:
# Flash written and verified! jolly good!
```

---

## Step 4: First Test (3 minutes)

### Connect to Serial Terminal

**Linux:**
```bash
screen /dev/ttyUSB0 115200
# or
minicom -D /dev/ttyUSB0 -b 115200
```

**Windows:**
- Open PuTTY
- Select Serial
- Port: COM3 (or your port)
- Speed: 115200
- Click Open

**macOS:**
```bash
screen /dev/tty.usbserial-* 115200
```

### Verify Boot Messages

You should see:
```
STM32F407 Robot Controller v1.0.0
Hardware: Rev A
Initializing...
  Motors: OK
  Encoders: OK
  CAN: OK
  Ethernet: OK
System Ready
>
```

### Test Motor Control

Type the following commands in the serial terminal:

```
1. Check version
> VERSION
OK VERSION:1.0.0,HW:A,2025-01-15

2. Enable motor 1
> EN1:1
OK EN1:1

3. Run motor at 25% speed
> M1:25
OK M1:25
```

**Your motor should start spinning!** 🎉

```
4. Increase speed to 50%
> M1:50
OK M1:50

5. Reverse direction
> M1:-50
OK M1:-50

6. Stop motor
> M1:0
OK M1:0

7. Disable motor
> EN1:0
OK EN1:0
```

### Test Encoder (if connected)

```
> EP1
OK EP1:12345

Rotate motor manually and read again:
> EP1
OK EP1:12567
```

Position should change when you rotate the motor!

---

## Step 5: Next Steps

Congratulations! Your board is working. Here's what to do next:

### Learn More

- **Full Documentation**: See [README.md](../README.md)
- **Hardware Details**: See [docs/HARDWARE_SPECS.md](HARDWARE_SPECS.md)
- **Software Guide**: See [docs/SOFTWARE_SETUP.md](SOFTWARE_SETUP.md)
- **API Reference**: See [docs/API_REFERENCE.md](API_REFERENCE.md)

### Try Advanced Features

1. **Connect multiple motors**:
   - Repeat motor connection for motors 2-6
   - Control with M2, M3, etc.

2. **Use CAN bus**:
   - Connect to CAN network
   - See API_REFERENCE.md for CAN protocol

3. **Enable Ethernet**:
   - Connect Ethernet cable
   - Access via JSON/TCP (port 8080)

4. **Develop custom firmware**:
   - Modify source code
   - Add your own control algorithms
   - Implement custom protocols

---

## Troubleshooting Quick Fixes

### Problem: Power LED not illuminating

**Solution**:
- Check power supply voltage (should be 12-24V)
- Verify polarity (+ and -)
- Check fuse (if equipped)
- Try different power supply

### Problem: Motor not running

**Solution**:
1. Verify motor is enabled: `EN1:1`
2. Check motor connections are secure
3. Try higher speed: `M1:75`
4. Test motor with different power supply
5. Check current limit: `MS1` (should show current)

### Problem: Can't connect with ST-Link

**Solution**:
- Check SWD cable connections
- Verify board has power (power LED on)
- Try different USB port
- Update ST-Link firmware
- Use lower SWD clock speed

### Problem: No serial output

**Solution**:
- Check TX/RX connections (they should be crossed)
- Verify baud rate is 115200
- Try different USB-UART adapter
- Check adapter is 3.3V logic level (not 5V)

### Problem: Encoder values not changing

**Solution**:
- Verify encoder power (3.3V or 5V as required)
- Check all 4 encoder wires connected
- Swap A and B channels if counting backwards
- Test encoder with multimeter (should see pulses)

---

## Getting Help

If you're stuck:

1. **Check full documentation** in [docs/](.)
2. **Review example code** in firmware repository
3. **Post an issue** on GitHub with:
   - What you tried
   - Error messages
   - Photos of your setup
4. **Join the community** forum (if available)

---

## Safety Reminders

⚠️ **Important**:
- Always disconnect power before changing connections
- Keep hands away from moving motors
- Use appropriate fuses for your application
- Don't exceed maximum current ratings (5A per motor)
- Ensure adequate ventilation and cooling

---

## What's Next?

Now that your board is working, you can:

- **Build a robot**: Use multiple motors for wheels, arms, grippers
- **Add sensors**: Connect via I2C for environmental sensing
- **Implement control**: PID loops, trajectory planning
- **Integrate systems**: Connect to ROS, PLCs, or other controllers
- **Contribute**: Share your projects and improvements!

---

**Happy building!** 🤖

For detailed information, see the complete documentation in the [docs](.) folder.

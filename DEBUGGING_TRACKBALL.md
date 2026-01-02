# Trackball Debugging Guide

## Configuration Changes
- **Right side is now CENTRAL** (connects via USB, has trackball)
- **Left side is now PERIPHERAL** (connects to right via BLE)
- USB logging is enabled on right side

## Hardware Connections
Right side pins (with trackball):
- SDIO (MOSI/MISO): P0.20
- SCLK: P0.22
- CS: P1.00 (GPIO_ACTIVE_HIGH)
- Motion/INT: P0.24 (GPIO_ACTIVE_LOW)
- SPI Frequency: 1MHz

## Accessing Logs on Windows 11

### Method 1: Device Manager + PuTTY (Recommended)
1. **Find COM Port:**
   - Press `Win + X` and select "Device Manager"
   - Expand "Ports (COM & LPT)"
   - Look for "USB Serial Device (COMx)" where x is a number
   - Note the COM port number (e.g., COM3, COM4, etc.)

2. **Install PuTTY:**
   - Download from: https://www.putty.org/
   - Install and run PuTTY

3. **Connect:**
   - In PuTTY configuration:
     - Connection type: Serial
     - Serial line: COM3 (use your COM port number)
     - Speed: 115200
   - Click "Open"
   - You should see log output immediately

### Method 2: PowerShell
```powershell
# Find COM port
Get-PnpDevice -Class Ports | Where-Object {$_.FriendlyName -like "*USB Serial*"}

# Connect (replace COM3 with your port)
$port = new-Object System.IO.Ports.SerialPort COM3,115200,None,8,one
$port.Open()
while($true) {
    if($port.BytesToRead) {
        $port.ReadExisting()
    }
}
```

### Method 3: Arduino IDE Serial Monitor
1. Install Arduino IDE
2. Tools → Port → Select COM port
3. Tools → Serial Monitor
4. Set baud rate to 115200

## What to Look For in Logs

### Successful PMW3610 Detection:
```
[input_pmw3610] PMW3610 found, chip id: 0x3E
[input_pmw3610] PMW3610 initialized
```

### SPI Communication Issues:
```
[spi] SPI transaction failed
[input_pmw3610] Failed to read chip id
```

### Motion Events:
```
[input] Motion event: x=10 y=5
[zmk] Mouse movement: dx=10 dy=5
```

## Troubleshooting Steps

If no chip detection:
1. Check power (3.3V) with multimeter
2. Verify all 4 SPI connections
3. Try CS pin polarity: change `GPIO_ACTIVE_HIGH` to `GPIO_ACTIVE_LOW`
4. Try lower SPI frequency: change `1000000` to `500000`

If chip detected but no motion:
1. Check motion pin connection (P0.24)
2. Try motion pin polarity: change `GPIO_ACTIVE_LOW` to `GPIO_ACTIVE_HIGH`
3. Verify trackball sensor is clean

## Reverting Configuration
To switch back to original (left=central):
- Edit `boards/shields/Comet/Kconfig.defconfig`
- Change `SHIELD_COMET_RIGHT` back to `SHIELD_COMET_LEFT`
- Swap col-offset values back
- Disable USB logging in comet_right.conf

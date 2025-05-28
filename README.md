# QMI8658 Arduino Library

A comprehensive Arduino library for the QMI8658 6-axis Inertial Measurement Unit (IMU) sensor.

## Features

- ✅ 3-axis accelerometer readings
- ✅ 3-axis gyroscope readings  
- ✅ Temperature sensor readings
- ✅ Configurable measurement ranges and output data rates
- ✅ Multiple unit options (m/s² or mg for acceleration, rad/s or dps for gyroscope)
- ✅ Wake-on-motion functionality
- ✅ I2C communication with custom pin configuration
- ✅ Compatible with ESP32, Arduino, and other microcontrollers
- ✅ Easy-to-use API

## Installation

### Method 1: Arduino Library Manager
1. Open Arduino IDE
2. Go to **Sketch > Include Library > Manage Libraries**
3. Search for "QMI8658"
4. Click "Install"

### Method 2: Manual Installation
1. Download this repository as ZIP
2. In Arduino IDE: **Sketch > Include Library > Add .ZIP Library**
3. Select the downloaded ZIP file

### Method 3: Git Clone
```bash
cd ~/Documents/Arduino/libraries/
git clone https://github.com/lahavg/QMI8658-Arduino-Library.git
```

## Wiring

## Wiring

### Basic I2C Connection
| QMI8658 Pin | Arduino Pin | ESP32 Pin | Raspberry Pi Pin | RP2040/Pico Pin | Notes |
|-------------|-------------|-----------|------------------|-----------------|-------|
| VCC         | 3.3V        | 3.3V      | 3.3V (Pin 1)     | 3V3 (Pin 36)    | **Important: Use 3.3V, not 5V** |
| GND         | GND         | GND       | GND (Pin 6)      | GND (Pin 38)    | Ground |
| SDA         | A4 (Uno/Nano) | 21 (default) | GPIO 2 (Pin 3) | GP6 (Pin 9) or GP4 (Pin 6) | I2C Data |
| SCL         | A5 (Uno/Nano) | 22 (default) | GPIO 3 (Pin 5) | GP7 (Pin 10) or GP5 (Pin 7) | I2C Clock |



### I2C Address
The QMI8658 has two possible I2C addresses:
- `0x6A` (default, when SA0 pin is low)
- `0x6B` (when SA0 pin is high)

The library automatically detects the correct address.

### Platform-Specific Notes

#### RP2040/Raspberry Pi Pico
- Two I2C interfaces available: I2C0 and I2C1
- **Default pins**: SDA=GP4, SCL=GP5 (I2C0) | SDA=GP6, SCL=GP7 (I2C1)
- Library automatically selects Wire or Wire1 based on pins
- Use `imu.begin(6, 7)` for pins 6,7 (uses Wire1 automatically)
- Use `imu.begin(4, 5)` for pins 4,5 (uses Wire)

#### Raspberry Pi
- I2C must be enabled: `sudo raspi-config` → Interface Options → I2C → Enable
- Install required packages: `sudo apt-get install i2c-tools python3-smbus`
- Test connection: `i2cdetect -y 1` (should show device at 0x6A or 0x6B)
- Default I2C bus is `/dev/i2c-1`

#### ESP32/ESP8266
- Can use any GPIO pins for I2C
- Default pins: SDA=21, SCL=22 (ESP32) | SDA=4, SCL=5 (ESP8266)
- Use `Wire.begin(SDA_PIN, SCL_PIN)` for custom pins

#### Arduino Uno/Nano
- Fixed I2C pins: SDA=A4, SCL=A5
- Pull-up resistors (4.7kΩ) recommended for long wires
- Use 3.3V logic level shifter if needed

### I2C Address
The QMI8658 has two possible I2C addresses:
- `0x6A` (default, when SA0 pin is low)
- `0x6B` (when SA0 pin is high)

The library automatically detects the correct address.

## Quick Start

```cpp
#include <QMI8658.h>

QMI8658 imu;

void setup() {
    Serial.begin(115200);
    
    // Initialize with default I2C pins
    if (!imu.begin()) {
        Serial.println("Failed to initialize QMI8658!");
        while(1);
    }
    
    // ⭐ NEW: Configure units and precision
    imu.setAccelUnit_mg(true);      // Use mg (milligravity) - common in IMU applications
    imu.setGyroUnit_dps(true);      // Use dps (degrees per second) - intuitive
    imu.setDisplayPrecision(6);     // 6 decimal places for high precision
    
    Serial.println("QMI8658 initialized successfully with mg/dps units!");
}

void loop() {
    QMI8658_Data data;
    
    if (imu.readSensorData(data)) {
        // Print with automatic formatting using current units and precision
        imu.printSensorData(data);
        
        // Or print manually:
        Serial.print("ACC_X = "); Serial.print(data.accelX, imu.getDisplayPrecision());
        Serial.println(imu.isAccelUnit_mg() ? " mg" : " m/s²");
    }
    
    delay(100);
}
```

## API Reference

### Initialization

#### `begin()`
Initialize the sensor with default I2C pins and address.
```cpp
bool begin(TwoWire &wire = Wire, uint8_t address = QMI8658_ADDRESS_LOW);
```

#### `begin()` with custom pins
Initialize the sensor with custom I2C pins.
```cpp
bool begin(uint8_t sda_pin, uint8_t scl_pin, uint8_t address = QMI8658_ADDRESS_LOW);
```

**Examples:**
```cpp
// Use default pins
imu.begin();

// Use custom pins (ESP32)
imu.begin(21, 22);  // SDA=21, SCL=22

// RP2040 with pins 6,7 (uses Wire1 automatically)
imu.begin(6, 7);

// Use specific I2C address
imu.begin(Wire, QMI8658_ADDRESS_HIGH);
```

### ⭐ NEW: Unit Configuration

The library now supports flexible unit selection for both accelerometer and gyroscope readings.

#### Accelerometer Units
```cpp
// Choose between mg (milligravity) and m/s²
void setAccelUnit_mg(bool use_mg = true);      // true = mg, false = m/s²
void setAccelUnit_mps2(bool use_mps2 = true);  // true = m/s², false = mg

// Check current setting
bool isAccelUnit_mg();    // Returns true if set to mg
bool isAccelUnit_mps2();  // Returns true if set to m/s²
```

#### Gyroscope Units
```cpp
// Choose between dps (degrees per second) and rad/s (radians per second)
void setGyroUnit_dps(bool use_dps = true);     // true = dps, false = rad/s
void setGyroUnit_rads(bool use_rads = true);   // true = rad/s, false = dps

// Check current setting
bool isGyroUnit_dps();    // Returns true if set to dps
bool isGyroUnit_rads();   // Returns true if set to rad/s
```

#### Unit Examples
```cpp
// Setup for IMU display compatibility (like your screen)
imu.setAccelUnit_mg(true);       // ACC_X = -965.82 mg format
imu.setGyroUnit_dps(true);       // GYR_X = 0.43 dps format

// Setup for scientific applications  
imu.setAccelUnit_mps2(true);     // 9.81 m/s² format
imu.setGyroUnit_rads(true);      // 0.123 rad/s format

// Check what's currently set
if (imu.isAccelUnit_mg()) {
    Serial.println("Accelerometer in mg");
}
```

### ⭐ NEW: Precision Control

Control the number of decimal places for sensor readings.

```cpp
// Set decimal precision
void setDisplayPrecision(int decimals);                    // 0-10 decimal places
void setDisplayPrecision(QMI8658_Precision precision);     // Use enum
int getDisplayPrecision();                                 // Get current setting

// Precision options
enum QMI8658_Precision {
    QMI8658_PRECISION_2 = 2,    // 2 decimal places (e.g., 9.81)
    QMI8658_PRECISION_4 = 4,    // 4 decimal places (e.g., 9.8100)  
    QMI8658_PRECISION_6 = 6     // 6 decimal places (e.g., 9.810000)
};
```

**Examples:**
```cpp
// High precision like your IMU screen
imu.setDisplayPrecision(6);                    // ACC_X = -965.820000
imu.setDisplayPrecision(QMI8658_PRECISION_6);  // Same as above

// Lower precision for simple displays
imu.setDisplayPrecision(2);                    // ACC_X = -965.82

// Get current precision
int precision = imu.getDisplayPrecision();    // Returns 6
```

### Data Reading Methods

#### Standard Reading (uses current unit settings)
```cpp
bool readAccel(float &x, float &y, float &z);       // Units depend on setAccelUnit_*
bool readGyro(float &x, float &y, float &z);        // Units depend on setGyroUnit_*
bool readTemp(float &temperature);                  // Always in °C
bool readSensorData(QMI8658_Data &data);           // All data with current units
```

#### ⭐ NEW: Specific Unit Reading (ignores current settings)
```cpp
bool readAccelMG(float &x, float &y, float &z);     // Always returns mg
bool readAccelMPS2(float &x, float &y, float &z);   // Always returns m/s²
bool readGyroDPS(float &x, float &y, float &z);     // Always returns dps
bool readGyroRADS(float &x, float &y, float &z);    // Always returns rad/s
```

**Examples:**
```cpp
QMI8658_Data data;
float ax_mg, ay_mg, az_mg;
float gx_dps, gy_dps, gz_dps;

// Method 1: Use current unit settings
if (imu.readSensorData(data)) {
    // data.accelX will be in mg or m/s² depending on setAccelUnit_* setting
    Serial.print("Accel: "); Serial.println(data.accelX, imu.getDisplayPrecision());
}

// Method 2: Force specific units regardless of settings
if (imu.readAccelMG(ax_mg, ay_mg, az_mg)) {
    // Always in mg, regardless of setAccelUnit_* setting
    Serial.print("Accel (mg): "); Serial.println(ax_mg, 6);
}

if (imu.readGyroDPS(gx_dps, gy_dps, gz_dps)) {
    // Always in dps, regardless of setGyroUnit_* setting  
    Serial.print("Gyro (dps): "); Serial.println(gx_dps, 6);
}
```

### ⭐ NEW: Formatted Output

```cpp
// Print sensor data with current units and precision
void printSensorData(QMI8658_Data &data);            // Uses current precision
void printSensorData(QMI8658_Data &data, int precision); // Override precision
```

**Example:**
```cpp
QMI8658_Data data;
if (imu.readSensorData(data)) {
    // Print with current settings
    imu.printSensorData(data);
    
    // Output example:
    // ACC_X = -965.820000 mg
    // ACC_Y = 44.920000 mg  
    // ACC_Z = 36.130000 mg
    // GYR_X = 0.430000 dps
    // GYR_Y = -1.620000 dps
    // GYR_Z = 0.280000 dps
    // TEMP  = 28.7 °C
    
    // Or with custom precision
    imu.printSensorData(data, 2);  // Override to 2 decimal places
}
```

### Configuration

#### Accelerometer Range
```cpp
bool setAccelRange(QMI8658_AccelRange range);
```

Available ranges:
- `QMI8658_ACCEL_RANGE_2G` (±2g)
- `QMI8658_ACCEL_RANGE_4G` (±4g)
- `QMI8658_ACCEL_RANGE_8G` (±8g, default)
- `QMI8658_ACCEL_RANGE_16G` (±16g)

#### Accelerometer Output Data Rate
```cpp
bool setAccelODR(QMI8658_AccelODR odr);
```

Available rates:
- `QMI8658_ACCEL_ODR_8000HZ` to `QMI8658_ACCEL_ODR_31_25HZ`
- `QMI8658_ACCEL_ODR_1000HZ` (default)
- Low power modes: `QMI8658_ACCEL_ODR_LOWPOWER_128HZ`, `QMI8658_ACCEL_ODR_LOWPOWER_21HZ`, etc.

#### Gyroscope Range
```cpp
bool setGyroRange(QMI8658_GyroRange range);
```

Available ranges:
- `QMI8658_GYRO_RANGE_32DPS` (±32°/s)
- `QMI8658_GYRO_RANGE_64DPS` (±64°/s)
- `QMI8658_GYRO_RANGE_128DPS` (±128°/s)
- `QMI8658_GYRO_RANGE_256DPS` (±256°/s)
- `QMI8658_GYRO_RANGE_512DPS` (±512°/s, default)
- `QMI8658_GYRO_RANGE_1024DPS` (±1024°/s)
- `QMI8658_GYRO_RANGE_2048DPS` (±2048°/s)
- `QMI8658_GYRO_RANGE_4096DPS` (±4096°/s)

#### Gyroscope Output Data Rate
```cpp
bool setGyroODR(QMI8658_GyroODR odr);
```

Available rates:
- `QMI8658_GYRO_ODR_8000HZ` to `QMI8658_GYRO_ODR_31_25HZ`
- `QMI8658_GYRO_ODR_1000HZ` (default)

### Unit Configuration

#### Set Accelerometer Units
```cpp
void setAccelUnit_mps2(bool use_mps2 = true);
```
- `true`: m/s² (default)
- `false`: mg (milligravity)

#### Set Gyroscope Units
```cpp
void setGyroUnit_rads(bool use_rads = true);
```
- `true`: rad/s (radians per second)
- `false`: dps (degrees per second, default)

### Reading Data

#### Read All Sensor Data (Recommended)
```cpp
bool readSensorData(QMI8658_Data &data);
```

The `QMI8658_Data` structure contains:
```cpp
struct QMI8658_Data {
    float accelX, accelY, accelZ;  // Acceleration
    float gyroX, gyroY, gyroZ;     // Angular velocity
    float temperature;             // Temperature in °C
    uint32_t timestamp;            // Internal timestamp
};
```

#### Read Individual Sensors
```cpp
bool readAccel(float &x, float &y, float &z);
bool readGyro(float &x, float &y, float &z);
bool readTemp(float &temperature);
```

### Sensor Control

#### Enable/Disable Sensors
```cpp
bool enableAccel(bool enable = true);
bool enableGyro(bool enable = true);
bool enableSensors(uint8_t enableFlags);
```

Enable flags:
- `QMI8658_DISABLE_ALL` (0x00)
- `QMI8658_ENABLE_ACCEL` (0x01)
- `QMI8658_ENABLE_GYRO` (0x02)
- `QMI8658_ENABLE_MAG` (0x04)
- `QMI8658_ENABLE_AE` (0x08)

### Utility Functions

#### Check Data Ready
```cpp
bool isDataReady();
```

#### Get Device ID
```cpp
uint8_t getWhoAmI();  // Should return 0x05
```

#### Reset Sensor
```cpp
bool reset();
```

### Wake-on-Motion

#### Enable Wake-on-Motion
```cpp
bool enableWakeOnMotion(uint8_t threshold = 32);
```

#### Disable Wake-on-Motion
```cpp
bool disableWakeOnMotion();
```

## Complete Example

```cpp
#include <QMI8658.h>

QMI8658 imu;

// Pin definitions for ESP32
#define SDA_PIN 21
#define SCL_PIN 22

void setup() {
    Serial.begin(115200);
    Serial.println("QMI8658 Advanced Example");
    
    // Initialize with custom pins
    if (!imu.begin(SDA_PIN, SCL_PIN)) {
        Serial.println("Failed to initialize QMI8658!");
        while(1) delay(1000);
    }
    
    Serial.println("QMI8658 initialized!");
    Serial.print("Device ID: 0x");
    Serial.println(imu.getWhoAmI(), HEX);
    
    // Configure sensor
    imu.setAccelRange(QMI8658_ACCEL_RANGE_8G);
    imu.setAccelODR(QMI8658_ACCEL_ODR_1000HZ);
    imu.setGyroRange(QMI8658_GYRO_RANGE_512DPS);
    imu.setGyroODR(QMI8658_GYRO_ODR_1000HZ);
    
    // Set units
    imu.setAccelUnit_mps2(true);   // m/s²
    imu.setGyroUnit_rads(false);   // degrees per second
    
    // Enable sensors
    imu.enableSensors(QMI8658_ENABLE_ACCEL | QMI8658_ENABLE_GYRO);
    
    Serial.println("Configuration complete!");
    delay(100);
}

void loop() {
    if (imu.isDataReady()) {
        QMI8658_Data data;
        
        if (imu.readSensorData(data)) {
            // Print formatted data
            Serial.printf("Accel: %6.2f, %6.2f, %6.2f m/s² | ", 
                         data.accelX, data.accelY, data.accelZ);
            Serial.printf("Gyro: %7.2f, %7.2f, %7.2f dps | ", 
                         data.gyroX, data.gyroY, data.gyroZ);
            Serial.printf("Temp: %5.1f°C\n", data.temperature);
        }
    }
    
    delay(50); // 20Hz output
}
```

## Troubleshooting

### Common Issues

1. **"Failed to initialize QMI8658!"**
   - Check wiring connections
   - Ensure 3.3V power supply (not 5V)
   - Try both I2C addresses (0x6A and 0x6B)
   - Add pull-up resistors to SDA/SCL lines (4.7kΩ)

2. **Erratic readings**
   - Ensure stable power supply
   - Keep sensor away from magnetic interference
   - Allow time for sensor stabilization after power-on

3. **No data or zero readings**
   - Check if sensors are enabled
   - Verify correct pin assignments
   - Use `isDataReady()` to check data availability

### Debug Tips

```cpp
// Check device communication
Serial.print("WHO_AM_I: 0x");
Serial.println(imu.getWhoAmI(), HEX);  // Should be 0x05

// Check data ready status
if (!imu.isDataReady()) {
    Serial.println("Data not ready");
}
```

## Hardware Compatibility

### Tested Boards
- ✅ ESP32 (all variants)
- ✅ Arduino Uno/Nano/Pro Mini
- ✅ Arduino Mega
- ✅ ESP8266
- ✅ Raspberry Pi Pico (Arduino framework)

### Requirements
- **Voltage**: 3.3V (DO NOT use 5V)
- **Communication**: I2C
- **Pull-up resistors**: May be required for SDA/SCL (4.7kΩ recommended)

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This library is released under the MIT License. See [LICENSE](LICENSE) file for details.

## Credits

Based on the original QMI8658 code from Waveshare, adapted and enhanced for Arduino compatibility.

## Support

If you encounter any issues or have questions:
1. Check the troubleshooting section above
2. Search existing issues on GitHub
3. Create a new issue with detailed information

---

**Happy coding! 🚀**

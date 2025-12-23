# MLX90642 Python Guide for Raspberry Pi 3B+

## Overview
This guide shows how to use the MLX90642 32x24 infrared sensor array with a Raspberry Pi using Python. The MLX90642 communicates via I2C and provides temperature measurements or normalized infrared data.

## Hardware Setup

### Connections
Connect the MLX90642 to your Raspberry Pi 3B+ as follows:
- **VDD** (Pin 2) → 3.3V (Pin 1)
- **GND** (Pin 3) → Ground (Pin 6)
- **SDA** (Pin 1) → GPIO 2/SDA (Pin 3)
- **SCL** (Pin 4) → GPIO 3/SCL (Pin 5)

### Enable I2C
Enable I2C on your Raspberry Pi:
```bash
sudo raspi-config
# Navigate to Interfacing Options → I2C → Enable
sudo reboot
```

Install required Python packages:
```bash
sudo apt-get update
sudo apt-get install python3-smbus python3-dev
pip3 install smbus2
```

## Python Implementation

### Step 1: Create the Base MLX90642 Class

```python
import smbus2
import time
import struct
from typing import List, Tuple, Optional

class MLX90642:
    # Constants from MLX90642.h
    DEFAULT_I2C_ADDR = 0x66
    
    # Error codes
    NACK_ERR = 1
    INVAL_VAL_ERR = 2
    TIMEOUT_ERR = 3
    
    # Status values
    YES = 1
    NO = 0
    
    # Memory addresses
    ID1_ADDR = 0x1230
    FW_VER_ADDRESS1 = 0xFFF8
    FW_VER_ADDRESS2 = 0xFFFA
    
    # Data addresses
    AUX_DATA_ADDRESS = 0x2E02
    IR_DATA_ADDRESS = 0x2E2A
    TO_DATA_ADDRESS = 0x342C
    TA_DATA_ADDRESS = 0x3A2C
    PROGRESS_DATA_ADDRESS = 0x3C10
    FLAGS_ADDRESS = 0x3C14
    
    # Configuration addresses
    REFRESH_RATE_ADDRESS = 0x11F0
    EMISSIVITY_ADDRESS = 0x11F2
    APPLICATION_CONFIG_ADDRESS = 0x11F4
    I2C_CONFIG_ADDRESS = 0x11FC
    I2C_SA_ADDRESS = 0x11FE
    REFLECTED_TEMP_ADDRESS = 0xEEEE
    
    # Masks and shifts
    FLAGS_BUSY_MASK = 0x0001
    FLAGS_READY_MASK = 0x0100
    FLAGS_READY_SHIFT = 8
    REFRESH_RATE_MASK = 0x0007
    OUTPUT_FORMAT_MASK = 0x0100
    MEAS_MODE_MASK = 0x0800
    
    # Opcodes and commands
    CONFIG_OPCODE = 0x3A2E
    CMD_OPCODE = 0x0180
    RESET_CMD = 0x0006
    START_SYNC_MEAS_CMD = 0x0001
    SLEEP_CMD = 0x0007
    
    # Timing constants
    RESET_TIME = 10  # ms
    EE_WRITE_TIME = 15  # ms
    POLL_TIME_MS = 2
    MAX_POLL_TRIES = 100
    REF_TIME = 2000
    
    # Configuration values
    CONT_MEAS_MODE = 0
    STEP_MEAS_MODE = 0x0800
    TEMPERATURE_OUTPUT = 0
    NORMALIZED_DATA_OUTPUT = 0x0100
    REF_RATE_2HZ = 2
    REF_RATE_4HZ = 3
    REF_RATE_8HZ = 4
    REF_RATE_16HZ = 5
    REF_RATE_32HZ = 6
    
    # Pixel count
    TOTAL_NUMBER_OF_PIXELS = 768
    TOTAL_NUMBER_OF_AUX = 20
    NUMBER_OF_ID_WORDS = 4
    
    def __init__(self, i2c_bus: int = 1, i2c_addr: int = DEFAULT_I2C_ADDR):
        """Initialize MLX90642 sensor."""
        self.i2c_addr = i2c_addr
        self.bus = smbus2.SMBus(i2c_bus)
    
    def __del__(self):
        """Clean up I2C bus."""
        if hasattr(self, 'bus'):
            self.bus.close()
```

### Step 2: Implement I2C Communication Functions

```python
    def _i2c_read(self, start_address: int, num_words: int) -> List[int]:
        """
        MLX90642 block read I2C command.
        Implements the block read protocol from the datasheet.
        """
        try:
            # Send start address (16-bit, MSB first)
            msg_write = smbus2.i2c_msg.write(self.i2c_addr, 
                                           [start_address >> 8, start_address & 0xFF])
            
            # Read data (2 bytes per word, MSB first)
            msg_read = smbus2.i2c_msg.read(self.i2c_addr, num_words * 2)
            
            self.bus.i2c_rdwr(msg_write, msg_read)
            
            # Convert bytes to 16-bit words
            data = []
            raw_data = list(msg_read)
            for i in range(0, len(raw_data), 2):
                word = (raw_data[i] << 8) | raw_data[i + 1]
                data.append(word)
            
            return data
            
        except Exception as e:
            print(f"I2C read error: {e}")
            return []
    
    def _i2c_config(self, write_address: int, data: int) -> int:
        """
        MLX90642 configuration I2C command.
        Implements configuration protocol from datasheet.
        """
        try:
            # Configuration command format: opcode + address + data
            cmd_data = [
                (self.CONFIG_OPCODE >> 8) & 0xFF,  # MSB of opcode
                self.CONFIG_OPCODE & 0xFF,          # LSB of opcode
                (write_address >> 8) & 0xFF,        # MSB of address
                write_address & 0xFF,               # LSB of address
                (data >> 8) & 0xFF,                 # MSB of data
                data & 0xFF                         # LSB of data
            ]
            
            msg = smbus2.i2c_msg.write(self.i2c_addr, cmd_data)
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C config error: {e}")
            return -1
    
    def _i2c_command(self, command: int) -> int:
        """
        MLX90642 I2C command send.
        For reset, start/sync, and sleep commands.
        """
        try:
            cmd_data = [
                (self.CMD_OPCODE >> 8) & 0xFF,  # MSB of opcode
                self.CMD_OPCODE & 0xFF,          # LSB of opcode
                (command >> 8) & 0xFF,           # MSB of command
                command & 0xFF                   # LSB of command
            ]
            
            msg = smbus2.i2c_msg.write(self.i2c_addr, cmd_data)
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C command error: {e}")
            return -1
    
    def _i2c_wake_up(self) -> int:
        """MLX90642 wake-up command."""
        try:
            # Wake up command is just opcode 0x57
            msg = smbus2.i2c_msg.write(self.i2c_addr, [0x57])
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C wake up error: {e}")
            return -1
    
    def _wait_ms(self, milliseconds: int):
        """Wait for specified milliseconds."""
        time.sleep(milliseconds / 1000.0)
```

### Step 3: Implement Device Information Functions

```python
    def get_device_id(self) -> Optional[List[int]]:
        """Get the device ID (4 words)."""
        return self._i2c_read(self.ID1_ADDR, self.NUMBER_OF_ID_WORDS)
    
    def get_firmware_version(self) -> Optional[Tuple[int, int, int]]:
        """Get firmware version as (major, minor, patch)."""
        data = self._i2c_read(self.FW_VER_ADDRESS1, 2)
        if len(data) != 2:
            return None
        
        # Extract version numbers according to datasheet
        major = (data[0] >> 8) & 0xFF
        minor = data[1] & 0xFF
        patch = (data[1] >> 8) & 0xFF
        
        return (major, minor, patch)
```

### Step 4: Implement Configuration Functions

```python
    def get_refresh_rate(self) -> int:
        """Get the refresh rate setting."""
        data = self._i2c_read(self.REFRESH_RATE_ADDRESS, 1)
        if not data:
            return -1
        
        rate = data[0] & self.REFRESH_RATE_MASK
        return rate if rate >= self.REF_RATE_2HZ else self.REF_RATE_2HZ
    
    def set_refresh_rate(self, rate: int) -> int:
        """Set the refresh rate."""
        if rate < self.REF_RATE_2HZ or rate > self.REF_RATE_32HZ:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.REFRESH_RATE_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.REFRESH_RATE_MASK) | rate
        result = self._i2c_config(self.REFRESH_RATE_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def get_measurement_mode(self) -> int:
        """Get the measurement mode."""
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.MEAS_MODE_MASK
    
    def set_measurement_mode(self, mode: int) -> int:
        """Set the measurement mode."""
        if mode not in [self.CONT_MEAS_MODE, self.STEP_MEAS_MODE]:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.MEAS_MODE_MASK) | mode
        result = self._i2c_config(self.APPLICATION_CONFIG_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def get_output_format(self) -> int:
        """Get the output data format."""
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.OUTPUT_FORMAT_MASK
    
    def set_output_format(self, format_type: int) -> int:
        """Set the output data format."""
        if format_type not in [self.TEMPERATURE_OUTPUT, self.NORMALIZED_DATA_OUTPUT]:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.OUTPUT_FORMAT_MASK) | format_type
        result = self._i2c_config(self.APPLICATION_CONFIG_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def set_emissivity(self, emissivity: int) -> int:
        """Set the emissivity (scaled by 2^14, so 1.0 = 0x4000)."""
        result = self._i2c_config(self.EMISSIVITY_ADDRESS, emissivity & 0xFFFF)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
```

### Step 5: Implement Status and Control Functions

```python
    def is_device_busy(self) -> int:
        """Check if device is busy."""
        data = self._i2c_read(self.FLAGS_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.FLAGS_BUSY_MASK
    
    def is_data_ready(self) -> int:
        """Check if data is ready for read-out."""
        data = self._i2c_read(self.FLAGS_ADDRESS, 1)
        if not data:
            return -1
        return (data[0] & self.FLAGS_READY_MASK) >> self.FLAGS_READY_SHIFT
    
    def clear_data_ready(self) -> int:
        """Clear the data ready flag."""
        # Reading from TO_DATA_ADDRESS clears the flag
        data = self._i2c_read(self.TO_DATA_ADDRESS, 1)
        if not data:
            return -1
        return self.is_data_ready()
    
    def is_read_window_open(self) -> int:
        """Check if read window is open for consistent frame data."""
        ready_status = self.is_data_ready()
        if ready_status != self.YES:
            return ready_status
        
        busy_status = self.is_device_busy()
        if busy_status < 0:
            return busy_status
        
        return self.YES if busy_status == self.NO else self.NO
    
    def get_progress(self) -> int:
        """Get measurement progress (0-100%)."""
        data = self._i2c_read(self.PROGRESS_DATA_ADDRESS, 1)
        return data[0] if data else -1
    
    def start_sync_measurement(self) -> int:
        """Start/sync a new measurement."""
        return self._i2c_command(self.START_SYNC_MEAS_CMD)
    
    def goto_sleep(self) -> int:
        """Put device in sleep mode."""
        return self._i2c_command(self.SLEEP_CMD)
```

### Step 6: Implement Data Reading Functions

```python
    def get_refresh_time_ms(self) -> int:
        """Calculate expected refresh time based on refresh rate setting."""
        rate = self.get_refresh_rate()
        if rate < 0:
            return rate
        
        return self.REF_TIME >> rate
    
    def get_image_data(self) -> Optional[List[int]]:
        """Get the calculated temperature/normalized image data."""
        data = self._i2c_read(self.TO_DATA_ADDRESS, self.TOTAL_NUMBER_OF_PIXELS)
        return data if len(data) == self.TOTAL_NUMBER_OF_PIXELS else None
    
    def get_raw_ir_data(self) -> Optional[List[int]]:
        """Get the raw IR data."""
        data = self._i2c_read(self.IR_DATA_ADDRESS, self.TOTAL_NUMBER_OF_PIXELS)
        return data if len(data) == self.TOTAL_NUMBER_OF_PIXELS else None
    
    def get_aux_data(self) -> Optional[List[int]]:
        """Get auxiliary data."""
        data = self._i2c_read(self.AUX_DATA_ADDRESS, self.TOTAL_NUMBER_OF_AUX)
        return data if len(data) == self.TOTAL_NUMBER_OF_AUX else None
    
    def get_sensor_temperature(self) -> Optional[float]:
        """Get sensor temperature in Celsius."""
        data = self._i2c_read(self.TA_DATA_ADDRESS, 1)
        if not data:
            return None
        
        # Convert according to datasheet: value / 100
        return data[0] / 100.0
    
    def initialize(self) -> int:
        """Initialize the device and wait for first valid data."""
        ref_time = self.get_refresh_time_ms()
        if ref_time < 0:
            return ref_time
        
        # Clear any existing data ready flag
        if self.clear_data_ready() != 0:
            return -self.INVAL_VAL_ERR
        
        # Start a new measurement
        if self.start_sync_measurement() < 0:
            return -1
        
        # Wait for expected measurement time
        self._wait_ms(ref_time)
        
        # Poll for data ready
        for _ in range(self.MAX_POLL_TRIES):
            self._wait_ms(self.POLL_TIME_MS)
            status = self.is_data_ready()
            if status < 0:
                return status
            if status == self.YES:
                return 0
        
        return -self.TIMEOUT_ERR
    
    def measure_now(self) -> Optional[List[int]]:
        """Take a single measurement and return the image data."""
        ref_time = self.get_refresh_time_ms()
        if ref_time < 0:
            return None
        
        # Clear data ready flag
        if self.clear_data_ready() != 0:
            return None
        
        # Start measurement
        if self.start_sync_measurement() < 0:
            return None
        
        # Wait for measurement
        self._wait_ms(ref_time)
        
        # Poll for completion
        for _ in range(self.MAX_POLL_TRIES):
            self._wait_ms(self.POLL_TIME_MS)
            if self.is_data_ready() == self.YES:
                return self.get_image_data()
        
        return None
```

### Step 7: Convert Temperature Data

```python
    def convert_temperature_data(self, raw_data: List[int]) -> List[float]:
        """
        Convert raw temperature data to Celsius.
        According to datasheet: Temperature = value / 50.0
        """
        temperatures = []
        for value in raw_data:
            # Handle two's complement for negative values
            if value > 32767:
                value = value - 65536
            temp_celsius = value / 50.0
            temperatures.append(temp_celsius)
        
        return temperatures
    
    def get_pixel_array(self, data: List[int]) -> List[List[float]]:
        """Convert linear data to 24x32 pixel array (24 rows, 32 columns)."""
        if len(data) != self.TOTAL_NUMBER_OF_PIXELS:
            return []
        
        pixel_array = []
        for row in range(24):
            pixel_row = []
            for col in range(32):
                index = row * 32 + col
                pixel_row.append(data[index])
            pixel_array.append(pixel_row)
        
        return pixel_array
```

## Usage Examples

### Example 1: Basic Device Information

```python
# Create sensor instance
sensor = MLX90642()

# Get device information
device_id = sensor.get_device_id()
print(f"Device ID: {device_id}")

fw_version = sensor.get_firmware_version()
if fw_version:
    print(f"Firmware version: {fw_version[0]}.{fw_version[1]}.{fw_version[2]}")

# Check current configuration
refresh_rate = sensor.get_refresh_rate()
print(f"Refresh rate: {refresh_rate}Hz")

measurement_mode = sensor.get_measurement_mode()
mode_str = "Continuous" if measurement_mode == sensor.CONT_MEAS_MODE else "Step"
print(f"Measurement mode: {mode_str}")
```

### Example 2: Configure and Initialize Sensor

```python
# Create sensor instance
sensor = MLX90642()

# Configure sensor
sensor.set_measurement_mode(sensor.CONT_MEAS_MODE)
sensor.set_output_format(sensor.TEMPERATURE_OUTPUT)
sensor.set_refresh_rate(sensor.REF_RATE_4HZ)
sensor.set_emissivity(0x4000)  # Emissivity = 1.0

# Initialize sensor
if sensor.initialize() == 0:
    print("Sensor initialized successfully")
else:
    print("Sensor initialization failed")
```

### Example 3: Continuous Temperature Measurement

```python
import numpy as np

def continuous_measurement_loop():
    sensor = MLX90642()
    
    # Configure for continuous temperature measurement
    sensor.set_measurement_mode(sensor.CONT_MEAS_MODE)
    sensor.set_output_format(sensor.TEMPERATURE_OUTPUT)
    sensor.set_refresh_rate(sensor.REF_RATE_4HZ)
    
    # Initialize
    if sensor.initialize() != 0:
        print("Failed to initialize sensor")
        return
    
    try:
        while True:
            # Wait for read window to open
            while sensor.is_read_window_open() != sensor.YES:
                time.sleep(0.01)  # 10ms delay
            
            # Read image data
            raw_data = sensor.get_image_data()
            if raw_data:
                # Convert to temperatures
                temperatures = sensor.convert_temperature_data(raw_data)
                
                # Convert to 24x32 array
                temp_array = sensor.get_pixel_array(temperatures)
                
                # Calculate statistics
                temp_array_np = np.array(temp_array)
                min_temp = np.min(temp_array_np)
                max_temp = np.max(temp_array_np)
                avg_temp = np.mean(temp_array_np)
                
                print(f"Temperature range: {min_temp:.2f}°C to {max_temp:.2f}°C, Average: {avg_temp:.2f}°C")
                
                # You can process the temp_array here
                # For example, save to file, display, etc.
            
            time.sleep(0.1)  # Small delay between readings
            
    except KeyboardInterrupt:
        print("Measurement stopped by user")
    finally:
        sensor.goto_sleep()

# Run continuous measurement
continuous_measurement_loop()
```

### Example 4: Step Mode Single Measurements

```python
def step_mode_measurement():
    sensor = MLX90642()
    
    # Configure for step mode
    sensor.set_measurement_mode(sensor.STEP_MEAS_MODE)
    sensor.set_output_format(sensor.TEMPERATURE_OUTPUT)
    sensor.set_refresh_rate(sensor.REF_RATE_8HZ)
    
    # Initialize
    if sensor.initialize() != 0:
        print("Failed to initialize sensor")
        return
    
    # Take 10 measurements
    for i in range(10):
        print(f"Taking measurement {i+1}...")
        
        # Single measurement
        raw_data = sensor.measure_now()
        if raw_data:
            temperatures = sensor.convert_temperature_data(raw_data)
            temp_array = np.array(sensor.get_pixel_array(temperatures))
            
            center_temp = temp_array[12, 16]  # Center pixel
            print(f"Center pixel temperature: {center_temp:.2f}°C")
        
        time.sleep(1)  # Wait 1 second between measurements
    
    # Put sensor to sleep
    sensor.goto_sleep()

# Run step mode measurement
step_mode_measurement()
```

### Example 5: Save Temperature Data to File

```python
import csv
from datetime import datetime

def save_temperature_data():
    sensor = MLX90642()
    
    # Configure sensor
    sensor.set_measurement_mode(sensor.CONT_MEAS_MODE)
    sensor.set_output_format(sensor.TEMPERATURE_OUTPUT)
    sensor.set_refresh_rate(sensor.REF_RATE_2HZ)
    
    if sensor.initialize() != 0:
        print("Failed to initialize sensor")
        return
    
    # Create CSV file with timestamp
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"mlx90642_data_{timestamp}.csv"
    
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        
        # Write header
        header = ['timestamp', 'sensor_temp'] + [f'pixel_{i}' for i in range(768)]
        writer.writerow(header)
        
        try:
            for measurement in range(60):  # Take 60 measurements
                # Wait for data
                while sensor.is_read_window_open() != sensor.YES:
                    time.sleep(0.01)
                
                # Read data
                raw_data = sensor.get_image_data()
                sensor_temp = sensor.get_sensor_temperature()
                
                if raw_data and sensor_temp is not None:
                    temperatures = sensor.convert_temperature_data(raw_data)
                    
                    # Write to CSV
                    row = [datetime.now().isoformat(), sensor_temp] + temperatures
                    writer.writerow(row)
                    
                    print(f"Saved measurement {measurement + 1}/60")
                
                time.sleep(0.5)  # 0.5 second between measurements
                
        except KeyboardInterrupt:
            print("Data collection stopped by user")
        
    print(f"Data saved to {filename}")
    sensor.goto_sleep()

# Run data collection
save_temperature_data()
```

## Troubleshooting

### Common Issues:

1. **I2C Communication Errors**: Check wiring and ensure I2C is enabled on Raspberry Pi
2. **Device Not Responding**: Verify power supply (3.3V) and try different I2C address
3. **Timeout Errors**: Increase polling time or check refresh rate settings
4. **Invalid Temperature Readings**: Check emissivity settings and calibration

### Debug Functions:

```python
def debug_sensor_status(sensor):
    """Print comprehensive sensor status."""
    print("=== MLX90642 Status ===")
    
    # Device info
    device_id = sensor.get_device_id()
    print(f"Device ID: {device_id}")
    
    fw_version = sensor.get_firmware_version()
    if fw_version:
        print(f"Firmware: {fw_version[0]}.{fw_version[1]}.{fw_version[2]}")
    
    # Configuration
    print(f"Refresh rate: {sensor.get_refresh_rate()}")
    print(f"Measurement mode: {sensor.get_measurement_mode()}")
    print(f"Output format: {sensor.get_output_format()}")
    
    # Status
    print(f"Device busy: {sensor.is_device_busy()}")
    print(f"Data ready: {sensor.is_data_ready()}")
    print(f"Progress: {sensor.get_progress()}%")
    
    # Sensor temperature
    sensor_temp = sensor.get_sensor_temperature()
    if sensor_temp:
        print(f"Sensor temperature: {sensor_temp:.2f}°C")

# Usage
sensor = MLX90642()
debug_sensor_status(sensor)
```

This implementation provides a complete Python interface for the MLX90642 sensor, following the communication protocols and functionality described in the datasheet and C driver library.



```python
import smbus2
import time
import struct
from typing import List, Tuple, Optional

class MLX90642:
    # Constants from MLX90642.h
    DEFAULT_I2C_ADDR = 0x66
    
    # Error codes
    NACK_ERR = 1
    INVAL_VAL_ERR = 2
    TIMEOUT_ERR = 3
    
    # Status values
    YES = 1
    NO = 0
    
    # Memory addresses
    ID1_ADDR = 0x1230
    FW_VER_ADDRESS1 = 0xFFF8
    FW_VER_ADDRESS2 = 0xFFFA
    
    # Data addresses
    AUX_DATA_ADDRESS = 0x2E02
    IR_DATA_ADDRESS = 0x2E2A
    TO_DATA_ADDRESS = 0x342C
    TA_DATA_ADDRESS = 0x3A2C
    PROGRESS_DATA_ADDRESS = 0x3C10
    FLAGS_ADDRESS = 0x3C14
    
    # Configuration addresses
    REFRESH_RATE_ADDRESS = 0x11F0
    EMISSIVITY_ADDRESS = 0x11F2
    APPLICATION_CONFIG_ADDRESS = 0x11F4
    I2C_CONFIG_ADDRESS = 0x11FC
    I2C_SA_ADDRESS = 0x11FE
    REFLECTED_TEMP_ADDRESS = 0xEEEE
    
    # Masks and shifts
    FLAGS_BUSY_MASK = 0x0001
    FLAGS_READY_MASK = 0x0100
    FLAGS_READY_SHIFT = 8
    REFRESH_RATE_MASK = 0x0007
    OUTPUT_FORMAT_MASK = 0x0100
    MEAS_MODE_MASK = 0x0800
    
    # Opcodes and commands
    CONFIG_OPCODE = 0x3A2E
    CMD_OPCODE = 0x0180
    RESET_CMD = 0x0006
    START_SYNC_MEAS_CMD = 0x0001
    SLEEP_CMD = 0x0007
    
    # Timing constants
    RESET_TIME = 10  # ms
    EE_WRITE_TIME = 15  # ms
    POLL_TIME_MS = 2
    MAX_POLL_TRIES = 100
    REF_TIME = 2000
    
    # Configuration values
    CONT_MEAS_MODE = 0
    STEP_MEAS_MODE = 0x0800
    TEMPERATURE_OUTPUT = 0
    NORMALIZED_DATA_OUTPUT = 0x0100
    REF_RATE_2HZ = 2
    REF_RATE_4HZ = 3
    REF_RATE_8HZ = 4
    REF_RATE_16HZ = 5
    REF_RATE_32HZ = 6
    
    # Pixel count
    TOTAL_NUMBER_OF_PIXELS = 768
    TOTAL_NUMBER_OF_AUX = 20
    NUMBER_OF_ID_WORDS = 4
    
    def __init__(self, i2c_bus: int = 1, i2c_addr: int = DEFAULT_I2C_ADDR):
        """Initialize MLX90642 sensor."""
        self.i2c_addr = i2c_addr
        self.bus = smbus2.SMBus(i2c_bus)
    
    def __del__(self):
        """Clean up I2C bus."""
        if hasattr(self, 'bus'):
            self.bus.close()
    def _i2c_read(self, start_address: int, num_words: int) -> List[int]:
        """
        MLX90642 block read I2C command.
        Implements the block read protocol from the datasheet.
        """
        try:
            # Send start address (16-bit, MSB first)
            msg_write = smbus2.i2c_msg.write(self.i2c_addr, 
                                           [start_address >> 8, start_address & 0xFF])
            
            # Read data (2 bytes per word, MSB first)
            msg_read = smbus2.i2c_msg.read(self.i2c_addr, num_words * 2)
            
            self.bus.i2c_rdwr(msg_write, msg_read)
            
            # Convert bytes to 16-bit words
            data = []
            raw_data = list(msg_read)
            for i in range(0, len(raw_data), 2):
                word = (raw_data[i] << 8) | raw_data[i + 1]
                data.append(word)
            
            return data
            
        except Exception as e:
            print(f"I2C read error: {e}")
            return []
    
    def _i2c_config(self, write_address: int, data: int) -> int:
        """
        MLX90642 configuration I2C command.
        Implements configuration protocol from datasheet.
        """
        try:
            # Configuration command format: opcode + address + data
            cmd_data = [
                (self.CONFIG_OPCODE >> 8) & 0xFF,  # MSB of opcode
                self.CONFIG_OPCODE & 0xFF,          # LSB of opcode
                (write_address >> 8) & 0xFF,        # MSB of address
                write_address & 0xFF,               # LSB of address
                (data >> 8) & 0xFF,                 # MSB of data
                data & 0xFF                         # LSB of data
            ]
            
            msg = smbus2.i2c_msg.write(self.i2c_addr, cmd_data)
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C config error: {e}")
            return -1
    
    def _i2c_command(self, command: int) -> int:
        """
        MLX90642 I2C command send.
        For reset, start/sync, and sleep commands.
        """
        try:
            cmd_data = [
                (self.CMD_OPCODE >> 8) & 0xFF,  # MSB of opcode
                self.CMD_OPCODE & 0xFF,          # LSB of opcode
                (command >> 8) & 0xFF,           # MSB of command
                command & 0xFF                   # LSB of command
            ]
            
            msg = smbus2.i2c_msg.write(self.i2c_addr, cmd_data)
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C command error: {e}")
            return -1
    
    def _i2c_wake_up(self) -> int:
        """MLX90642 wake-up command."""
        try:
            # Wake up command is just opcode 0x57
            msg = smbus2.i2c_msg.write(self.i2c_addr, [0x57])
            self.bus.i2c_rdwr(msg)
            return 0
            
        except Exception as e:
            print(f"I2C wake up error: {e}")
            return -1
    
    def _wait_ms(self, milliseconds: int):
        """Wait for specified milliseconds."""
        time.sleep(milliseconds / 1000.0)
        
    def get_device_id(self) -> Optional[List[int]]:
        """Get the device ID (4 words)."""
        return self._i2c_read(self.ID1_ADDR, self.NUMBER_OF_ID_WORDS)
    
    def get_firmware_version(self) -> Optional[Tuple[int, int, int]]:
        """Get firmware version as (major, minor, patch)."""
        data = self._i2c_read(self.FW_VER_ADDRESS1, 2)
        if len(data) != 2:
            return None
        
        # Extract version numbers according to datasheet
        major = (data[0] >> 8) & 0xFF
        minor = data[1] & 0xFF
        patch = (data[1] >> 8) & 0xFF
        
        return (major, minor, patch)
        
    def get_refresh_rate(self) -> int:
        """Get the refresh rate setting."""
        data = self._i2c_read(self.REFRESH_RATE_ADDRESS, 1)
        if not data:
            return -1
        
        rate = data[0] & self.REFRESH_RATE_MASK
        return rate if rate >= self.REF_RATE_2HZ else self.REF_RATE_2HZ
    
    def set_refresh_rate(self, rate: int) -> int:
        """Set the refresh rate."""
        if rate < self.REF_RATE_2HZ or rate > self.REF_RATE_32HZ:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.REFRESH_RATE_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.REFRESH_RATE_MASK) | rate
        result = self._i2c_config(self.REFRESH_RATE_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def get_measurement_mode(self) -> int:
        """Get the measurement mode."""
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.MEAS_MODE_MASK
    
    def set_measurement_mode(self, mode: int) -> int:
        """Set the measurement mode."""
        if mode not in [self.CONT_MEAS_MODE, self.STEP_MEAS_MODE]:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.MEAS_MODE_MASK) | mode
        result = self._i2c_config(self.APPLICATION_CONFIG_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def get_output_format(self) -> int:
        """Get the output data format."""
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.OUTPUT_FORMAT_MASK
    
    def set_output_format(self, format_type: int) -> int:
        """Set the output data format."""
        if format_type not in [self.TEMPERATURE_OUTPUT, self.NORMALIZED_DATA_OUTPUT]:
            return -self.INVAL_VAL_ERR
        
        data = self._i2c_read(self.APPLICATION_CONFIG_ADDRESS, 1)
        if not data:
            return -1
        
        new_data = (data[0] & ~self.OUTPUT_FORMAT_MASK) | format_type
        result = self._i2c_config(self.APPLICATION_CONFIG_ADDRESS, new_data)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
    def set_emissivity(self, emissivity: int) -> int:
        """Set the emissivity (scaled by 2^14, so 1.0 = 0x4000)."""
        result = self._i2c_config(self.EMISSIVITY_ADDRESS, emissivity & 0xFFFF)
        self._wait_ms(self.EE_WRITE_TIME)
        return result
    
	def is_device_busy(self) -> int:
        """Check if device is busy."""
        data = self._i2c_read(self.FLAGS_ADDRESS, 1)
        if not data:
            return -1
        return data[0] & self.FLAGS_BUSY_MASK
    
    def is_data_ready(self) -> int:
        """Check if data is ready for read-out."""
        data = self._i2c_read(self.FLAGS_ADDRESS, 1)
        if not data:
            return -1
        return (data[0] & self.FLAGS_READY_MASK) >> self.FLAGS_READY_SHIFT
    
    def clear_data_ready(self) -> int:
        """Clear the data ready flag."""
        # Reading from TO_DATA_ADDRESS clears the flag
        data = self._i2c_read(self.TO_DATA_ADDRESS, 1)
        if not data:
            return -1
        return self.is_data_ready()
    
    def is_read_window_open(self) -> int:
        """Check if read window is open for consistent frame data."""
        ready_status = self.is_data_ready()
        if ready_status != self.YES:
            return ready_status
        
        busy_status = self.is_device_busy()
        if busy_status < 0:
            return busy_status
        
        return self.YES if busy_status == self.NO else self.NO
    
    def get_progress(self) -> int:
        """Get measurement progress (0-100%)."""
        data = self._i2c_read(self.PROGRESS_DATA_ADDRESS, 1)
        return data[0] if data else -1
    
    def start_sync_measurement(self) -> int:
        """Start/sync a new measurement."""
        return self._i2c_command(self.START_SYNC_MEAS_CMD)
    
    def goto_sleep(self) -> int:
        """Put device in sleep mode."""
        return self._i2c_command(self.SLEEP_CMD)         
    
    def get_refresh_time_ms(self) -> int:
        """Calculate expected refresh time based on refresh rate setting."""
        rate = self.get_refresh_rate()
        if rate < 0:
            return rate
        
        return self.REF_TIME >> rate
    
    def get_image_data(self) -> Optional[List[int]]:
        """Get the calculated temperature/normalized image data."""
        data = self._i2c_read(self.TO_DATA_ADDRESS, self.TOTAL_NUMBER_OF_PIXELS)
        return data if len(data) == self.TOTAL_NUMBER_OF_PIXELS else None
    
    def get_raw_ir_data(self) -> Optional[List[int]]:
        """Get the raw IR data."""
        data = self._i2c_read(self.IR_DATA_ADDRESS, self.TOTAL_NUMBER_OF_PIXELS)
        return data if len(data) == self.TOTAL_NUMBER_OF_PIXELS else None
    
    def get_aux_data(self) -> Optional[List[int]]:
        """Get auxiliary data."""
        data = self._i2c_read(self.AUX_DATA_ADDRESS, self.TOTAL_NUMBER_OF_AUX)
        return data if len(data) == self.TOTAL_NUMBER_OF_AUX else None
    
    def get_sensor_temperature(self) -> Optional[float]:
        """Get sensor temperature in Celsius."""
        data = self._i2c_read(self.TA_DATA_ADDRESS, 1)
        if not data:
            return None
        
        # Convert according to datasheet: value / 100
        return data[0] / 100.0
    
    def initialize(self) -> int:
        """Initialize the device and wait for first valid data."""
        ref_time = self.get_refresh_time_ms()
        if ref_time < 0:
            return ref_time
        
        # Clear any existing data ready flag
        if self.clear_data_ready() != 0:
            return -self.INVAL_VAL_ERR
        
        # Start a new measurement
        if self.start_sync_measurement() < 0:
            return -1
        
        # Wait for expected measurement time
        self._wait_ms(ref_time)
        
        # Poll for data ready
        for _ in range(self.MAX_POLL_TRIES):
            self._wait_ms(self.POLL_TIME_MS)
            status = self.is_data_ready()
            if status < 0:
                return status
            if status == self.YES:
                return 0
        
        return -self.TIMEOUT_ERR
    
    def measure_now(self) -> Optional[List[int]]:
        """Take a single measurement and return the image data."""
        ref_time = self.get_refresh_time_ms()
        if ref_time < 0:
            return None
        
        # Clear data ready flag
        if self.clear_data_ready() != 0:
            return None
        
        # Start measurement
        if self.start_sync_measurement() < 0:
            return None
        
        # Wait for measurement
        self._wait_ms(ref_time)
        
        # Poll for completion
        for _ in range(self.MAX_POLL_TRIES):
            self._wait_ms(self.POLL_TIME_MS)
            if self.is_data_ready() == self.YES:
                return self.get_image_data()
        
        return None
	
	def convert_temperature_data(self, raw_data: List[int]) -> List[float]:
        """
        Convert raw temperature data to Celsius.
        According to datasheet: Temperature = value / 50.0
        """
        temperatures = []
        for value in raw_data:
            # Handle two's complement for negative values
            if value > 32767:
                value = value - 65536
            temp_celsius = value / 50.0
            temperatures.append(temp_celsius)
        
        return temperatures
    
    def get_pixel_array(self, data: List[int]) -> List[List[float]]:
        """Convert linear data to 24x32 pixel array (24 rows, 32 columns)."""
        if len(data) != self.TOTAL_NUMBER_OF_PIXELS:
            return []
        
        pixel_array = []
        for row in range(24):
            pixel_row = []
            for col in range(32):
                index = row * 32 + col
                pixel_row.append(data[index])
            pixel_array.append(pixel_row)
        
        return pixel_array



```
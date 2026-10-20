---
title: n_sensor_node
date: 2026-09-18
---

# n_sensor_node

## Cara Cek I2C Sensor

1. ```bash
   sudo apt update
   ```
2. ```bash
   sudo apt install i2c-tools -y
   ```
3. ```bash
   i2cdetect -y 1
   ```
   Hasilnya tinggal ditambah `0x..` untuk jadi alamat lengkap, contoh: `76` → `0x76`, `29` → `0x29`.

## Sensor MS5803

Sensor ini punya beberapa tipe dan berbeda-beda cara pembacaannya:

- MS5803-01BA ← yang dipakai sekarang
- MS5803-02BA
- MS5803-05BA
- MS5803-14BA
- MS5837

### Baca PROM

PROM isinya data mentah dari sensor berupa Raw pressure (D1), Raw temperature (D2), Calibration Coefficient (C1–C6).

```python
from smbus2 import SMBus

ADDR = 0x76
bus = SMBus(1)

coeff = []
for i in range(1, 7):
    reg = 0xA0 + (i * 2)
    data = bus.read_i2c_block_data(ADDR, reg, 2)
    value = (data[0] << 8) | data[1]
    coeff.append(value)
    print(f"C{i}: {value}")
```

### Full Program MS5803

```python
#!/usr/bin/env python3
from smbus2 import SMBus
import time

ADDR = 0x76
bus = SMBus(1)

# ========================
# Reset Sensor
# ========================
bus.write_byte(ADDR, 0x1E)
time.sleep(0.1)
print("Sensor reset OK")

# ========================
# Read PROM Coefficients
# ========================
C = [0] * 7
for i in range(1, 7):
    reg = 0xA0 + (i * 2)
    data = bus.read_i2c_block_data(ADDR, reg, 2)
    C[i] = (data[0] << 8) | data[1]

C1, C2, C3, C4, C5, C6 = C[1], C[2], C[3], C[4], C[5], C[6]
print("Calibration Data")
print(f"C1 = {C1}")
print(f"C2 = {C2}")
print(f"C3 = {C3}")
print(f"C4 = {C4}")
print(f"C5 = {C5}")
print(f"C6 = {C6}")

# ========================
# Read D1 (Raw Pressure)
# ========================
def read_pressure_raw():
    bus.write_byte(ADDR, 0x48)  # OSR 4096
    time.sleep(0.05)
    data = bus.read_i2c_block_data(ADDR, 0x00, 3)
    D1 = (data[0] << 16) | (data[1] << 8) | data[2]
    return D1

# ========================
# Read D2 (Raw Temperature)
# ========================
def read_temperature_raw():
    bus.write_byte(ADDR, 0x58)  # OSR 4096
    time.sleep(0.05)
    data = bus.read_i2c_block_data(ADDR, 0x00, 3)
    D2 = (data[0] << 16) | (data[1] << 8) | data[2]
    return D2

# ========================
# Main Loop
# ========================
while True:
    D1 = read_pressure_raw()
    D2 = read_temperature_raw()

    dT = D2 - (C5 * 256)
    TEMP = 2000 + (dT * C6) / 8388608
    OFF = (C2 * 65536 + (C4 * dT) / 128)
    SENS = (C1 * 32768 + (C3 * dT) / 256)
    P = ((D1 * SENS / 2097152) - OFF) / 32768

    temperature_c = TEMP / 100.0
    pressure_mbar = P / 100.0
    depth_m = (pressure_mbar - 1013.25) / 100.0

    print("=" * 50)
    print(f"D1 Raw : {D1}")
    print(f"D2 Raw : {D2}")
    print(f"Temperature : {temperature_c:.2f} C")
    print(f"Pressure : {pressure_mbar:.2f} mbar")
    print(f"Depth : {depth_m:.2f} m")
    time.sleep(1)
```

## Sensor IMU BNO055

Problem: sensor IMU tidak bisa baca I2C karena harus disetting dulu output sensor I2C bukan UART. Caranya: solder pin S0 dan S1 di board (tabel settingnya ada di PDF asli).

Ketika BNO055 diconnect dengan sensor MS5803 pada power rail 3,3V yang sama, BNO055 tidak terbaca. Lalu dicoba pindahkan BNO055 ke power rail 5V dan kedua sensor akhirnya terbaca.

### The Trial

Ada 2 cara untuk melakukan testing sensor: langsung di environment utama atau lewat virtual environment.

1. Environment virtual:
   ```bash
   mkdir -p ~/python_env
   cd ~/python_env
   python3 -m venv bno_env
   source bno_env/bin/activate   # untuk nonaktifin, tinggal ketik: deactivate
   ```

### Cek I2C

1. Cek alamat I2C:
   ```bash
   i2cdetect -y 1
   ```
2. Baca register chip:
   ```bash
   i2cget -y 1 0x29 0x00   # harus 0xa0
   ```

### Set NDOF

Kalau data tidak berubah, kemungkinan sensor pada mode CONFIGMODE — ubah ke NDOF:

```python
import time
from smbus2 import SMBus

bus = SMBus(1)
ADDR = 0x29

# OPR_MODE register
bus.write_byte_data(ADDR, 0x3D, 0x00)  # CONFIGMODE
time.sleep(0.1)

# NDOF mode
bus.write_byte_data(ADDR, 0x3D, 0x0C)
time.sleep(1)
print("NDOF Mode Set")
```

### Coba Sensor BNO055

```python
from smbus2 import SMBus
import time

ADDRESS = 0x29
bus = SMBus(1)

def read_signed_16(reg):
    lsb = bus.read_byte_data(ADDRESS, reg)
    msb = bus.read_byte_data(ADDRESS, reg + 1)
    value = (msb << 8) | lsb
    if value > 32767:
        value -= 65536
    return value

while True:
    yaw = read_signed_16(0x1A) / 16.0
    roll = read_signed_16(0x1C) / 16.0
    pitch = read_signed_16(0x1E) / 16.0
    accel_x = read_signed_16(0x08) / 100.0
    accel_y = read_signed_16(0x0A) / 100.0
    accel_z = read_signed_16(0x0C) / 100.0
    gyro_x = read_signed_16(0x14) / 16.0
    gyro_y = read_signed_16(0x16) / 16.0
    gyro_z = read_signed_16(0x18) / 16.0

    print("=" * 50)
    print(f"Yaw : {yaw:.2f}")
    print(f"Roll : {roll:.2f}")
    print(f"Pitch : {pitch:.2f}")
    print()
    print(f"Accel X : {accel_x:.2f}")
    print(f"Accel Y : {accel_y:.2f}")
    print(f"Accel Z : {accel_z:.2f}")
    print()
    print(f"Gyro X : {gyro_x:.2f}")
    print(f"Gyro Y : {gyro_y:.2f}")
    print(f"Gyro Z : {gyro_z:.2f}")
    time.sleep(1)
```

## Terkait

- [[Project_Overview]]
- [[24_25_September]]
- [[5_October_2026]]

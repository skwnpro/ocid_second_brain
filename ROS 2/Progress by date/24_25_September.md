---
title: 24–25 September 2026
date: 2026-09-25
---

# 24–25 September 2026

## Resume Progress Program Sensor Node (24-09-2026)

### Verifikasi Interface

```bash
ros2 interface show brushkart_msgs/msg/SensorData
```

### 1. Arsitektur Topic Saat Ini

| Node | Publish | Subscribe | Status |
|------|---------|-----------|--------|
| joystick_interface_node | /robot_command | — | ✅ Selesai |
| command_gateway_node | — | /robot_command, /emergency_stop | Basic |
| sensor_node | /imu/data, /pressure | — | ✅ Selesai |
| robot_state_node | /robot_state | /imu/data, /pressure, /drive_encoder, /network_status | ⬜ Belum |
| camera_node | /camera/image_raw | — | ⬜ Belum |
| network_monitor_node | /network_status | — | ⬜ Belum |
| telemetri_gui_node | — | /robot_state, /camera/image_raw | ⬜ Belum |

### Perbedaan Interface di Ground Station dan Underwater Robot

**1. Ground Station — package.xml:**
```xml
<buildtool_depend>ament_cmake</buildtool_depend>
<build_depend>rosidl_default_generators</build_depend>
<depend>std_msgs</depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

**2. Ground Station — CMakeLists.txt:**
```cmake
find_package(ament_cmake REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/OperatorCommand.msg"
  "msg/SensorData.msg"
  DEPENDENCIES std_msgs
)

ament_export_dependencies(rosidl_default_runtime)
ament_package()
```

### 2. Message Interface

| Message | Status | Keterangan |
|---------|--------|------------|
| OperatorCommand.msg | ✅ | Command dari joystick ke robot |
| SensorData.msg | ✅ | Dipakai untuk /imu/data dan /pressure |
| RobotState.msg | ⬜ | Belum dibuat |
| DriveEncoder.msg | ⬜ | Belum dibuat |
| NetworkStatus.msg | ⬜ | Belum dibuat |

### 3. Hardware Sensor

| Sensor | Alamat I2C | Status |
|--------|------------|--------|
| BNO055 | 0x29 | ✅ Terdeteksi |
| MS5803-01BA | 0x76 | ✅ Terdeteksi |

### Verifikasi I2C Sudah Terdaftar untuk User atau Belum

1. Cek dulu:
   ```bash
   groups
   ```
2. Daftarin:
   ```bash
   sudo usermod -aG i2c exedy
   sudo reboot
   ```

### 4. Masalah yang Dihadapi

| No | Masalah | Penyebab | Solusi | Status |
|----|---------|----------|--------|--------|
| 1 | BNO055 tidak muncul di i2cdetect | Modul masih bukan mode I2C | S0 → GND, S1 → GND | ✅ |
| 2 | Hanya MS5803 muncul (0x76) | BNO055 belum mode I2C | Konfigurasi PS0 & PS1 | ✅ |
| 3 | BNO055 hilang saat MS5803 dipasang | Supply drop sampai 1.4V | Pisahkan supply sensor | ✅ |
| 4 | Adafruit BNO055 library gagal penuh | Clone WCMCU-055 tidak kompatibel | Pakai smbus2 langsung | ✅ |
| 5 | Build brushkart_msgs gagal | Dependencies salah penulisan | Ubah menjadi DEPENDENCIES | ✅ |
| 6 | Git author identity unknown | Git belum dikonfigurasi | Set user.name & user.email | ✅ |
| 7 | Failed to push some refs | History repo berbeda | git pull --rebase | ✅ |
| 8 | No module named smbus2 | Library belum terinstall | Install smbus2 | ✅ |
| 9 | No module named brushkart_msgs | Dependency belum dibuild | Build ulang workspace | ✅ |
| 10 | TabError | Campur TAB dan spasi | Rapikan indentasi | ✅ |
| 11 | Driver crash saat import | File test dipakai jadi driver | Refactor ke class | ✅ |

### 5. Milestone BNO055

| Item | Status |
|------|--------|
| I2C detect | ✅ |
| Chip ID (0xA0) terbaca | ✅ |
| NDOF mode | ✅ |
| Roll | ✅ |
| Pitch | ✅ |
| Yaw | ✅ |
| Accelerometer X/Y/Z | ✅ |
| Gyroscope X/Y/Z | ✅ |
| Driver class | ✅ |

### 6. Milestone MS5803

| Item | Status |
|------|--------|
| I2C detect | ✅ |
| Read PROM | ✅ |
| C1–C6 calibration | ✅ |
| Read D1 (pressure raw) | ✅ |
| Read D2 (temperature raw) | ✅ |
| Temperatur calculation | ✅ |
| Pressure calculation | ✅ |
| Depth calculation | ✅ |
| Driver class | ✅ |

### 7. Software yang Sudah Dibuat

| Komponen | Status |
|----------|--------|
| brushkart_msgs package | ✅ |
| OperatorCommand.msg | ✅ |
| SensorData.msg | ✅ |
| brushkart_sensor package | ✅ |
| bno055_driver.py | ✅ |
| ms5803_driver.py | ✅ |
| sensor_node.py | ✅ |
| Publish /imu/data | ✅ |
| Publish /pressure | ✅ |
| ros2 topic echo berhasil | ✅ |

### 8. Hasil Topic yang Sudah Jalan

| Topic | Isi Data | Status |
|-------|----------|--------|
| /imu/data | Roll, pitch, yaw, accel, gyro, imu_ok | ✅ |
| /pressure | Pressure, temperature, depth, pressure_ok | ✅ |

### Underwater Robot

| Item | Status |
|------|--------|
| ROS2 Jazzy | ✅ |
| Ubuntu Server RPi | ✅ |
| Ethernet communication | ✅ |
| BNO055 driver | ✅ |
| MS5803 driver | ✅ |
| Sensor node | ✅ |
| IMU topic | ✅ |
| Pressure topic | ✅ |
| Encoder node | ⬜ |
| Robot State Node | ⬜ |
| Camera Node | ⬜ |
| Network Monitor Node | ⬜ |

### 9. Progress Keseluruhan Proyek

**Ground Station:**

| Item | Status |
|------|--------|
| Ubuntu Desktop | ✅ |
| ROS2 Jazzy | ✅ |
| GitHub | ✅ |
| Joystick Boxer | ✅ |
| robot_command topic | ✅ |

**Modul:**

| Modul | Progress | Prioritas |
|-------|----------|-----------|
| Infrastructure | 100% | 1 |
| ROS2 Communication | 100% | 2 |
| Sensor Driver | 100% | 3 |
| Sensor Node | 100% | 4 |
| Encoder | 0% | 5 |
| Robot State Node | 0% | 6 |
| Camera Node | 0% | 7 |
| Network Monitor | 0% | — |
| GUI Telemetry | 0% | — |

Estimasi keseluruhan software underwater robot: ~60% selesai.

### 10. Next Step (Milestone Berikutnya)

| Prioritas | Task |
|-----------|------|
| 1 | Buat DriveEncoder.msg |
| 2 | Buat encoder_node |
| 3 | Buat RobotState.msg |
| 4 | Buat robot_state_node |
| 5 | Subscribe /imu/data |
| 6 | Subscribe /pressure |
| 7 | Publish /robot_state |
| 8 | Test telemetri_gui_node |

### Kondisi Saat Ini

- ✅ Sensor fisik sudah terbaca
- ✅ Driver sensor sudah selesai
- ✅ Topic sensor sudah publish
- ✅ ROS2 communication berjalan

## Terkait

**Sebelum / sesudah:**

- [[22_September_2026]]
- [[24_September_2026]]
- [[26_September_2026]]

**Dokumen terkait:**

- [[Project_Overview]]
- [[n_sensor_node]]
- [[joystick_interface_node]]
- [[5_October_2026]]

---
title: 22 September 2026
date: 2026-09-22
---

# 22 September 2026

## Resume Progress ROS2 Ground Station ↔ Underwater Robot

Tanggal: 22 September 2026

Berdasarkan progress pada Quick Notes, komunikasi ROS2 antara Ground Station dan Raspberry Pi sudah berhasil berjalan hingga level publish-subscribe custom message. Hasil troubleshooting hari ini:

### 1. List Problem dan Solusinya

| No | Problem | Gejala / Error | Root Cause | Solusi | Status |
|----|---------|----------------|------------|--------|--------|
| 1 | Workspace ROS2 tidak bisa di-source | `install/setup.bash: No such file or directory` | Workspace dipindah dari `~/ros2_ws` ke `~/ros2_gs/ros2_ws` sehingga build lama tidak valid | Build ulang workspace | ✅ Selesai |
| 2 | Build package gagal | `CMakeCache.txt directory is different` | Cache build masih mengarah ke path lama `/home/exedy/ros2_ws` | Hapus folder build, install, dan log lalu build ulang | ✅ Selesai |
| 3 | ROS2 package tidak muncul | `ros2 pkg list` tidak menampilkan package | Workspace belum berhasil build setelah dipindahkan | Jalankan `colcon build --symlink-install` | ✅ Selesai |
| 4 | Raspi tidak bisa membaca custom topic | `The message type 'brushkart_msgs/msg/OperatorCommand' is invalid` | Package custom message belum ada di Raspi | Clone repo yang berisi brushkart_msgs ke Raspi | ✅ Selesai |
| 5 | Custom message tidak dikenali ROS2 | Error saat `ros2 topic echo /robot_command` | Interface message belum dibuild di Raspi | Build workspace ROS2 di Raspi dan source install | ✅ Selesai |
| 6 | Validasi antar device | Belum diketahui apakah DDS sudah jalan | Belum pernah diuji | `ros2 topic list` di Raspi berhasil melihat topic dari Mini PC | ✅ Selesai |
| 7 | Sinkronisasi source code Ground Station dan Raspi | Workspace berbeda di masing-masing device | Package custom harus identik | Copy repository brushkart_msgs ke repo ros2_ur | ✅ Selesai |
| 8 | Akses GitHub dari Raspi | Belum bisa clone repository | SSH key belum dikonfigurasi | Registrasi SSH key Raspi ke GitHub | ✅ Selesai |

### 2. Detail Command Troubleshooting yang Digunakan

| Tahap | Command |
|-------|---------|
| Hapus build lama | `rm -rf build install log` |
| Source ROS2 Jazzy | `source /opt/ros/jazzy/setup.bash` |
| Build workspace | `colcon build --symlink-install` |
| Source workspace | `source install/setup.bash` |
| Cek package | `ros2 pkg list \| grep brushkart` |
| Cek executable | `ros2 pkg executables \| grep brushkart` |
| Cek topic | `ros2 topic list` |
| Echo topic | `ros2 topic echo /robot_command` |
| Clone repo | `git clone git@github.com:skwnpro/ros2_ur.git` |
| Verifikasi interface | `ros2 interface show brushkart_msgs/msg/OperatorCommand` |

### 3. Milestone Terbaru

| Milestone | Detail | Status |
|-----------|--------|--------|
| Ground Station Workspace | Workspace ROS2 berhasil dipindahkan ke `~/ros2_gs/ros2_ws` | ✅ |
| Joystick Interface | Radiomaster Boxer berhasil dibaca oleh ROS2 | ✅ |
| Joy Node | joy_node berjalan normal | ✅ |
| Custom Interface Node | joystick_interface_node berjalan normal | ✅ |
| Publish Robot Command | Data joystick dipublish ke /robot_command | ✅ |
| Custom Message | brushkart_msgs berhasil dibuat | ✅ |
| GitHub Ground Station | Repo Ground Station tersedia | ✅ |
| GitHub Underwater Robot | Repo ros2_ur tersedia | ✅ |
| SSH Mini PC → Raspi | Remote akses berhasil | ✅ |
| Ethernet Mini PC ↔ Raspi | Komunikasi jaringan berhasil | ✅ |
| ROS2 Jazzy di Raspi | Instalasi selesai | ✅ |
| DDS Discovery | Topic ROS2 dari Mini PC terlihat di Raspi | ✅ |
| Clone Repository di Raspi | Repo berhasil disalin | ✅ |
| Build Custom Message di Raspi | brushkart_msgs berhasil dibuild | ✅ |
| Subscribe Custom Topic di Raspi | /robot_command berhasil dibaca dari Raspi | ✅ |

### 4. Arsitektur Sistem yang Saat Ini Sudah Berjalan

```
Radiomaster Boxer
        ▼
Ground Station (Mini PC) — Ubuntu 24.04 + ROS2 Jazzy
        ▼
joy_node → joystick_interface_node
        ▼
/robot_command (brushkart_msgs/OperatorCommand)
        ▼  (Ethernet Tether)
Raspberry Pi 4B — Ubuntu Server 24.04 + ROS2 Jazzy
        ▼
Subscriber berhasil menerima data
```

Status komunikasi ROS2 Ground Station → Raspberry Pi sudah berhasil 100%.

### 5. Milestone Berikutnya (Recommended)

| Prioritas | Task | Target |
|-----------|------|--------|
| 1 | Buat package command_gateway | Raspi dapat mengolah /robot_command |
| 2 | Subscriber OperatorCommand | Parsing command operator |
| 3 | Print Debug Command | Verifikasi mapping joystick |
| 4 | GPIO Test | Kontrol output tanpa aktuator |
| 5 | Solenoid Driver | Kontrol aktuator hidrolik |
| 6 | Sensor Node MS5803 | Publish pressure/depth |
| 7 | Sensor Node BNO055 | Publish IMU data |
| 8 | Robot State Node | Health check seluruh sistem |
| 9 | Camera Node | Streaming CSI camera |
| 10 | Telemetry GUI | Monitoring dari Ground Station |

### Kondisi Project Saat Ini

Fondasi komunikasi sistem sudah selesai. Yang sudah terbukti bekerja:

- ✅ Radiomaster Boxer → ROS2
- ✅ ROS2 Ground Station → publish /robot_command
- ✅ Ethernet tether Mini PC ↔ Raspi
- ✅ DDS Discovery ROS2
- ✅ Custom message OperatorCommand
- ✅ Raspberry Pi menerima /robot_command

Milestone besar berikutnya: membuat command_gateway_node di Raspberry Pi yang subscribe /robot_command dan mengubah command tersebut menjadi output GPIO/solenoid untuk sistem hidrolik underwater robot.

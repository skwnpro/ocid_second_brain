---
title: Operate ROS2
date: 2026-09-24
---

# Operate ROS2

## Cara Build dari Awal

1. Masuk ke workspace:
   ```bash
   cd ~/ros2_ur/ros2_ws
   ```
2. Source ROS2:
   ```bash
   source /opt/ros/jazzy/setup.bash
   ```
3. Build package tertentu:
   ```bash
   colcon build --packages-select brushkart_msgs
   ```
4. Source workspace hasil build:
   ```bash
   source install/setup.bash
   ```

## Kalo Gagal dan Mau Build Ulang

1. Masuk ke workspace:
   ```bash
   cd ~/ros2_ur/ros2_ws
   ```
2. Hapus file yang lama:
   ```bash
   rm -rf build install log
   ```
3. Source ROS2:
   ```bash
   source /opt/ros/jazzy/setup.bash
   ```
4. Build package:
   ```bash
   colcon build --packages-select brushkart_msgs
   ```
5. Source workspace hasil build:
   ```bash
   source install/setup.bash
   ```

## Penambahan Dependencies / Interfaces / Message Baru

Tinggal tambah di baris baru di bawah itu. Ini potongan file `CMakeLists.txt`:

```cmake
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/OperatorCommand.msg"
  "msg/SensorData.msg"   # <---- di sini langsung
  DEPENDENCIES std_msgs
)
```

## Cara Bikin Package

Tipe data C dan Python:

```bash
# Python
ros2 pkg create --build-type ament_python --license Apache-2.0 --node-name my_node my_package

# CMake
ros2 pkg create --build-type ament_cmake --license Apache-2.0 --node-name my_node my_package
```

## Terkait

- [[ROS2_Bruskart_Robot_Project_Overview]]
- [[ROS2_Installation]]
- [[22_September_2026]]
- [[joystick_interface_node]]

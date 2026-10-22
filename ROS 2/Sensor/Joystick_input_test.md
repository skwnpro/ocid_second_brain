---
title: Joystick Input Test
date: 2026-09-19
---

# Joystick Input Test

Perintah untuk test: `jstest-gtk`

## Mapping Axis & Button

| Input   | Fungsi                     |
|---------|----------------------------|
| Axis0   | joystick kanan, kanan-kiri  |
| Axis1   | joystick kanan, atas-bawah  |
| Axis2   | joystick kiri, atas-bawah   |
| Axis3   | joystick kiri, kanan-kiri   |
| Axis4   | SA, 2 switch               |
| Axis5   | SB, 3 switch               |
| Axis6   | SC, 3 switch               |
| Button0 | SE, detend button          |
| Button1 | SF, pushbutton             |

## Nilai Terbaca

- Axis0 — kanan: -0.992, kiri: 1
- Axis1 — atas: -1, bawah: 1
- Axis2 — atas: -0.97738, bawah: 0.98666
- Axis3 — kanan: -1, kiri: 0.996
- Axis4 — atas: 0, bawah: -1
- Axis5 — atas: 1, tengah: 0, bawah: -1
- Axis6 — atas: 1, tengah: 0, bawah: -1
- Button0 — on: 1, off: 0
- Button1 — on: 1, off: 0

## Cara Run Joystick

1. Buka terminal B:
   ```bash
   cd ~/ros2_gs/ros2_ws
   source install/setup.bash
   source /opt/ros/jazzy/setup.bash
   ros2 run joy joy_node
   ```
2. Buka terminal A:
   ```bash
   cd ~/ros2_gs/ros2_ws
   source install/setup.bash
   ros2 run brushkart_joystick joystick_interface_node
   # muncul "joy received"
   ```
3. Buka terminal C:
   ```bash
   cd ~/ros2_gs/ros2_ws
   source install/setup.bash
   ros2 topic echo /robot_command
   ```

## Terkait

- [[ROS2_Bruskart_Robot_Project_Overview]]
- [[joystick_interface_node]]
- [[1_October_2026]]

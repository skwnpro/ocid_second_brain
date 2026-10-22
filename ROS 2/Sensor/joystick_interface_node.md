---
title: joystick_interface_node
date: 2026-09-18
---

# joystick_interface_node

## Connect Radiomaster Boxer dengan Ubuntu Mini PC

1. Cek apakah Ubuntu bisa baca Boxer:
   ```bash
   lsusb
   ```
   Harus muncul `Radiomaster Boxer` atau `HID Device`.
2. Install:
   ```bash
   sudo apt install joystick
   ```
3. Test device:
   ```bash
   ls /dev/input/js0
   ```
4. Install ROS2 Joy package:
   ```bash
   sudo apt install ros-jazzy-joy
   ```
5. Run:
   ```bash
   ros2 run joy joy_node
   ```
6. Lihat datanya:
   ```bash
   ros2 topic echo /joy
   ```
   Output seharusnya muncul nilai tiap kita gerakkan joystick. Catat tiap gerakan di `axes:` dan `button:`.

## Mapping Joystick ke Perintah Drive dan Steering

- Left stick Y = Drive (maju mundur)
- Right stick X = Steering (kiri kanan)
- SA Switch = Brush ON/OFF
- SB Switch = Emergency Stop
- Potentiometer S1 = Thruster

## Code

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Joy
from std_msgs.msg import Bool
from hull_brush_interfaces.msg import RobotCommand


class JoystickInterfaceNode(Node):
    def __init__(self):
        super().__init__('joystick_interface_node')
        self.robot_pub = self.create_publisher(
            RobotCommand,      # <- type
            '/robot_command',  # <- topic
            10                 # <- queue
        )
        self.estop_pub = self.create_publisher(
            Bool,
            '/emergency_stop',
            10
        )
        self.joy_sub = self.create_subscription(
            Joy,                  # <- type
            '/joy',               # <- topic
            self.joy_callback,    # <- callback: kalo ada data baru di /joy, jalankan joy_callback()
            10                    # <- kalo belum sempat keproses, simpan maksimal 10 pesan
        )

    def joy_callback(self, msg):
        cmd = RobotCommand()

        # Left Stick Y -> drive
        cmd.drive = int(msg.axes[1] * 100)
        # Right Stick X -> steering
        cmd.steering = int(msg.axes[3] * 100)
        # S1 knob -> thruster
        cmd.thruster = int(msg.axes[5] * 100)

        # SA Switch -> brush ON/OFF
        if msg.buttons[4]:
            cmd.brush = 100
        else:
            cmd.brush = 0

        self.robot_pub.publish(cmd)

        # SB Switch -> emergency stop
        estop = Bool()
        if msg.buttons[5]:
            estop.data = True
        else:
            estop.data = False
        self.estop_pub.publish(estop)


def main(args=None):
    rclpy.init(args=args)
    node = JoystickInterfaceNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

## Penjelasan Kode

- **Membuat class node**: `JoystickInterfaceNode` mewarisi `Node` — semua node ROS biasanya turunan dari class Node ini, jadi node ini punya semua fitur ROS2.
- **Akses library**: `rclpy` (library utama ROS2 Python), `Node`, `Joy` (message standar ROS2 untuk joystick), `Bool` (dipakai untuk emergency stop), `RobotCommand` (custom message yang kita buat sendiri, berisi drive, steering, brush, thruster).
- **Constructor (`__init__`)**: kalau di Arduino ini kayak `void setup()`. Kasih nama node lewat `super().__init__('joystick_interface_node')` — bisa dicek dengan `ros2 node list`.
- **Publisher robot command**: `self.robot_pub = self.create_publisher(RobotCommand, '/robot_command', 10)` — node ini diizinkan mengirim data ke `/robot_command`, nanti dikirim pakai `self.robot_pub.publish(msg)`.
- **Publisher emergency**: `self.estop_pub = self.create_publisher(Bool, '/emergency_stop', 10)` — bisa kirim data ke `/emergency_stop` pakai tipe data `std_msgs/Bool`.
- **Subscriber**: `self.joy_sub = self.create_subscription(Joy, '/joy', self.joy_callback, 10)` — node subscribe topic `/joy` pakai tipe data `Joy`; kalau ada data baru di `/joy`, jalankan fungsi `joy_callback()`.
- **Callback**: fungsi ini otomatis dipanggil ROS2 setiap ada data baru dari topic `/joy`. Buat message baru `cmd = RobotCommand()` (data kosong), isi `drive` dari `msg.axes[1]`, `steering` dari `msg.axes[3]`, `thruster` dari `msg.axes[5]`, `brush` dari switch SA (`msg.buttons[4]`), lalu publish. Emergency stop dari switch SB (`msg.buttons[5]`) dipublish ke `/emergency_stop`.

## RobotCommand.msg

```
int16 drive
int16 steering
int16 thruster
bool brush_enable
```

## Terkait

- [[ROS2_Bruskart_Robot_Project_Overview]]
- [[Joystick_input_test]]
- [[1_October_2026]]
- [[24_25_September]]

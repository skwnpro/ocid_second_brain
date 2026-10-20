---
title: Brush Kart — Review Project, Status Node, dan Rencana ke Depan
date: 2026-10-20
tags:
  - brush-kart
  - ros2
  - review
---

# Brush Kart — Review Project, Status Node, dan Rencana ke Depan

Review berdasarkan: diagram node terbaru (`ROS2_node_0_omks.json`, termasuk flowchart semi-auto dan penanda merah = belum dibuat), repo `ros2_ur` (robot), dan repo `ros2_gs` (ground station). Status per 20 Oktober 2026.

## 1. Ringkasan Project

Brush Kart adalah robot underwater berpenggerak hidrolik (drive, steering, brush, thruster) yang dikendalikan dari ground station lewat tether Ethernet. Arsitektur ROS 2 Jazzy terbagi dua sisi:

- **Ground Station** (mini PC, Ubuntu Desktop 24.04) — repo `ros2_gs`: operator membaca Radiomaster Boxer, perintah diterjemahkan menjadi logika relay, status robot diagregasi untuk ditampilkan.
- **Underwater Robot** (Raspberry Pi 4B, Ubuntu Server 24.04) — repo `ros2_ur`: node yang berhadapan langsung dengan hardware (sensor I2C, kamera, GPIO relay/solenoid, monitor jaringan).

Alur kendali: joystick → `/robot_command` → command_gateway → `/relay_operator` → GPIO → solenoid hidrolik.
Alur telemetri: sensor → `/sensor_data`, network_monitor → `/network_status`, keduanya diagregasi robot_state_node → `/robot_state`, kamera → topic image — semua ditampilkan GUI di ground station.

Kontrak data (`brushkart_msgs`) ada di kedua repo: `OperatorCommand`, `RelayOperator`, `SensorData`, `NetworkStatus`, dan `RobotState` (yang terakhir **hanya ada di ros2_gs**).

**Kesimpulan umum:** fondasi V1 (kontrol manual + telemetri dasar) sudah kebentuk — 7 dari 8 node di diagram sudah ada kodenya. Tapi sistem **belum pernah terintegrasi penuh**: belum ada launcher, satu node (robot_state) rusak di runtime, satu node (GUI) belum ada sama sekali, dan rantai failsafe belum ada. Fitur semi-auto (semua penanda merah di diagram) belum dimulai dan memang terblokir oleh dua sensor yang belum dibuat programnya (limit switch + rotary encoder).

## 2. Status Tiap Node

### Ground Station (ros2_gs)

| Node | Package | Status | Catatan |
|---|---|---|---|
| joystick_interface_node | brushkart_joystick | 🟡 Jalan untuk manual dasar | Baca `/joy`, mapping axis hardcoded, publish `/robot_command` tiap callback. **Belum ada:** mode auto/manual + auto start (merah di diagram), deadzone/filter. Bergantung `joy_node` eksternal yang belum ada launcher-nya. |
| command_gateway_node | brushkart_gateway | 🟡 Jalan, ada celah | Terjemahkan command → relay dengan deadband 0,30; e-stop meng-nol-kan semua relay. **Celah:** field `thruster` dari joystick **diabaikan total** (tidak ada relay thruster di `RelayOperator.msg`); belum subscribe `/sensor_data` (itu V2/semi-auto, merah); belum ada watchdog kalau command berhenti datang. |
| robot_state_node | brushkart_state | 🔴 Rusak di runtime | **Bug:** kode menulis `msg.connected`, padahal field di `RobotState.msg` bernama `connection` → `AttributeError` begitu kedua input (sensor + network) sudah masuk, node crash di timer publish. Logika health-check-nya sendiri sesuai diagram, tapi **belum ada cek data basi (staleness)** — kalau sensor_node mati, data terakhir tetap dianggap sehat selamanya. Field `esp32_ok` di contoh diagram juga tidak ada di message. |
| telemetri_gui_node | — | 🔴 Belum ada sama sekali | Tidak ada package GUI di kedua repo. Ini satu-satunya node diagram yang kodenya nol: tampilan `/robot_state` + kamera di monitor 7" belum dibuat. |

### Underwater Robot (ros2_ur)

| Node | Package | Status | Catatan |
|---|---|---|---|
| sensor_node | brushkart_sensor | 🟡 Sebagian | Baca BNO055 (IMU) + MS5803 (pressure/depth) @10 Hz, driver I2C-nya beneran dan rapi. **Belum ada (merah di diagram):** program baca rotary encoder dan limit switch — dan `SensorData.msg` memang belum punya field untuk keduanya. **Celah:** `imu_ok`/`pressure_ok` di-hardcode `True` (tidak pernah deteksi gagal baca), header timestamp tidak pernah diisi, ada `print("timer running")` 10×/detik. |
| relay_operator_node | brushkart_operator | 🟡 Paling matang, tapi tanpa failsafe | Mapping GPIO (brush 5, fwd 6, retreat 13, left 19, right 26, light 16), e-stop prioritas tertinggi, semua relay mati saat start/exit. **Celah kritis: tidak ada watchdog** — kalau `/relay_operator` berhenti datang (tether putus, GS crash), relay **mempertahankan state terakhir selamanya**; motor yang sedang jalan tidak akan berhenti. Juga belum ada interlock forward+retreat bersamaan, flag `ACTIVE_HIGH` harus diverifikasi ke relay board fisik (umumnya active-low), dan `RPi.GPIO` berisiko bermasalah di Ubuntu 24.04. |
| camera_node | brushkart_camera | 🟡 Jalan, beda kontrak | Publish raw + compressed @10 FPS, 320×240. **Beda dari diagram:** nama topic-nya `/camera/raw_image` dan `/camera/compressed`, diagram menulis `/camera/image_raw`. **Beda dari overview:** kode pakai kamera USB/V4L2 (`VideoCapture(0)`), overview menyebut kamera CSI. QoS RELIABLE untuk video di tether kurang ideal (potensi latency menumpuk; umumnya BEST_EFFORT). |
| network_monitor_node | brushkart_network | 🟢 Paling lengkap | Ping ke GS (192.168.10.1) @1 Hz → latency, packet loss, bandwidth, CPU/RAM Raspi ke `/network_status`. Sesuai diagram. Celah minor: ping subprocess bisa memblokir callback sampai ±4 detik saat koneksi buruk; `last_rx_ms` masih kasar. |

## 3. Temuan Lintas Sistem

1. **Tidak ada launcher** (sudah dikonfirmasi) — tidak ada satu pun file `launch/` di kedua repo; semua node harus dijalankan manual satu-satu, termasuk `joy_node` yang tidak tercatat di mana pun sebagai dependensi. Konfigurasi dua mesin (ROS_DOMAIN_ID, network) juga belum terdokumentasi di repo (README kedua repo praktis kosong).
2. **Diagram ≠ kode di beberapa titik:** topic kamera (di atas), `/emergency_stop` di diagram adalah topic terpisah tapi di kode e-stop dilipat ke field `estop` di `OperatorCommand`/`RelayOperator` (keputusan yang sah, tapi diagram perlu disamakan), dan nama topic di diagram tertulis `/robot_comand` (typo) vs kode `/robot_command`.
3. **`brushkart_msgs` ganda dan mulai divergen:** 4 message yang sama masih identik antar repo, tapi `RobotState.msg` hanya ada di ros2_gs. Selama dua copy ini diedit terpisah, risiko definisi message tidak sinkron antar dua mesin selalu ada. Butuh satu sumber kebenaran.
4. **Konvensi tanda (sign) rantai kendali rapuh:** joystick menegasikan axis (`drive = -axes[2]`), gateway menafsirkan nilai negatif sebagai forward. Konsisten hari ini, tapi sekali ada yang mengubah satu sisi saja, robot bergerak terbalik. Perlu dibakukan + didokumentasikan.
5. **Peran ESP32 belum jelas:** diagram menyebut `esp32_ok`, "program bridging mini pc to ESP32", dan monitor 3,5" untuk data limit switch/pressure/encoder — tidak ada satu pun di kode maupun di Project Overview. Ini sub-sistem yang belum terdefinisi.
6. **Thruster belum punya jalur kendali ujung-ke-ujung:** joystick memetakan axis-nya dan message membawanya, tapi gateway tidak meneruskannya dan tidak ada relay untuknya.

## 4. Rencana ke Depan (berurutan)

Urutan di bawah mengikuti dependensi: fondasi dulu, baru fitur, dan robot simulasi kecil menjadi tempat pembuktian sebelum menyentuh robot hidrolik asli.

### Fase 0 — Fondasi & failsafe (prasyarat semua fase)
1. Fix crash robot_state_node (`connected` → `connection`) + tambah cek staleness data sensor/network.
2. Watchdog failsafe: relay_operator_node matikan semua relay bila `/relay_operator` diam melewati batas waktu; command_gateway_node publish stop bila `/robot_command` berhenti datang. **Ini perbaikan safety nomor satu untuk robot hidrolik.**
3. Samakan kontrak: putuskan nama topic final (kamera), samakan diagram dengan kode (e-stop, typo), dan jadikan `brushkart_msgs` satu sumber (satu repo khusus / submodule).
4. Launcher per sisi + dokumentasi setup dua mesin (termasuk `joy_node`, ROS_DOMAIN_ID, IP statis). Target: seluruh sistem nyala dengan satu perintah per mesin.

### Fase 1 — Lengkapi V1 manual
5. telemetri_gui_node: tampilkan `/robot_state`, `/sensor_data`, dan kamera (compressed) di monitor 7" — versi paling sederhana dulu.
6. sensor_node: health flag beneran (gagal baca I2C → `*_ok = False`, bukan crash), isi header timestamp, buang print debug.
7. Verifikasi hardware: polaritas relay board (ACTIVE_HIGH/LOW), mapping axis + konvensi tanda joystick dibakukan dan ditulis di README.
8. Putuskan jalur thruster: tambah relay di `RelayOperator.msg` + gateway + relay node, atau resmi ditunda ke V2.

### Fase 2 — Sensor yang masih merah (pengunci semi-auto)
9. Limit switch: reader GPIO di sensor_node + field baru di `SensorData.msg` (`lswitch_left/right`) — ini pengunci utama flowchart semi-auto.
10. Rotary encoder: reader + field rpm/pulse di `SensorData.msg` — untuk kontrol jarak/kecepatan dan odometri sederhana.

### Fase 3 — Semi-auto (V2 di diagram)
11. Mode auto/manual + auto start di joystick_interface_node.
12. State machine semi-auto di command_gateway_node sesuai flowchart: run forward → limit switch kena → stop → turn 15° (pakai yaw IMU) → retreat → turn 15° arah sebaliknya → forward lagi → sampai "depth reach" → END. Feedback dari `/sensor_data`.

### Fase 4 — Robot simulasi kecil & deployment
13. Bangun robot kecil (sasis mini, motor DC, limit switch depan, IMU, encoder, kamera USB, Raspi) sebagai testbed di darat. Karena logika kendali sistem ini berbasis relay on/off, seluruh stack (kedua workspace) bisa dideploy apa adanya ke robot kecil: GS di mini PC/laptop, UR di Raspi robot kecil.
14. Uji berurutan di robot kecil: kontrol manual → failsafe (cabut tether saat jalan) → telemetri/GUI → semi-auto flowchart sampai tuntas.
15. Setelah terbukti di robot kecil, port ke robot hidrolik asli dengan kepercayaan diri penuh.

## 5. Pertanyaan yang Masih Terbuka

1. ESP32 itu untuk apa sebenarnya — akuisisi sensor (limit switch/pressure/encoder) terpisah, atau display 3,5" saja? Jawabannya mengubah desain sensor_node di Fase 2.
2. Thruster: dipakai di V1 (tambah relay sekarang) atau memang nanti?
3. Kamera final: CSI (sesuai overview) atau USB (sesuai kode sekarang)?

## Terkait

- [[Project_Overview]]
- [[joystick_interface_node]]
- [[n_sensor_node]]
- [[Operate_ROS2]]
- [[Connection_Ethernet]]

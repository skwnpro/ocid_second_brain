---
title: 1 September 2026
date: 2026-10-02
---

# 1 September 2026

## Summary Chat Terbaru (01-10-2026)

### 1. Topik Utama yang Dibahas

| Area | Detail |
|------|--------|
| Command Gateway Node | Revisi alur data dan logika pengambilan keputusan node utama robot |
| Robot State Node | Dipisahkan dari command_gateway_node untuk meningkatkan reliability dan maintainability |
| Joystick Mapping | Mapping Axis0–Axis3 ke gerakan robot dan aktuator |
| Truth Table | Penyusunan tabel lengkap input joystick → output relay |
| Relay Control | Pembuatan program untuk mengontrol relay/I/O Raspberry Pi |
| GitHub | Troubleshooting koneksi SSH ke GitHub |

### 2. Revisi Arsitektur Sistem

**Keputusan akhir:**

Sebelumnya:
```
Joystick → command_gateway_node → robot_state → keputusan output relay
```

Revisi akhir:
```
Joystick → command_gateway_node → output relay langsung
robot_state_node → monitoring status robot
```

**Alasan pemisahan:**

| Pertimbangan | Hasil |
|---------------|-------|
| Keandalan | Lebih tinggi |
| Kompleksitas | Lebih rendah |
| Troubleshooting | Lebih mudah |
| Latency | Lebih kecil |
| Maintenance | Lebih mudah |
| Risiko satu node gagal | Lebih kecil |

**Kesimpulan:**
- ✅ command_gateway_node hanya menerima command dan mengontrol relay
- ✅ robot_state_node hanya memonitor kondisi robot

### 3. Mapping Joystick yang Digunakan

Data joystick dibaca dari topic `/robot_command`:

| Axis | Fungsi |
|------|--------|
| Axis0 | Joystick kanan, kiri-kanan |
| Axis1 | Joystick kanan, atas-bawah |
| Axis2 | Joystick kiri, atas-bawah |
| Axis3 | Joystick kiri, kiri-kanan |

### 4. Truth Table Gerakan Robot

| Input Joystick | Output |
|----------------|--------|
| Maju | Relay thruster maju ON |
| Mundur | Relay thruster mundur ON |
| Geser kiri | Relay kiri ON |
| Geser kanan | Relay kanan ON |
| Putar kiri | Relay rotasi kiri ON |
| Putar kanan | Relay rotasi kanan ON |
| Naik | Relay vertikal naik ON |
| Turun | Relay vertikal turun ON |
| Joystick dilepas (deadband) | Semua relay OFF |

Beserta kombinasi gerakan diagonal dan prioritas command.

**Fungsi deadband:**
- Membaca nilai joystick
- Menerapkan deadband
- Semua relay OFF saat joystick di posisi netral

### 5. Program yang Dibuat

**command_gateway_node:**
- Subscribe /robot_command
- Menentukan mode gerakan
- Mengaktifkan relay sesuai truth table
- Publish status jika diperlukan

**robot_state_node:**
- Baca sensor
- Status robot
- Status relay
- Status kamera
- Status jaringan
- Tidak ikut memutuskan gerakan robot
- Publish kondisi robot ke Ground Station

### 6. Masalah GitHub

Error:
```
ssh: Could not resolve hostname github.com: Temporary failure in name resolution
fatal: Could not read from remote repository
```

Analisis — masalah **bukan** pada:
- SSH key
- GitHub account
- Repository

Masalah pada:
- DNS resolution
- Network connection

Area yang harus dicek:

| Pemeriksaan | Command |
|-------------|---------|
| Ping IP Google | `ping 8.8.8.8` |
| Ping github | `ping github.com` |
| DNS config | `cat /etc/resolv.conf` |
| Interface network | `ip addr` |
| Routing | `ip route` |

### 7. Progress Saat Ini

| Modul | Status |
|-------|--------|
| ROS2 Jazzy | ✅ Selesai |
| DDS Ethernet Raspi ↔ Mini PC | ✅ Selesai |
| Joystick Publisher | ✅ Selesai |
| command_gateway_node | ✅ Logic selesai |
| robot_state_node | ✅ Arsitektur selesai |
| Kamera streaming | ✅ Berfungsi |
| Network status node | ✅ Berfungsi |
| UI planning | ✅ Draft selesai |
| Relay driver | Sedang dikerjakan |
| Sensor integration | Berikutnya |
| Full system integration | ⏳ Belum |

### 8. Milestone Terakhir

- ✅ Kamera berhasil streaming ke Ground Station
- ✅ Network status node berhasil publish data
- ✅ Arsitektur node sudah final
- ✅ command_gateway_node dipisah dari robot_state_node
- ✅ Truth table joystick → relay selesai
- ✅ Siap masuk tahap kontrol relay hardware nyata

### Prioritas Testing Berikutnya

| Prioritas | Task |
|-----------|------|
| 1 | Test relay ON/OFF manual melalui GPIO Raspberry Pi |
| 2 | Verifikasi nomor GPIO yang terhubung ke relay board |
| 3 | Jalankan program relay standalone |
| 4 | Integrasikan relay control ke command_gateway_node |
| 5 | Uji joystick → relay nyata |
| 6 | Verifikasi fail-safe saat joystick dilepas |
| 7 | Integrasikan sensor ke robot_state_node |
| 8 | Integrasi penuh Ground Station ↔ Robot |

## Terkait

**Sebelum / sesudah:**

- [[22_September_2026]]

**Dokumen terkait:**

- [[Project_Overview]]
- [[joystick_interface_node]]
- [[Joystick_input_test]]
- [[Git_Uses]]

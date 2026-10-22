---
title: Brush Kart — Form & Checklist Testing
date: 2026-10-21
tags:
  - brush-kart
  - ros2
  - testing
  - checklist
---

# Brush Kart — Form & Checklist Testing

Cara pakai: salin file ini untuk setiap sesi tes, beri nama `Testing_YYYY-MM-DD_<nama-sesi>.md`, isi semua kolom hasil. Satu sesi = satu file. Status hanya boleh diisi **Lulus** bila kriteria terpenuhi *dan* buktinya tercatat (screenshot, log, atau file rosbag).

## Info Sesi

| Item | Isi |
|---|---|
| Tanggal / jam | |
| Nama sesi | |
| Penguji | |
| Lokasi & medium (darat / kolam / air tawar / air laut) | |
| Versi `ros2_ur` (commit/tag) | |
| Versi `ros2_gs` (commit/tag) | |
| ROS_DOMAIN_ID | |
| IP Ground Station / Robot | 192.168.10.1 / 192.168.10.2 |
| Varian sensor pressure terpasang | |
| File rosbag sesi ini | |
| Catatan konfigurasi khusus | |

---

## Bagian A — Checklist Pra-Operasi (sebelum sistem dinyalakan)

- [ ] Tether Ethernet terpasang & terkunci di kedua sisi
- [ ] Power robot: AC 220V → PSU 24VDC → DC-DC 5V/3,3V, semua tegangan terukur normal
- [ ] Semua relay dalam keadaan OFF sebelum node dinyalakan (cek fisik / dengarkan klik)
- [ ] Overlay IMX290 aktif di Raspi (`dtoverlay -l` atau `cam -l` menunjukkan imx290)
- [ ] Sensor terdeteksi di I2C (`i2cdetect`: BNO055 0x29, MS58xx 0x76)
- [ ] Radiomaster Boxer terhubung ke mini PC, profil/channel map sesuai dokumentasi
- [ ] E-stop fisik/joystick diuji manual: menekan tombol membuat semua relay mati
- [ ] Versi software di kedua mesin sama dengan yang tercatat di Info Sesi
- [ ] Area tes aman: tidak ada orang di dekat aktuator hidrolik / jalur robot

## Bagian B — Clean Build (hanya untuk sesi clean-room build test)

| Langkah                                          | Kriteria lulus                                                | Hasil (✅/❌) | Catatan |
| ------------------------------------------------ | ------------------------------------------------------------- | ----------- | ------- |
| Clone `ros2_ur` + `ros2_gs` di mesin bersih      | Clone berhasil                                                |             |         |
| `rosdep install` dari package.xml                | Semua dependensi ter-resolve, tidak ada package tidak dikenal |             |         |
| `colcon build` workspace GS                      | Sukses tanpa error                                            |             |         |
| `colcon build` workspace robot                   | Sukses tanpa error                                            |             |         |
| Semua executable muncul (`ros2 pkg executables`) | 7 node terdaftar                                              |             |         |
| Sistem dinyalakan hanya dari README/dokumentasi  | Berhasil tanpa langkah dari ingatan                           |             |         |

## Bagian C — Smoke Test Integrasi (semua node, dua mesin menyala bersamaan)

| Node | Mesin | Yang diverifikasi | Kriteria lulus | Hasil (✅/❌) | Catatan |
|---|---|---|---|---|---|
| joy_node | GS | Publish `/joy` saat stick digerakkan | Topic ada & berubah | | |
| joystick_interface_node | GS | Publish `/robot_command` | Nilai mengikuti stick | | |
| command_gateway_node | GS | Publish `/relay_operator` | Berubah sesuai command | | |
| relay_operator_node | Robot | Subscribe `/relay_operator` → GPIO | Relay berbunyi sesuai pesan | | |
| sensor_node | Robot | Publish `/sensor_data` ±10 Hz, stabil 5 menit | Tidak ada gap > 1 detik | | |
| network_monitor_node | Robot | Publish `/network_status` ±1 Hz | `connected=true` | | |
| robot_state_node | GS | Publish `/robot_state` ±2 Hz | `robot_ok=true`, tidak crash dalam 10 menit | | |
| camera_node | Robot | Publish `/camera/compressed` | Terlihat di GS, frame tidak hitam | | |

## Bagian D — Uji Rantai Kendali (mapping & arah)

Isi kolom "Aktual" persis seperti yang terjadi, bukan yang diharapkan.

| # | Aksi | Relay yang seharusnya aktif | Aktual | Sesuai? (✅/❌) |
|---|---|---|---|---|
| D1 | Stick netral | Semua OFF | | |
| D2 | Stick maju penuh | relay2_forward (GPIO 6) | | |
| D3 | Stick mundur penuh | relay3_retreat (GPIO 13) | | |
| D4 | Stick kiri penuh | relay4_left (GPIO 19) | | |
| D5 | Stick kanan penuh | relay5_right (GPIO 26) | | |
| D6 | Trigger brush (axes[4] penuh) | relay1_brush (GPIO 5) | | |
| D7 | Stick maju < 30% (di dalam deadband) | Semua OFF | | |
| D8 | Stick maju tepat di atas deadband | relay2_forward, tanpa hentakan berlebih | | |
| D9 | Kombinasi maju + kiri | relay2 + relay4 bersamaan | | |
| D10 | Thruster digerakkan | *(belum ada jalur relay — catat perilaku aktual)* | | |

### Uji E-Stop

| # | Aksi | Kriteria target | Aktual | Sesuai? (✅/❌) |
|---|---|---|---|---|
| D11 | Robot bergerak → tekan e-stop | Semua relay mati < 0,5 detik | | |
| D12 | E-stop dilepas saat stick masih miring | Robot **tidak** bergerak sampai ada re-arm sadar | | |
| D13 | E-stop ditekan saat brush menyala | Brush ikut mati | | |

## Bagian E — Uji Sensor & Akurasi

### E1. Pressure / Depth

> Catatan: MS5803-**01BA** ber-range 0–1 bar absolut dan akan jenuh di sekitar permukaan air. Uji ini justru untuk membuktikan perilaku itu secara terukur. Catat juga tekanan mentah (mbar), bukan hanya depth.

| Kedalaman terukur (meteran) | Pressure terbaca (mbar) | Depth terbaca (m) | Error | Catatan |
|---|---|---|---|---|
| Di udara (permukaan) | | | — | Referensi tekanan udara lokal |
| 0,5 m | | | | |
| 1,0 m | | | | |
| Kedalaman kerja target: ___ m | | | | |

### E2. IMU (BNO055)

| Uji | Kriteria lulus | Hasil aktual | Sesuai? (✅/❌) |
|---|---|---|---|
| Robot diam di permukaan datar | roll & pitch ≈ 0° (±2°) | | |
| Robot dimiringkan ±30° (sudut terukur) | Pembacaan mengikuti ±3° | | |
| Diam 5 menit | Yaw drift tercatat: ___ ° | | |
| Status kalibrasi (SYS/GYRO/ACC/MAG) | Tercatat: ___/___/___/___ | | |
| Robot di dekat lambung baja / motor | Yaw dibandingkan kompas referensi, selisih: ___ ° | | |

### E3. Jaringan

| Uji | Kriteria lulus | Hasil aktual | Sesuai? (✅/❌) |
|---|---|---|---|
| Ping robot → GS, 60 detik | Loss 0%, latency stabil | | |
| Latency perintah GS → klik relay (stopwatch, 5×) | Rata-rata: ___ ms, maks: ___ ms | | |
| `/network_status` selama beban kamera penuh | `connected` tetap true, loss 0% | | |

## Bagian F — Uji Kamera (IMX290 via libcamerify)

| # | Uji | Kriteria lulus | Hasil (✅/❌) | Catatan |
|---|---|---|---|---|
| F1 | Boot dingin Raspi → kamera langsung bisa dipakai tanpa perintah manual | Stream jalan setelah boot | | |
| F2 | Frame tidak hitam | Mean pixel > 10 pada ruangan menyala | | |
| F3 | Warna benar (objek warna dikenal di depan kamera) | Tidak hijau / tidak pink | | |
| F4 | FPS diterima di GS | Tercatat: ___ FPS | | |
| F5 | Latency gerak tangan di depan kamera → tampil di GS | < 0,5 detik (perkiraan stopwatch) | | |
| F6 | Stream stabil 15 menit | Tidak freeze; jumlah freeze: ___ | | |
| F7 | Kondisi gelap + lampu robot menyala | Objek pada jarak kerja terlihat jelas | | |

## Bagian G — Fault Injection (sistem harus gagal dengan aman)

Kolom "Perilaku aktual" adalah inti sesi ini — tulis apa adanya, termasuk yang memalukan.

| # | Gangguan | Perilaku target | Perilaku aktual | Aman? (✅/❌) |
|---|---|---|---|---|
| G1 | Tether dicabut saat robot bergerak maju | Semua relay mati ≤ 1 detik | | |
| G2 | Tether dicabut saat brush menyala | Brush mati ≤ 1 detik | | |
| G3 | joystick_interface_node dimatikan (kill) | Relay mati ≤ 1 detik | | |
| G4 | command_gateway_node dimatikan | Relay mati ≤ 1 detik | | |
| G5 | sensor_node dimatikan | `/robot_state` menunjukkan fault ≤ 2 detik | | |
| G6 | Salah satu sensor dicabut (I2C) | sensor_node tetap hidup, flag `*_ok=false` | | |
| G7 | Kamera dicabut / pipeline libcamera mati | Node lain tidak terpengaruh; error jelas di log | | |
| G8 | Joystick USB dicabut dari GS | Tidak ada command basi yang diteruskan | | |
| G9 | Mini PC (GS) restart total | Robot berhenti & diam sampai GS pulih | | |
| G10 | Raspi restart total | Saat boot: relay tetap OFF sampai perintah pertama | | |

## Bagian H — Uji Semi-Auto *(belum berlaku — isi setelah state machine selesai ditulis)*

| # | Skenario | Kriteria lulus | Hasil (✅/❌) | Catatan |
|---|---|---|---|---|
| H1 | Auto start dari mode manual | State machine masuk state pertama dengan benar | | |
| H2 | Run forward → limit switch kena | Stop ≤ ___ cm setelah kontak | | |
| H3 | Urutan belok 15° (IMU relatif) | Error sudut ≤ ___ ° | | |
| H4 | Pola zig-zag 3 lane | Tidak ada celah/overlap > ___ cm | | |
| H5 | Abort oleh operator di tengah auto | Kembali ke manual ≤ 0,5 detik | | |
| H6 | Robot ditahan (stall) saat auto | Sistem mendeteksi & berhenti ≤ ___ detik | | |
| H7 | Depth target tercapai | Auto berhenti & masuk END | | |

## Bagian I — Regresi (bug lama yang tidak boleh kambuh)

- [ ] robot_state_node publish terus-menerus tanpa AttributeError
- [ ] Isi `brushkart_msgs` di kedua mesin identik (diff file .msg)
- [ ] Kamera tidak kembali hitam setelah restart node
- [ ] `/robot_command` terbaca di Raspi (DDS lintas mesin sehat)
- [ ] Tidak ada topic duplikat / node ganda setelah restart seluruh sistem

## Bagian J — Catatan Defect Sesi Ini

| # | Temuan | Severity (Safety/Mayor/Minor) | Issue (link GitHub) | Status |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |

## Kesimpulan Sesi

| Item | Isi |
|---|---|
| Hasil keseluruhan (Lulus / Lulus bersyarat / Gagal) | |
| Jumlah uji lulus / total | |
| Defect safety yang masih terbuka | |
| Boleh lanjut ke tahap berikutnya? (Ya/Tidak + alasan) | |
| Tindak lanjut utama sesi berikutnya | |
| Paraf penguji | |

## Terkait

- [[project_review_201026]]
- [[Brush_Kart_Audit_Devils_Advocate]]
- [[ROS2_Bruskart_Robot_Project_Overview]]
- [[Operate_ROS2]]

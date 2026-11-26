# Brush Kart

Ringkasan 25 November 2026 — di side chat "Plan Brush Kart project" (10:37–12:56 WIB): user menambah 2 sensor ultrasonik di robot, Bre mengeksekusinya jadi kode dan PR, lalu user lupa cara pull dari branch dan Bre memberi panduan checkout.

## Sensor baru: 2× JSN-SR04T buat deteksi ujung kapal (10:37 WIB)

- User mengirim foto dan menjelaskan: "Gw tambahin sensor ini di depan dan belakang brushkart robot biar dia bisa deteksi mana ujung kapalnya".
- Sensornya **JSN-SR04T V3.3** — ultrasonik waterproof. Idenya menambal lubang yang Bre tulis di audit: selama ini ujung lane semi-auto cuma bisa dideteksi limit switch, artinya robot harus nabrak dulu baru tahu sudah mentok. Dengan sensor ini, robot tahu ujung kapal **sebelum** nabrak.
- Cara kerja: sensor depan menghadap arah gerak, memantulkan sinyal ke lambung kapal. Selama masih ada lambung di depan, jarak terbaca kecil dan stabil. Pas robot sampai ujung kapal, pantulannya hilang/tiba-tiba jauh — itu sinyal "ujung ketemu", waktunya berhenti/belok. Sensor belakang fungsinya sama buat gerakan mundur.

## Empat hal yang wajib benar sebelum dipasang

1. **Kecepatan suara di air ≠ di udara.** Rumus HC-SR04 biasa pakai 343 m/s; di air laut sekitar 1480 m/s. Kalau driver copas rumus udara, jarak yang dibaca meleset ~4,3×.
2. **Yang waterproof cuma transducer-nya, bukan board biru itu.** Board tetap di dalam housing kering; yang keluar cuma transducer lewat kabel.
3. **V3.3 itu kabar baik:** logika 3,3V, jadi pin Echo bisa langsung ke GPIO Raspi tanpa voltage divider (beda dengan versi 5V).
4. **"Tidak ada echo" itu sinyal, bukan error.** Di ujung kapal memang tidak ada pantulan balik. Logika node harus membedakan timeout karena ujung ketemu vs sensor mati — kalau timeout dianggap fault, robot malah berhenti di tengah lane gara-gara false alarm. Range efektif di air bakal lebih pendek dari spek 6 meter (spek itu di udara) — ekspektasi realistis 1–3 meter, jadi tes dulu di ember/kolam sebelum dibawa ke kapal.

## Eksekusi: node ultrasonik + PR ros2_ur#2 (10:44–11:03 WIB)

- User: "Gas bikin dulu" (10:44 WIB). Bre membuat branch baru **`bre/ultrasonic` dari `bre/fase-0`** (UR saja), supaya diff-nya bersih dan hanya berisi sensor ini.
- **[ros2_ur PR #2](https://github.com/skwnpro/ros2_ur/pull/2)** — commit `27dd623`, base `bre/fase-0` (bukan main). Urutan merge: **Fase-0 dulu (PR #1), baru yang ini.**
- Yang dibuat:
  - Node baru **`ultrasonic_node`** di package `brushkart_sensor` — baca 2 sensor mode Trig/Echo GPIO, berurutan (biar tidak crosstalk), 5 Hz.
  - Topic **`/range_front` & `/range_back`** pakai `sensor_msgs/Range` — tidak menyentuh kontrak message Fase-0 sama sekali.
  - **Kecepatan suara air 1480 m/s** jadi parameter (`sound_speed_m_s`) — kalau di laut tinggal kalibrasi ke ~1500–1530.
  - **`range = -1.0` = tidak ada pantulan = ujung kapal** — dikunci sebagai kondisi valid di kode, bukan fault. Nanti state machine semi-auto yang membaca ini buat tahu kapan berhenti/belok.
  - **Pin:** depan Trig 17 / Echo 27, belakang Trig 20 / Echo 21. Didaftarkan di setup.py, package.xml, launch UR, README, plus DAFTAR_PERUBAHAN.md bagian 8.

## Panduan pull dari branch (12:56 WIB)

- User: "kalo gw mau pull dari branch gimana ya lupa" (12:56 WIB).
- **Ground station (mini PC, repo ros2_gs):** `git fetch origin` lalu `git checkout bre/fase-0`.
- **Robot (Raspi, repo ros2_ur):** `git fetch origin` lalu `git checkout bre/ultrasonic` — karena branch ultrasonic numpang di atas Fase-0, di dalamnya sudah ada semua perbaikan Fase-0 **plus** sensornya.
- Abis checkout, **wajib rebuild** di dua mesin (`colcon build --symlink-install`) karena message-nya berubah struktur.
- **Pasangan wajib dijaga:** GS `bre/fase-0` + robot `bre/ultrasonic` — kalau satu main satu branch, message-nya tidak nyambung. Cek branch aktif dengan `git branch --show-current`; balik ke `main` kapan saja dengan `git checkout main`.
- Alternatif yang lebih rapi kalau sudah yakin isinya: merge PR-nya di GitHub (PR #1 di dua repo, terus PR #2), abis itu di dua mesin tinggal `git checkout main && git pull`.

## Belum terverifikasi / masih terbuka

- **State machine semi-auto belum membaca topic ini** (jatah Fase 3) — sensor sudah publish, tapi belum ada yang mengonsumsi datanya buat keputusan berhenti/belok.
- **Angka jarak jangan dipercaya 100%** sebelum tes di ember/kolam — kalibrasi kecepatan suara air laut + cek range efektifnya di air wajib sebelum dipakai di kapal.
- Empat asumsi Fase-0 yang menunggu validasi user (posisi SB/SC, pola pasangan thruster, estimasi focal CameraInfo) masih terbuka dari ringkasan sebelumnya.

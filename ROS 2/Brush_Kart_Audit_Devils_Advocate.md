---
title: Brush Kart — Audit Devil's Advocate (Logika & Struktur)
date: 2026-10-20
tags:
  - brush-kart
  - ros2
  - audit
---

# Brush Kart — Audit Devil's Advocate: Celah Logika & Struktur

Audit lanjutan dari [[project_review_201026]], sengaja **hanya berisi temuan baru** — celah yang sudah tertulis di review pertama (GUI belum ada, crash `robot_state_node`, thruster tidak diteruskan gateway, encoder/limit switch belum dibuat, nama topic kamera beda, `brushkart_msgs` ganda, health flag hardcoded, belum ada launcher) tidak diulang di sini kecuali sebagai konteks.

Sumber: seluruh kode di `ros2_ur` + `ros2_gs` (termasuk `package.xml` semua package), diagram `ROS2_node_0_omks.json`, dan catatan harian di vault (terutama 22 & 24–25 September 2026).

## A. Logika Rantai Kendali & Safety

### A1. E-stop bukan fungsi safety — cuma tombol biasa
E-stop diimplementasikan sebagai `estop = bool(msg.buttons[0])` di joystick: **momentary, tidak latching**, dan jalurnya persis sama dengan jalur kendali normal (joy_node → joystick_interface → tether → gateway → relay). Konsekuensinya:
- Satu mata rantai mati = e-stop mati, persis di situasi dia paling dibutuhkan.
- Tombol dilepas → e-stop lepas → kalau stick masih miring, robot **langsung jalan lagi** tanpa prosedur re-arm apa pun.
- Skenario klasik: operator panik, remote terjatuh/terlepas dari tangan → tidak ada yang menahan tombol → robot bebas.
E-stop yang benar: latching (sekali ditekan, tetap stop sampai operator secara sadar me-reset), dan idealnya tidak bergantung pada kesehatan jalur data normal.

### A2. Tidak ada konsep heartbeat di protokol — "diam" tidak bisa dibedakan dari "mati"
`command_gateway_node` hanya publish saat menerima command; `relay_operator_node` hanya bereaksi saat pesan datang. Tidak ada publish periodik, dan `OperatorCommand`/`RelayOperator` tidak punya timestamp maupun nomor urut. Jadi tidak ada pihak pun yang bisa membedakan tiga keadaan yang sangat berbeda: (a) operator memang diam, (b) GS hang, (c) tether putus. Perbaikan watchdog di review pertama hanya setengah solusi — yang sebenarnya dibutuhkan adalah perubahan protokol: gateway **wajib publish terus-menerus** (mis. 10 Hz) walau tidak ada input baru, relay menganggap command basi (lebih tua dari ±300 ms) sebagai perintah berhenti.

### A3. Aktuator boleh stall di end-stop tanpa batas waktu
Relay steering/drive dapat menyala terus-menerus tanpa batas: stick ditahan, command basi, atau auto yang macet — semuanya membuat motor hidrolik menekan mentok mekanis tanpa henti. Pressure relief bekerja terus, oli panas, seal aus. Tidak ada *max-on-time* di gateway maupun di relay node. Untuk steering ini bukan skenario tepi — menahan belokan sampai mentok adalah pemakaian normal.

### A4. Mapping joystick rapuh persis di titik kritis
- `brush_enable` memakai `msg.axes[4] == -1` — kesetaraan float **persis**. Trim bergeser sedikit saja, brush tidak pernah nyala; noise ke arah sebaliknya, brush nyala sendiri.
- Drive memakai axis yang dipetakan dari throttle — bila channel-nya non-centering (gaya throttle radio), perintah drive **tetap ada saat tangan lepas**.
- Urutan channel Radiomaster Boxer (AETR/TAER dan reverse per channel) adalah setting firmware radio yang tidak tercatat di repo mana pun. Ganti profil radio atau update firmware → mapping berubah diam-diam, dan rantai tanda (radio → negasi di joystick node → perbandingan di gateway) tidak punya satu pun titik yang dinyatakan sebagai sumber kebenaran.

### A5. Sinyal proporsional dibuang jadi hentakan
Command float dari joystick dipotong deadband 0,30 lalu langsung full power: 30% travel stick mati, sisanya on/off penuh. Setiap start/stop adalah hentakan hidrolik (pressure spike) ke selang dan struktur. Relay memang tidak bisa proporsional penuh, tapi konsekuensi ini belum pernah ditulis sebagai keputusan desain — dan mitigasi murah ada (deadband lebih kecil, pulsing sederhana di gateway untuk start halus).

## B. Sensor & Kebenaran Data

### B1. Varian MS5803 patut dicurigai salah pilih — dan depth adalah fondasi semi-auto
Catatan 24–25 September menulis sensor yang terdeteksi adalah **MS5803-01BA**. Varian 01BA ber-range **0–1 bar absolut**. Di bawah air, tekanan absolut ≈ 1 bar (atmosfer) + ±0,1 bar per meter — artinya sensor jenuh praktis **di permukaan air** dan depth tidak akan pernah terbaca benar. Perlu verifikasi fisik (silkscreen modul GY-MS5803):
- Kalau benar 01BA → wajib ganti varian (mis. 05BA/14BA atau MS5837-30BA), dan asumsi "Depth reach?" di flowchart semi-auto ikut gugur sampai diganti.
- Dua error tambahan yang tetap ada di varian apa pun: referensi permukaan dikunci 1013,25 mbar (tekanan udara nyata berayun ±30 mbar ≈ ±30 cm error depth) dan konstanta konversi /100 (akurasi ±2–3% tergantung salinitas).
- Pertanyaan fisik yang belum terjawab di dokumen mana pun: sensor ini **terekspos air (wetted)** atau di dalam housing kedap? Kalau di dalam housing, yang diukur adalah tekanan internal housing — depth-nya fiksi.

### B2. BNO055 di Raspberry Pi adalah jebakan I2C yang terkenal — dan tidak ada pertahanan apa pun
BNO055 notorious soal *clock stretching* yang membuat bus I2C BCM2835 (Raspberry Pi) hang. Driver saat ini membaca 18 register satu byte per transaksi, tanpa timeout, tanpa recovery, tanpa try/except: sekali bus hang, `sensor_node` mati total dan diam-diam. Dua hal lain:
- Status kalibrasi NDOF tidak pernah dibaca, apalagi disimpan antar-boot. Setiap restart, kalibrasi mulai dari nol — dan robot di dalam air tidak bisa melakukan gerakan kalibrasi.
- Yaw BNO055 bergantung pada magnetometer. Di samping **lambung baja dan motor hidrolik**, heading magnetik adalah referensi yang paling tidak bisa dipercaya — persis di lingkungan kerja robot ini. Strategi "Turn 15 degree" untuk semi-auto jangan memakai yaw absolut; pakai integrasi gyro relatif per manuver.

### B3. sensor_node adalah single point of failure yang paling rapuh
Dua driver membuka `SMBus(1)` secara terpisah; pembacaan MS5803 blocking ±100 ms (dua konversi + sleep) di dalam timer callback 10 Hz — laju efektif ±5 Hz dengan jitter, dan satu glitch I2C membunuh node yang menjadi satu-satunya sumber telemetri seluruh sistem. Tidak ada isolasi error per sensor: pressure gagal = IMU ikut hilang dari topic.

## C. Arsitektur & Struktur Repo

### C1. Otak safety duduk di sisi yang salah dari tether
`command_gateway_node` — satu-satunya tempat logika keselamatan (deadband, e-stop, penerjemahan perintah ke relay) — berada di repo ground station, sementara `relay_operator_node` di robot tidak punya intelijen sama sekali: dia tidak bisa membedakan perintah sah dari keheningan. Pembagian repo mengikuti "kode ditulis di mesin mana", bukan domain kegagalan. Begitu robot suatu hari harus bereaksi lokal terhadap fault (sensor mati, tekanan abnormal), logikanya ada di seberang kabel. Arah yang lebih sehat: gateway pindah ke sisi robot; ground station murni input + tampilan.

### C2. Deklarasi dependensi package.xml fiktif di 4 dari 9 package — deploy bersih akan meledak di sini
- `brushkart_sensor` depend ke **`bruhskart_msgs`** (typo — package itu tidak ada).
- `brushkart_operator` depend ke **`brushkart_interfaces`** (tidak ada di repo mana pun).
- `brushkart_network` depend ke **`brushkart_utils`** (juga tidak ada).
- `brushkart_joystick` malah **tidak mendeklarasikan `brushkart_msgs`** sama sekali, padahal node-nya mengimpor package itu.
- Dependensi Python runtime yang nyata — `psutil` (network), `smbus2` (sensor), `RPi.GPIO` (relay) — **tidak dideklarasikan di mana pun**.
Build hari ini berhasil karena keberuntungan urutan alfabet dan karena kedua mesin diprovison manual sedikit demi sedikit. `rosdep install` dari mesin bersih akan gagal, dan urutan build paralel tidak dijamin. Ini akan menjadi tembok pertama yang ditabrak rencana deploy ke robot simulasi.

### C3. Dua package bernama identik menunggu konflik satu mesin
Selain risiko divergensi isi (review pertama), nama `brushkart_msgs` yang ada di dua workspace akan konflik begitu kedua workspace disource atau digabung di satu mesin — skenario yang sangat wajar saat bench-test robot simulasi di satu laptop.

### C4. Nol konfigurasi: semua konstanta hardcoded, mapping pin hanya hidup sebagai komentar
IP ground station, pin GPIO, deadband, indeks kamera, resolusi, FPS — semuanya tertanam di kode. Tidak ada file parameter, tidak ada launch, tidak ada config. Menyetel robot di pinggir kolam berarti edit kode → rebuild → redeploy. Mapping pin ↔ katup hidrolik hanya ada sebagai komentar di `relay_operator_node.py`; kalau kabel dipindah, komentarnya bohong dan tidak ada sumber kebenaran lain.

### C5. Tidak ada waktu sinkron, tidak ada rekaman — robot tak terlihat didebug buta
Dua mesin tanpa sinkronisasi waktu (NTP/chrony) yang terdokumentasi: timestamp antar-mesin tidak bisa dikorelasikan saat debugging integrasi — fase yang justru akan dijalani berikutnya. Dan tidak ada rencana `rosbag`: untuk robot yang tidak bisa diamati saat bekerja, merekam `/sensor_data` + `/robot_command` + `/relay_operator` di setiap tes adalah satu-satunya cara post-mortem.

### C6. DDS tanpa konfigurasi, dan network_monitor mengukur arah yang salah
`ROS_DOMAIN_ID`/RMW tidak diatur di mana pun — mesin dev lain di jaringan yang sama bisa saling melihat atau bertabrakan domain secara diam-diam. Sementara itu `network_monitor_node` (di robot) mengukur kesehatan arah robot→GS — arah yang menurut definisi sehat begitu pesannya terkirim. Arah yang mematikan, GS→robot untuk perintah, tidak pernah diukur siapa pun.

### C7. Kamera membebani tether dua kali untuk penonton yang belum ada
`camera_node` mempublikasikan raw (320×240 bgr8 ≈ 18 Mbps) **dan** compressed sekaligus, dengan QoS RELIABLE, di tether yang sama dengan jalur kendali. Raw stream tidak dikonsumsi siapa pun. Untuk video, RELIABLE berarti head-of-line blocking: frame lama menumpuk, latency naik, dan GUI (kapan pun dibuat) akan menampilkan masa lalu. Link ke GS cukup satu stream compressed dengan BEST_EFFORT.

### C8. Higiene: tidak ada test logika, README kosong, sisa file sampah
Semua folder `test/` isinya linter bawaan template — nol test untuk logika deadband gateway atau health-check robot_state (crash di review pertama adalah bukti harga dari ketiadaan ini). README `ros2_gs` kosong total, `ros2_ur` satu baris. Deskripsi/maintainer/license masih "TODO" di hampir semua package, dan ada `package.xml` terpotong (fragmen tidak valid) terselip di dalam folder modul `brushkart_joystick`. Tidak ada CI.

## D. Desain Semi-Auto (Flowchart) — Sebelum Jadi Kode

### D1. Flowchart-nya sendiri ambigu
Dari file JSON-nya, flowchart terdiri dari dua loop cermin (konektor A/B) dengan node Stop/Turn ganda dan alur yang tidak bisa dibaca sebagai state machine deterministik. Menulisnya ulang sebagai tabel state–transisi (state, guard, aksi, timeout, transisi gagal) adalah pekerjaan desain yang harus selesai **sebelum** satu baris pun state machine dikoding.

### D2. Geometri sapuan tidak terdefinisi; angka 15° melayang
"Depth reach?" sebagai satu-satunya syarat berhenti mengasumsikan lane vertikal; flowchart tidak pernah mendefinisikan orientasi lane, lebar sapuan (= lebar brush?), dan hubungan sudut 15° dengan jarak antar-lane. Tanpa itu, hasil sapuan akan punya celah tak tersentuh atau tumpang tindih besar — dan tidak ada cara memverifikasinya.

### D3. Tabrak dulu, baru tahu — tanpa deteksi stall
Limit switch adalah satu-satunya deteksi ujung: robot menabrak apa pun di jalurnya dengan kecepatan penuh — termasuk benda yang tidak boleh ditabrak (propeller, rudder, sea chest, anode). Tidak ada perlambatan saat mendekat, dan tidak ada deteksi "motor menyala tapi robot tidak bergerak" (brush nyangkut, robot selip): flowchart bisa menggiling satu titik selamanya.

### D4. Tidak ada pintu keluar dari mode auto
"Input Mode" hanya muncul sekali di START. Sekali auto berjalan, tidak ada transisi override manual atau abort selain mencapai END secara alami. Untuk sistem yang masih dites, itu tidak bisa diterima — abort harus bisa dilakukan operator kapan pun, dan harus menjadi transisi kelas satu di state machine.

## Urutan Perbaikan (Prioritas)

### P0 — Sebelum tes gerak berikutnya
1. **Rancang ulang rantai stop sebagai protokol** (A1+A2+A3): gateway publish periodik (heartbeat), relay menolak command basi & timeout → semua relay mati, e-stop latching dengan re-arm sadar, max-on-time untuk steering & drive.
2. **Verifikasi MS5803** (B1): cek silkscreen varian + cara pemasangan (wetted?). Murah, cepat, dan bisa membatalkan asumsi depth seluruh desain.
3. **Kunci mapping joystick** (A4): ganti kesetaraan float persis dengan threshold, dokumentasikan channel map Boxer + konvensi tanda sebagai satu sumber kebenaran.

### P1 — Fondasi deploy & data (sebelum Fase 1 dan robot simulasi)
4. **Bereskan package.xml + deklarasikan semua dependensi Python** (C2) — prasyarat deploy bersih.
5. **Pindahkan konstanta ke parameter/config** (C4): pin, IP, deadband, kamera; jadikan mapping pin↔katup dokumen resmi.
6. **Putuskan relokasi command_gateway ke sisi robot** (C1) — semakin lama ditunda, semakin banyak kode mengasumsikan posisi sekarang.
7. **Hardening jalur sensor** (B2+B3): isolasi error per driver, timeout/recovery I2C, pantau status kalibrasi BNO055, strategi heading relatif untuk manuver belok.

### P2 — Spesifikasi sebelum semi-auto ditulis
8. **Tulis ulang flowchart menjadi state machine tertulis** (D1–D4): geometri lane & asal-usul 15°, deteksi stall, perlambatan ujung, transisi abort/override operator.

### P3 — Workflow & kebersihan
9. **Sinkronisasi waktu (chrony) + biasakan rosbag di setiap tes** (C5).
10. **Konfigurasi DDS/domain + kamera satu stream compressed BEST_EFFORT** (C6+C7).
11. **Satu sumber `brushkart_msgs`, README cara menjalankan, test logika minimal (gateway + robot_state), buang file stray** (C3+C8).

## Terkait

- [[project_review_201026]]
- [[ROS2_Bruskart_Robot_Project_Overview]]
- [[joystick_interface_node]]
- [[n_sensor_node]]
- [[24_25_September]]
- [[22_September_2026]]

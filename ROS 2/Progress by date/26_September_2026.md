---
title: 26 September 2026
date: 2026-09-26
---

# 26 September 2026

## Rangkuman Progress Kamera IMX290 — 26 Sept 2026

### Tabel Progress Troubleshooting

| Tahap | Hasil Awal | Hasil Akhir | Status |
|-------|------------|-------------|--------|
| Deteksi CSI camera | /dev/video0 tidak ada | /dev/video0 muncul | ✅ |
| Driver IMX290 | Tidak terdeteksi | imx290 10-001a terdeteksi | ✅ |
| Unicam driver | Tidak muncul | unicam-image aktif | ✅ |
| Overlay IMX290 | Tidak ter-load otomatis | Berhasil load manual | ✅ |
| Media controller | Tidak ada sensor | IMX290 masuk media graph | ✅ |
| OpenCV capture | Belum test | ret=True | ✅ |
| Capture JPG | Belum bisa | test.jpg berhasil dibuat | ✅ |
| FPS kamera | Belum diketahui | ~26.3 FPS | ✅ |
| ROS2 package | Belum ada | brushkart_camera dibuat | ✅ |
| Camera node | Belum ada | Publish topic berhasil | ✅ |
| ROS topic | Tidak ada | /camera/raw_image muncul | ✅ |
| DDS Ethernet | Belum test | Topic terlihat di Mini PC | ✅ |
| Topic rate | Belum ada | ~19 Hz | ✅ |
| RViz2 | Tidak ada image | No image | ❌ |
| rqt_image_view | Tidak ada image | No image | ❌ |
| Pixel data kamera | Belum diketahui | Semua pixel = 0 | ❌ |

### Tabel Hasil Investigasi Teknis

| Item | Hasil |
|------|-------|
| Camera model | Arducam B0424 IMX290 Starvis + motorized IR-cut |
| OS Raspi | Ubuntu Server 24.04.5 |
| ROS version | ROS2 Jazzy |
| Kernel | 6.8.0-1065-raspi |
| Deteksi libcamera | Berhasil |
| Deteksi cam -l | Berhasil |
| Tuning file | imx290.json ditemukan |
| Sensor address | imx290@1a |
| Subdevice | /dev/v4l-subdev0 |
| Video device | /dev/video0 |
| Resolution | 1920x1080 |
| Pixel format | BGR3 |
| Raw frame size | 6 MB/frame |
| Stream 10 frame | 60 MB |
| OpenCV mean pixel | 0.0 |
| OpenCV min pixel | 0 |
| OpenCV max pixel | 0 |
| RAW dump | Seluruh byte = 0x00 |

### Tabel Masalah yang Berhasil Diselesaikan

| Masalah | Penyebab | Solusi | Status |
|---------|----------|--------|--------|
| IMX290 tidak terdeteksi / /dev/video0 tidak muncul | Overlay tidak aktif, unicam belum aktif | Load dtoverlay imx290, load overlay IMX290 | ✅ |
| OpenCV tidak bisa akses kamera | Device belum tersedia | /dev/video0 berhasil muncul | ✅ |
| ROS2 topic tidak ada | Camera node belum dibuat | Buat camera_node.py | ✅ |
| Topic tidak terlihat di Mini PC | Belum publish | Publish ROS2 image | ✅ |
| DDS belum terverifikasi | Belum ada data | Topic 19 Hz terlihat | ✅ |
| QoS mismatch dengan RViz | Reliability berbeda | Ditemukan penyebabnya | ✅ (belum diterapkan permanen) |

### Tabel Masalah yang Masih Terbuka

| No | Masalah | Kondisi Saat Ini |
|----|---------|------------------|
| 1 | Frame kamera hitam | Mean=0, Max=0 |
| 2 | Raw capture berisi nol | Semua byte = 0x00 |
| 3 | RViz tidak tampil gambar | Karena frame kosong |
| 4 | rqt_image_view blank | Karena frame kosong |
| 5 | Root cause pixel kosong | Belum ditemukan |

### Progress Keseluruhan

| Sub sistem | Progress |
|------------|----------|
| Hardware kamera | 100% |
| Driver kamera | 100% |
| Libcamera detection | 100% |
| ROS2 package | 100% |
| ROS2 publisher | 100% |
| DDS communication | 100% |
| Ground Station visibility | 100% |
| Valid image streaming | 20% |
| End-to-end camera system | 80% |

**Total progress: 80%**

### To Do List Hari Senin

**Prioritas 1 — Validasi libcamera capture:**
- Cek seluruh executable libcamera tersedia
- Test capture image menggunakan libcamera
- Test capture image menggunakan cam

**Prioritas 2 — Investigasi pixel data kosong:**
- Analisis pipeline IMX290 → unicam → ISP
- Cek kemungkinan driver Arducam khusus
- Verifikasi ISP menghasilkan frame valid
- Target: mean > 0, max > 0

**Prioritas 3 — Kamera menampilkan gambar valid:**
- OpenCV menghasilkan gambar sebenarnya
- Simpan JPG valid
- Verifikasi warna dan exposure

**Prioritas 4 — Finalisasi ROS2 camera node:**
- Publish valid image frame (bukan frame hitam)
- Hilangkan warning QoS
- Tampilkan image di Raspi, viewer tampil

**Prioritas 5 — Streaming ke Ground Station:**
- DDS kirim image valid, frame terlihat di Mini PC
- RViz2 tampil realtime, video live
- rqt_image_view tampil realtime, video live

**Prioritas 6 — Optimasi robot operasional:**
- Tambah compressed transport (hemat bandwidth)
- Buat topic /camera/compressed (operasional robot)
- Uji latency Ethernet tether (Ground Station siap deploy)

Arsitektur target:
```
IMX290 → camera_node → /camera/compressed → DDS Ethernet → Mini PC Ground Station → RViz2 / GUI
```

### Goal Hari Senin

Target minimum:
- ✅ IMX290 menghasilkan pixel valid
- ✅ Mean pixel > 0
- ✅ RViz menampilkan gambar
- ✅ rqt_image_view menampilkan gambar

Target ideal:
- ✅ Live video IMX290
- ✅ Streaming ke Mini PC
- ✅ Siap integrasi Ground Station BrushKart

**Kesimpulan:** hari ini progres sangat besar. Dari kondisi awal kamera sama sekali tidak terdeteksi, sekarang IMX290 sudah terdeteksi, ROS2 node sudah publish, DDS sudah mengirim ke Ground Station. Bottleneck terakhir tinggal satu: **pixel data dari pipeline kamera masih bernilai 0 (frame hitam)**. Setelah itu beres, streaming kamera ke Mini PC hampir pasti selesai.

## Terkait

**Sebelum / sesudah:**

- [[24_25_September]]
- [[28_September_2026]]

**Dokumen terkait:**

- [[ROS2_Bruskart_Robot_Project_Overview]]

---
title: 28 September 2026
date: 2026-09-28
---

# 28 September 2026

## Summary Troubleshooting Kamera IMX290 (26–28 Sept 2026)

### Goal Project

```
IMX290 CSI
    ↓
Raspberry Pi 4B — Ubuntu Server 24.04
    ↓
ROS2 Jazzy
    ↓
DDS Ethernet
    ↓
Mini PC Ground Station
    ↓
RViz / rqt_image_view
```

Status akhir: ✅ Kamera tampil di Ground Station, ✅ ROS2 publish berjalan, ✅ DDS berjalan, ✅ topic camera berhasil diterima Mini PC, ✅ IMX290 berhasil digunakan.

### 1. Tabel Problem & Solusi

| No | Problem | Gejala | Solusi | Hasil |
|----|---------|--------|--------|-------|
| 1 | Kamera tidak terdeteksi | /dev/video0 tidak ada | Troubleshoot overlay IMX290 | ✅ |
| 2 | IMX290 tidak muncul di dmesg | `dmesg \| grep imx` kosong | Load overlay manual | ✅ |
| 3 | Overlay tidak aktif saat boot | `dtoverlay -l` = No overlays loaded | `sudo dtoverlay imx290` | ✅ |
| 4 | Sensor tidak muncul di media graph | Tidak ada unicam | Load overlay IMX290 | ✅ |
| 5 | Libcamera tidak terlihat | `libcamera-hello` tidak ada | Gunakan `cam -l` | ✅ |
| 6 | OpenCV mean = 0 | Frame hitam total | Gunakan libcamerify | ✅ |
| 7 | ROS2 topic tampil tapi blank | RViz hitam | Validasi pixel image | ✅ |
| 8 | QoS mismatch | RViz warning RELIABILITY | Ubah QoS RELIABLE | ✅ |
| 9 | V4L2 camera hijau | Warna salah | Stop pakai v4l2_camera | ✅ diketahui |
| 10 | Lag/freeze di RViz | Queue penuh | Resize image | ✅ |
| 11 | Compressed image tidak tampil di RViz | Pink screen | Karena camera display butuh CameraInfo | ✅ diketahui |
| 12 | FPS drop 10→1→10 | OpenCV streaming tidak stabil | Gunakan CAP_V4L2 + warmup frame | ✅ membaik |

### 2. Command Troubleshooting yang Digunakan

**Identifikasi kamera:**
```bash
ls /dev/video*
sudo dmesg | grep -i imx
sudo dmesg | grep -i unicam
sudo dmesg | grep -i media
```
Kondisi awal: ❌ /dev/video0 tidak ada, ❌ imx290 tidak terdeteksi, ❌ unicam tidak muncul, ❌ ROS tidak bisa digunakan.

**Cek device tree & overlay:**
```bash
cat /boot/firmware/config.txt
sudo dtoverlay -l
sudo dtoverlay imx290
ls /boot/firmware/overlays | grep imx
```

**Identifikasi sensor:**
```bash
media-ctl -p -d /dev/media4
# Output penting: imx290 10-001a, unicam-image, /dev/video0
```

**I2C kamera:**
```bash
i2cdetect -l
sudo i2cdetect -y 10
```

**Test OpenCV:**
```python
cap = cv2.VideoCapture(0)
ret, frame = cap.read()
print(frame.mean())
```
Awal: mean = 0.

**Test libcamera:**
```bash
cam -l
# Output: Available cameras: imx290
```

**Capture libcamera:**
```bash
cam -c1 --capture=1 --file=test.ppm
# Output: test.ppm, gambar valid
```

**Test OpenCV dengan libcamera:**
```bash
libcamerify python3 test_opencv.py
# Output: Mean = 79, Max = 255 — ini breakthrough terbesar
```

### Cara Jalanin dtoverlay imx290 Manual

1. ```bash
   sudo dtoverlay imx290
   ```
2. Cek apakah sudah terload:
   ```bash
   lsmod | grep imx
   ```
   Kalau kosong, ketik:
   ```bash
   sudo modprobe imx290
   ```
3. Cek overlay aktif:
   ```bash
   sudo dtoverlay -l
   ```
4. Cek sensor setelah driver hidup:
   ```bash
   media-ctl -p -d /dev/media4
   ```

### 3. Installed Package / Library

**Kamera:**
```bash
sudo apt install v4l-utils
sudo apt install media-ctl
sudo apt install i2c-tools
```

**Libcamera:**
```bash
sudo apt install libcamera-tools
sudo apt install gstreamer1.0-libcamera
```

**OpenCV:**
```bash
sudo apt install python3-opencv
```

**ROS2 camera:**
```bash
sudo apt install ros-jazzy-cv-bridge
sudo apt install ros-jazzy-image-transport
sudo apt install ros-jazzy-vision-opencv
sudo apt install ros-jazzy-v4l2-camera
```

**Viewer:**
```bash
sudo apt install ros-jazzy-rviz2
sudo apt install ros-jazzy-rqt-image-view
```

### 4. Urutan Milestone

| Tahap | Status |
|-------|--------|
| IMX290 terpasang | ✅ |
| Overlay IMX290 ditemukan | ✅ |
| Overlay berhasil diload | ✅ |
| IMX290 muncul di media graph | ✅ |
| Unicam aktif | ✅ |
| /dev/video0 muncul | ✅ |
| OpenCV berhasil membuka kamera | ✅ |
| Libcamera berhasil mendeteksi kamera | ✅ |
| Test image berhasil dibuat | ✅ |
| ROS2 camera_node dibuat | ✅ |
| /camera/raw_image publish | ✅ |
| DDS Ethernet berhasil | ✅ |
| Topic terlihat di Mini PC | ✅ |
| RViz tampil gambar | ✅ |
| Compressed image publish | ✅ |
| Monitoring kamera Ground Station | ✅ |

### 5. Evolusi Kamera

- **Awal:** ❌ /dev/video0 tidak ada, ❌ imx290 tidak terdeteksi, ❌ unicam tidak muncul, ❌ ROS tidak bisa digunakan
- **Tengah:** ✅ IMX290 detect, ✅ unicam detect, ✅ /dev/video0 ada, ❌ OpenCV menghasilkan frame hitam, ✅ test.ppm valid
- **Setelah libcamera:** ✅ `cam -l` menemukan IMX290, ❌ OpenCV biasa masih hitam
- **Setelah libcamerify:** ✅ mean > 0, ✅ pixel valid, ✅ image valid, ✅ ROS image valid
- **Akhir:** ✅ kamera tampil di Ground Station, ✅ DDS berjalan, ✅ ROS2 berjalan, ✅ monitoring operator berhasil

### 6. Setting Final yang Saat Ini Paling Stabil

| Parameter | Nilai |
|-----------|-------|
| Camera | IMX290 CSI |
| Capture method | libcamerify + OpenCV |
| Resolution | 320x240 |
| FPS | 5 (0.2 setting program) |
| JPEG quality | 70 |
| Raw topic | /camera/raw_image |
| Compressed topic | /camera/compressed |
| DDS | Ethernet |
| Viewer | RViz |

### 7. Struktur Node Final

```
IMX290 → libcamera → libcamerify → OpenCV → camera_node
    → /camera/raw_image, /camera/compressed
    → DDS Ethernet
    → Mini PC Ground Station → RViz2
```

### Progress Akhir

| Sistem | Progress |
|--------|----------|
| Hardware kamera | 100% |
| Driver IMX290 | 100% |
| Libcamera | 100% |
| ROS2 camera node | 100% |
| DDS Ethernet | 100% |
| Ground Station receive | 100% |
| Video monitoring | 100% |
| Optimasi FPS | 80% |
| Optimasi compressed | 80% |
| Overall | 100% |

## Terkait

**Sebelum / sesudah:**

- [[26_September_2026]]
- [[30_September_2026]]

**Dokumen terkait:**

- [[ROS2_Bruskart_Robot_Project_Overview]]

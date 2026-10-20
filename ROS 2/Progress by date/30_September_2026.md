---
title: 30 September 2026
date: 2026-09-30
---

# 30 September 2026

## Summary Chat 30-09-2026 — Network Monitor Node (Raspberry Pi → Ground Station)

### Objective

Membuat node ROS2 `network_monitor_node` yang berjalan di Raspberry Pi untuk memonitor kualitas koneksi ke Mini PC dan publish ke `/network_status` menggunakan message `brushkart_msgs/msg/NetworkStatus`.

### 1. List Problem dan Solusi

| No | Problem | Gejala / Error | Penyebab | Solusi | Status |
|----|---------|----------------|----------|--------|--------|
| 1 | Tidak bisa install psutil | `externally-managed-environment` | Ubuntu 24.04 menggunakan PEP668, bukan pip | Install dari apt: `sudo apt install python3-psutil` | ✅ |
| 2 | ROS2 node gagal dijalankan | `ModuleNotFoundError: No module named 'brushkart_network.network_monitor_node'` | File dibuat tanpa ekstensi `.py` | Rename file menjadi `network_monitor_node.py` | ✅ |
| 3 | Node jalan tapi log hanya menampilkan latency | Hanya muncul `Latency=5.0` | Logger hanya mencetak satu field | Ubah logger agar menampilkan seluruh field | ✅ |
| 4 | Topic echo gagal | `The message type 'brushkart_msgs/msg/NetworkStatus' is invalid` | Package message tidak dikenali / environment belum source workspace | Source workspace ulang dan validasi interface | ✅ |
| 5 | Interface tidak dikenali | `Unknown package 'brushkart_msgs'` | Environment ROS2 belum memuat package message | Source kembali workspace terbaru | ✅ |

**Detail command per problem:**

Problem 1 — psutil gagal install:
```bash
pip install psutil
# Output: externally-managed-environment
# Solusi:
sudo apt update
sudo apt install python3-psutil
# Verifikasi:
python3 -c "import psutil; print(psutil.__version__)"
```

Problem 2 — node tidak bisa run:
```bash
cd ~/ros2_ur/ros2_ws/src/brushkart_network/brushkart_network
ls
# Output: __init__.py, network_monitor_node   <- tanpa .py!
# ROS2 mencari: brushkart_network.network_monitor_node
# Solusi:
mv network_monitor_node network_monitor_node.py
```

Problem 3 — data yang muncul hanya latency:
```python
# Penyebab: logger hanya mencetak
msg.latency_ms
# Solusi: ganti logger menjadi seluruh field
```
Contoh output:
```
Connected=True
Latency=5ms
Loss=0%
TX=500kbps
RX=1200kbps
CPU=15%
RAM=30%
```

Problem 4 — topic echo gagal:
```bash
ros2 topic echo /network_status
# Output: The message type 'brushkart_msgs/msg/NetworkStatus' is invalid
# Investigasi: topic ada (ros2 topic list menampilkan /network_status),
# publisher ada (ros2 topic info -> count: 1, node aktif).
# Kesimpulan: node berhasil publish, masalah ada di environment/message registration.
```

Problem 5 — interface tidak dikenali:
```bash
ros2 interface show brushkart_msgs/msg/NetworkStatus
# Output: Unknown package 'brushkart_msgs'
# Dugaan: terminal belum source workspace terbaru
# Solusi:
source /opt/ros/jazzy/setup.bash
source ~/ros2_ur/ros2_ws/install/setup.bash
ros2 pkg list | grep brushkart
ros2 interface list | grep NetworkStatus
ros2 topic echo /network_status
# Jika sudah berhasil echo: cabut kabel ethernet untuk validasi
# Target: connected=false, packet_loss_percent=100, last_rx_ms meningkat
```

### 2. Installed Library / Package

**Python library:**

| Library | Fungsi | Status | Install Command |
|---------|--------|--------|-----------------|
| psutil | Monitoring CPU, RAM, network usage | ✅ Installed | `sudo apt install python3-psutil` |

**ROS2 package:**

| Package | Fungsi | Status |
|---------|--------|--------|
| brushkart_camera | Publish kamera | ✅ |
| brushkart_sensor | Publish sensor | ✅ |
| brushkart_gateway | Gateway komunikasi | ✅ |
| brushkart_msgs | Custom ROS2 message | ✅ |
| brushkart_network | Network monitor node | ✅ |

**Clean build:**
```bash
cd ~/ros2_ur/ros2_ws
rm -rf build install log
colcon build
source install/setup.bash
```

**Run:**
```bash
ros2 run brushkart_network network_monitor_node
# Hasil: ✅ node berhasil hidup
```

**NetworkStatus.msg:**
```
bool connected
float32 latency_ms
float32 packet_loss_percent
uint32 last_rx_ms
float32 tx_kbps
float32 rx_kbps
float32 cpu_usage
float32 ram_usage
```

### 3. Urutan Milestone

| Milestone | Deskripsi | Status |
|-----------|-----------|--------|
| M1 | Menentukan data yang akan dimonitor | ✅ |
| M2 | Mendesain NetworkStatus.msg | ✅ |
| M3 | Menambahkan NetworkStatus ke brushkart_msgs | ✅ |
| M4 | Build package message | ✅ |
| M5 | Membuat package brushkart_network | ✅ |
| M6 | Membuat node dummy publisher | ✅ |
| M7 | Menjalankan node dummy | ✅ |
| M8 | Membuat monitoring ping | ✅ |
| M9 | Membuat monitoring packet loss | ✅ |
| M10 | Membuat monitoring CPU usage | ✅ |
| M11 | Membuat monitoring RAM usage | ✅ |
| M12 | Membuat monitoring bandwidth TX/RX | ✅ |
| M13 | Integrasi seluruh data ke ROS2 node | ✅ |
| M14 | Validasi topic echo | (proses) |
| M15 | Integrasi robot_state_node | ⏳ |
| M16 | Integrasi Ground Station GUI | ⏳ |

### 4. Arsitektur Final Network Node

```
brushkart_network
        ▼
network_monitor_node ──publish──→ /network_status ──→ robot_state_node
```

### 5. Data yang Dipublish

| Data | Sumber | Fungsi |
|------|--------|--------|
| connected | Ping result | Status koneksi |
| latency_ms | Ping RTT | Monitoring delay |
| packet_loss_percent | Ping | Monitoring kualitas link |
| last_rx_ms | Internal counter | Lama koneksi terputus |
| tx_kbps | psutil.net_io_counters | Traffic keluar |
| rx_kbps | psutil.net_io_counters | Traffic masuk |
| cpu_usage | psutil.cpu_percent | Beban Raspberry Pi 4 |
| ram_usage | psutil.virtual_memory | Penggunaan RAM |

### Status Project Saat Ini

| Modul | Progress |
|-------|----------|
| Camera Node | ✅ 100% |
| Sensor Node | ✅ 100% |
| Network Node Logic | ✅ 95% |
| Message Definition | ✅ 100% |
| Topic Validation | 90% |
| Robot State Node | ⏳ 0% |
| Ground Station Dashboard | ⏳ 0% |

**Next action (urutan pengerjaan):** integrasi ke robot_state_node.

## Terkait

**Sebelum / sesudah:**

- [[28_September_2026]]
- [[5_October_2026]]

**Dokumen terkait:**

- [[Project_Overview]]
- [[Connection_Ethernet]]
- [[5_October_2026]]

---
title: Connection Ethernet
date: 2026-09-22
---

# Connection Ethernet

## Troubleshooting SSH Raspberry Pi 4B via Tether Ethernet (Tanpa Internet)

### Latar Belakang

Sistem Brush Kart menggunakan arsitektur:

- Ground Station: Mini PC + Ubuntu Desktop 24.04
- Robot: Raspberry Pi 4B + Ubuntu Server 24.04
- Middleware: ROS 2 Jazzy Jalisco
- Koneksi komunikasi: tether Ethernet langsung antara Mini PC dan Raspberry Pi
- SSH digunakan untuk akses ke Raspberry Pi dari Ground Station

Arsitektur komunikasi memang dirancang menggunakan Ethernet antara Ground Station dan Robot.

### Hasil Akhir

- ✅ Ethernet Raspberry Pi terdeteksi
- ✅ Ethernet Mini PC terdeteksi
- ✅ Link Gigabit aktif (1000 Mbps)
- ✅ Kabel tether normal
- ✅ SSH berhasil digunakan tanpa koneksi internet
- ✅ Arsitektur komunikasi Ground Station ↔ Robot melalui Ethernet dapat digunakan sesuai desain sistem Brush Kart

### Cara Cek Koneksi Jika SSH Down

1. Ping dari Mini PC:
   ```bash
   ping 192.168.10.2
   ```
2. Cek port 22 dari Mini PC:
   ```bash
   nc -zv 192.168.10.2 22
   # atau
   nmap -p 22 192.168.10.2
   ```
3. Cek IP Raspi berubah atau tidak:
   ```bash
   ip addr
   hostname -I
   ```
4. Kalau SSH mati:
   ```bash
   sudo systemctl status ssh
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```
5. Cek kalau interface Ethernet masih up:
   ```bash
   ip link   # cari eth0, harus up
   sudo ip link set eth0 up   # kalau down
   ```
6. Cek konfigurasi netplan:
   ```bash
   ls /etc/netplan
   sudo nano /etc/netplan/*.yaml
   ```
   ```yaml
   network:
     version: 2
     ethernets:
       eth0:
         optional: true
         dhcp4: false
         addresses:
           - 192.168.10.2/24
   ```
   ```bash
   sudo netplan apply
   ```
7. Cek di Mini PC harus benar dan sama dengan awal:
   ```bash
   ip addr   # harus ada 192.168.10.1/24
   sudo ip addr add 192.168.10.1/24 dev eth0   # kalau hilang, set lagi
   ```

Contoh output ethtool:

```
Speed: 1000Mb/s
Duplex: Full
Link detected: yes
```

### Lessons Learned

1. Jika `ssh exedy@exedy.local` gagal, belum tentu SSH yang bermasalah.
2. Selalu cek: `ip addr`, `ip link`, `ethtool`.
3. `Link detected: yes` menandakan layer fisik sudah berfungsi.
4. Untuk robot berbasis ROS 2, lebih aman menggunakan IP statis dibanding mengandalkan `.local`.
5. Pada sistem Brush Kart, komunikasi tether Ethernet lebih cocok menggunakan skema:
   - **Ground Station: 192.168.10.1**
   - **Robot: 192.168.10.2**

Cara SSH ke Raspi:

```bash
ssh exedy@192.168.10.2
```

## Terkait

- [[Project_Overview]]
- [[22_September_2026]]
- [[30_September_2026]]

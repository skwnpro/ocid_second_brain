---
title: Project Overview
date: 2026-09-18
---

# Project Overview

Fokus topik pembuatan robot brush kart dengan spesifikasi sebagai berikut:

## A. Sistem Kontrol

1. Sistem dibagi menjadi 2 yaitu ground station dan robot brush kart (underwater)
2. Ground Station pake mini PC + Ubuntu Desktop 24.04
3. Robot brush kart pake Raspi 4B + Ubuntu Server 24.04
4. Middleware pake ROS 2 Jazzy Jalisco
5. Koneksi pake tether ethernet dari mini PC ke Raspi 4B
6. Kontroller pake Radiomaster Boxer, USB connect ke mini PC atau ground station
7. Penggerak menggunakan motor hidrolik semua
8. Terdapat motor drive, steering, brush, dan thruster
9. Kamera pake kamera Raspi CSI

## B. Sistem Elektrikal

1. Sistem elektrikal utama: AC 220V → PSU 24VDC → DC-DC 5V & 3,3V
2. Power yang ditransfer ke brush kart robot setelah di-stepdown ke 24VDC
3. Kontrol hardware (hidrolik) di brush kart robot menggunakan Raspi, output ke solenoid 24VDC
4. Pada brush kart robot terdapat DC-DC converter ke 5V dan 3,3V untuk power sensor dan Raspi
5. Sensor yang digunakan: sensor tekanan GY-MS5803 dan IMU BN055

## Terkait

**Dokumen teknis:**

- [[ROS2_Installation]]
- [[Operate_ROS2]]
- [[Git_Uses]]
- [[Connection_Ethernet]]
- [[joystick_interface_node]]
- [[Joystick_input_test]]
- [[n_sensor_node]]

**Log harian proyek:**

- [[1_October_2026]]
- [[22_September_2026]]
- [[24_25_September]]
- [[26_September_2026]]
- [[28_September_2026]]
- [[30_September_2026]]
- [[5_October_2026]]

**Review & rencana:**

- [[project_review_201026]]

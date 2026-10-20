---
title: Git Uses
date: 2026-09-24
---

# Git Uses

## Git Push

1. Cek status file:
   ```bash
   cd ~/ros2_ws/src/brushkart_msgs
   git status   # hasilnya muncul mana aja yang dimodif dan dibuat baru
   ```
2. Add perubahan:
   ```bash
   git add .
   ```
3. Commit:
   ```bash
   git commit -m "Add SensorData message"
   ```
4. Push ke GitHub:
   ```bash
   git branch          # lihat dulu di mana (main/master?)
   git push origin main    # kalau main
   git push origin master  # kalau master
   ```
5. Verifikasi:
   ```bash
   git log --oneline -5
   # atau git status lagi
   ```

## Git Pull

1. Cek status file:
   ```bash
   cd ~/ros2_ws/src/brushkart_msgs
   git status   # kalau mau pull harus bersih
   ```
2. Langsung pull:
   ```bash
   git pull
   ```

## Cek Konfigurasi Git

1. ```bash
   git config --global --list   # hasilnya muncul username dan email, kalo belum lanjut step 2
   ```
2. ```bash
   git config --global user.name "skwnpro"
   git config --global user.email "skwn.pro@gmail.com"
   ```
3. ```bash
   git remote -v   # cek apakah masih terkoneksi
   ```
   Contoh hasil:
   ```
   origin  git@github.com:skwnpro/ros2_ur.git (fetch)
   origin  git@github.com:skwnpro/ros2_ur.git (push)
   ```
4. ```bash
   ssh -T git@github.com
   ```
   Hasilnya harus seperti:
   ```
   Hi skwnpro! You've successfully authenticated...
   ```

## Problem: failed to push some refs

Biasanya terjadi karena pull dari GitHub tapi ada commit di GitHub yang tidak ada di Raspi. Solusinya:

```bash
git pull --rebase origin main
git push origin main
```

## Cara Buat Branch di GitHub

1. Cek branch sekarang:
   ```bash
   git branch
   # Biasanya muncul: * main
   ```
2. Buat branch baru:
   ```bash
   git checkout -b feature/camera-improvement
   # atau versi baru git:
   git switch -c feature/camera-improvement
   ```
3. Cek lagi:
   ```bash
   git branch
   # Output:
   # * feature/camera-improvement
   #   main
   ```
4. Push branch pertama kali:
   ```bash
   git push -u origin feature/camera-improvement
   ```

## Kalau Eksperimen Gagal Total

```bash
git switch main                                      # balik ke main
git branch -D feature/camera-improvement             # hapus branch lokal
git push origin --delete feature/camera-improvement  # hapus branch di GitHub
```

main tetap utuh.

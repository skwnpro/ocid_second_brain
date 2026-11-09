---
title: Hermes Agent Install & Setup di InstaBox VPS
---

# Hermes Agent Install & Setup di InstaBox VPS

Panduan untuk menginstal dan menyiapkan Hermes Agent sendiri di VPS InstaBox yang menjalankan Debian 13. Setup awal memakai Hermes Agent stable **v0.23.0**, Nous Portal Free, model gratis `upstage/solar-mini4:free`, dan gateway Telegram. Tidak ada model berbayar, langganan, atau top-up yang dipakai.

> Update 2026-11-09: provider utama pindah ke 9Router + Gemini `gemini-3.6-flash` dan gateway jadi systemd service. Detail lengkap: [[9Router_Hermes_Setup]].

Dokumentasi terkait: [[finance_assistant_dokumentasi]], [[9Router_Hermes_Setup]]

> Versi stable bisa berubah. Perintah installer di bawah mengambil rilis stable terkini; versi yang terpasang saat setup sebelumnya adalah v0.23.0. Jangan menganggap versi terbaru akan tetap sama.

## Ringkasan setup

- **Server:** InstaBox VPS, Debian 13.
- **Instalasi:** installer POSIX resmi Hermes Agent.
- **Provider/model:** 9Router di `http://localhost:20128/v1` dengan `gemini/gemini-3.6-flash` (sejak 2026-11-09). Sebelumnya Nous Portal Free dengan `upstage/solar-mini4:free`.
- **Kunci API dan endpoint khusus yang sudah ada:** dibiarkan seperti semula; jangan ditimpa atau dihapus jika masih dipakai.
- **Telegram:** gateway memakai bot Telegram, akses dibatasi ke akun Telegram pemilik setelah akun itu mengirim `/start` ke bot.
- **Gateway:** system service `hermes-gateway.service` (enabled, auto-start). Sebelumnya dijalankan manual.
- **File dan skill lama:** tidak ada yang dihapus.

## 0. Masuk ke VPS dan cek sistem

Gunakan SSH key yang sudah disiapkan untuk VPS. Jangan menaruh private key, token, atau password di file panduan, perintah yang dibagikan, atau chat.

Contoh format koneksi—ganti nilai placeholder dengan detail VPS milik sendiri:

```bash
ssh -i ~/.ssh/NAMA_PRIVATE_KEY -p PORT USER@HOST
```

Pastikan mesin yang dimasuki benar dan periksa versi OS serta arsitektur:

```bash
cat /etc/debian_version
uname -m
command -v curl
```

Target setup sebelumnya adalah Debian 13. Jika `curl` tidak ditemukan, pasang dengan `sudo apt update && sudo apt install -y curl`.

## 1. Instal Hermes Agent

Jalankan installer POSIX resmi dari repositori Hermes Agent:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Setelah proses selesai, buka sesi SSH baru agar perubahan `PATH` dari installer terbaca. Verifikasi CLI dan versinya:

```bash
hermes --version
```

Pada setup sebelumnya, hasilnya adalah v0.23.0. Jika shell belum menemukan `hermes`, buka ulang sesi SSH lalu coba lagi sebelum mengulang instalasi.

## 2. Setup provider dan model gratis

Jalankan setup awal Hermes:

```bash
hermes setup
```

Ikuti prompt konfigurasi dan pilih **Nous Portal** dengan akses **Free**. Pilih model berikut:

```text
upstage/solar-mini4:free
```

Pastikan pilihan model tetap yang gratis. Tidak perlu membeli paket, mengaktifkan langganan berbayar, atau melakukan top-up untuk konfigurasi ini.

> Update 2026-11-09: ini setup awal. Provider utama sekarang 9Router + Gemini `gemini-3.6-flash` (detail: [[9Router_Hermes_Setup]]); Nous Portal Free jadi pilihan sebelumnya, fallback-nya belum dipasang.

VPS sebelumnya sudah memiliki pengaturan API key dan custom endpoint. Pengaturan itu **tidak diubah**. Jika saat setup terlihat konfigurasi tersebut, jangan hapus atau ganti kecuali memang sengaja ingin mengubah konfigurasi yang ada. Jangan menyalin token atau API key ke catatan ini.

## 3. Siapkan bot Telegram

1. Buat atau pilih bot Telegram sendiri melalui BotFather, lalu simpan token bot dengan aman.
2. Ikuti langkah konfigurasi channel Telegram/gateway yang tersedia pada Hermes Agent v0.23.0 untuk memasukkan token. Hindari mengetik token sebagai bagian dari perintah shell karena perintah dapat tersimpan di riwayat terminal.
3. Buka chat dengan bot dan kirim `/start` dari akun Telegram yang akan diizinkan.
4. Batasi akses bot ke akun Telegram milik sendiri melalui pengaturan allowlist/access control Hermes. Jangan membuka akses ke semua pengguna jika bot hanya untuk pemakaian pribadi.
5. Pertahankan menu slash command bawaan. Jangan menghapus file, skill, atau konfigurasi lama yang tidak terkait.

Token bot bersifat rahasia seperti password. Jangan masukkan token ke dokumen, repositori Git, screenshot, atau chat. Jika token pernah bocor, ganti token melalui BotFather dan perbarui konfigurasi Hermes.

## 4. Jalankan gateway sebagai system service

Sejak 2026-11-09, gateway jalan sebagai system service, bukan proses manual. Daftarkan sekali (setelah konfigurasi Telegram selesai):

```bash
sudo HERMES_HOME=/data/home/.hermes /data/home/.local/bin/hermes gateway install --system
sudo systemctl status hermes-gateway.service --no-pager | head -n 6
```

Harusnya `hermes-gateway.service` jadi `enabled` + `active (running)`. Uji dengan mengirim pesan ke bot dari akun Telegram yang sudah diizinkan. Menutup sesi SSH tidak menghentikan gateway; VPS yang ke-pause lalu bangun lagi juga nyalain gateway sendiri.

Verifikasi gateway pakai model yang benar:

```bash
grep "Model context warmed" ~/.hermes/logs/gateway.log | tail -n 2
```

Harusnya keluar `Model context warmed: gemini/gemini-3.6-flash -> 1048576 tokens`. Perintah `hermes gateway` (foreground) masih bisa dipakai buat debug, tapi bukan cara jalan sehari-hari lagi.

## 5. InstaBox auto-pause dan cron

Ada dua hal berbeda yang perlu diperhatikan:

- **Auto-pause InstaBox** dapat menghentikan atau menjeda VPS sesuai kebijakan paket/panel. Saat VPS tidak berjalan, Hermes dan gateway Telegram di dalamnya juga tidak dapat berjalan. Periksa pengaturan serta status VPS di panel InstaBox; jangan menganggap proses Hermes dapat mencegah VPS di-pause.
- **Cron di dalam VPS** berjalan di mesin virtual itu sendiri dan hanya dapat menjalankan pekerjaan ketika VPS aktif. Cron tidak membuat mesin yang sedang berhenti menjadi aktif.
- **Cron di luar VPS** adalah scheduler pada mesin/layanan lain. Ia dapat meminta VPS menjalankan pekerjaan hanya jika scheduler itu sendiri tetap hidup dan punya cara untuk mengakses VPS. Cron eksternal tidak sama dengan cron di dalam Debian.

Gateway sekarang system service, jadi dia hidup sendiri tiap VPS bangun (resume) atau reboot. Yang tetap nggak bisa: cron di dalam VPS tidak bisa membangunkan VPS yang sedang pause — banguninnya dari luar (`instabox resume finance-bot` atau tombol Resume di dashboard instabox). Catatan: tiap gateway cold boot, update Telegram yang ngantri selama dia offline di-drop — chat yang dikirim pas gateway mati harus dikirim ulang.

## Troubleshooting singkat

- **`hermes: command not found`:** buka sesi SSH baru supaya `PATH` yang ditambahkan installer dimuat. Pastikan instalasi selesai tanpa error.
- **Bot Telegram tidak merespons:** pastikan VPS aktif, `sudo systemctl status hermes-gateway.service` nunjukin `active (running)`, token bot benar, akun sudah mengirim `/start`, dan akun tersebut diizinkan oleh access control. Kalau VPS sempat ke-pause, bangunin dulu (`instabox resume finance-bot`), terus kirim ulang pesannya — pesan yang dikirim pas gateway mati itu ke-drop.
- **Gateway berhenti setelah sesi SSH ditutup:** nggak seharusnya terjadi lagi sejak gateway jadi system service. Cek `sudo systemctl status hermes-gateway.service` dan `journalctl -u hermes-gateway -n 50` buat lihat kenapa dia mati.
- **Gateway berhenti ketika VPS di-pause/reboot:** aktifkan kembali VPS (`instabox resume finance-bot`); service `hermes-gateway` nyala sendiri pas mesin bangun. Kalau nggak nyala, cek `sudo systemctl status hermes-gateway.service`.
- **Model/provider tidak sesuai:** provider utama sekarang 9Router + `gemini/gemini-3.6-flash` (cek `grep -n "^  default:\|^  provider:\|^  base_url:" ~/.hermes/config.yaml`). Nous Portal Free + `upstage/solar-mini4:free` itu settingan lama sebelum 2026-11-09. Jangan menghapus API key atau endpoint khusus yang sudah ada untuk memperbaikinya; detail pindahnya ada di [[9Router_Hermes_Setup]].

## Checklist akhir

- [ ] SSH masuk ke VPS Debian 13 yang benar.
- [ ] `hermes --version` menampilkan versi yang terpasang.
- [ ] Provider utama adalah 9Router (`http://localhost:20128/v1`), sebelumnya Nous Portal Free.
- [ ] Model aktif adalah `gemini/gemini-3.6-flash`.
- [ ] Pengaturan Nous lama tersimpan di `config.yaml.bak-before-9router`, tidak dihapus.
- [ ] Gateway Telegram memakai token yang disimpan aman.
- [ ] Akun Telegram pemilik sudah mengirim `/start` dan akses dibatasi melalui allowlist.
- [ ] `hermes-gateway.service` `active (running)` dan bot merespons.
- [ ] Dipahami bahwa gateway sekarang system service (auto-start pas VPS bangun), dan VPS dapat terdampak auto-pause.
- [ ] Tidak ada file atau skill lama yang dihapus.

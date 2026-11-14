# Penyelamatan Data G480

Ringkasan 14 November 2026 — malam ini pekerjaan G480 beneran jalan, bukan lagi tanya-jawab konsep: SSH dibenerin, 9Router terpasang, WiFi akhirnya jalan setelah ganti driver, dan Hermes mulai di-install (side chat "Choose OS for 4GB laptop server", 22:29–23:53 WIB).

## SSH: user balik arah, Bre yang remote

- **Pukul 22:29 user membatalkan rencana "pandu saja".** Alasannya praktis: ribet ketik perintah panjang di laptop G480. Jadinya Bre yang mengerjakan lewat SSH.
- **Password `skwn` dikirim user sekali di chat** untuk login pertama. Dipakai sekali untuk masuk, lalu kunci SSH Bre langsung dipasang di G480 — setelah itu login cukup pakai kunci, tanpa password lagi. Passwordnya tidak disimpan di mana-mana.
- **Yang sempat nyangkut:** SSH Bre sempat nyari kunci di `/root/.ssh`, padahal kuncinya di `/home/hatch/.ssh`. Sudah dibetulkan di config (`IdentityFile`), dan login kunci saja sudah terverifikasi masuk sebagai `skwn` di `g480-server`.
- **Sudo di G480 masih butuh password.** Bre sempat menawarkan agar `skwn` bisa sudo tanpa password supaya instalasi berikutnya tidak mengganggu user lagi — belum dijawab user.

## 9Router terpasang di G480

- **9Router v0.5.119 jalan sebagai systemd service** (`9router.service`), status `active (running)` dan auto-start saat boot.
- Dashboard dicek dari dalam G480: `http://127.0.0.1:20128/dashboard` mengembalikan 200. Akses tetap localhost saja, sesuai rencana Tailscale + SSH tunnel.
- **Yang masih belum:** provider 9Router belum disambungkan ke Gemini, dan password dashboard masih default. Kalau dipakai kunci Gemini yang sama seperti bot finance, kuota 20 req/hari itu kepakai bareng.

## WiFi: ternyata drivernya yang rusak, bukan hotspot

- **Gejala awal:** G480 nyambung ke hotspot Redmi 3S (status `COMPLETED`, password benar), tapi tidak pernah dapat IP — DHCP Discover dikirim berkali-kali, nol balasan. USB tethering jalan normal, jadi bukan salah G480 secara umum. WiFi ini sempat jalan sekali pukul 21:24, dapat IP `172.26.197.151`, lalu tidak pernah lagi.
- **Tes demi tes menyingkirkan penyebab lain:** restart hotspot sempat membuat DHCP menawarkan IP sekali (`172.26.197.151`), tapi konfirmasi (DHCPACK) tidak pernah datang. IP statis di subnet yang benar (`172.26.197.160`) tetap 100% loss ke gateway — jadi bukan cuma DHCP yang rusak, data sama sekali tidak lewat. Power save WiFi sudah dicek, sudah off.
- **Tes penentu:** hotspot kedua, Xiaomi 14T Pro, hasilnya persis sama — nyambung, DHCP nol balasan. Di log kernel (`dmesg`) muncul warning berulang `txop exceeded` dari driver `brcmsmac` (chip BCM4313) saat mau kirim data. Kesimpulan: driver bawaan `brcmsmac` bermasalah di chip ini — nyambung bisa, kirim data tidak bisa.
- **Perbaikan (disetujui user pukul 23:04, "Gas, install driver wl + reboot sekarang"):** paket `broadcom-sta-dkms 6.30.223.271` ter-install, driver lama (`brcmsmac`, `bcma`, dkk.) di-blacklist, G480 di-reboot. Tethering dibiarkan nyolok selama reboot sebagai jalur masuk cadangan.
- **Hasil setelah reboot:** driver `wl` yang dipakai, dan nama interface berubah dari `wlp2s0b1` jadi `wlp2s0` — config netplan ikut di-update, kalau tidak WiFi tidak terkonfigurasi. Tes lewat WiFi Xiaomi 14T Pro: IP `10.40.62.151/24`, gateway `10.40.62.90`, ping gateway 3/3 ~2,7 ms, ping 8.8.8.8 3/3 ~52 ms, buka example.com 200. WiFi sekarang jalur utama (metric 600), tethering jadi cadangan (metric 1024).

## Hermes: install dipandu, lalu user install sendiri

- **Awalnya user minta dipandu saja, jangan di-remote** ("pandu gw install hermes bre, jangan ambil alih remote. Pandu aja biar gw paham"). Bre memandu instalasi lewat installer resmi Hermes (`curl …/NousResearch/hermes-agent/main/scripts/install.sh | bash`), lalu minta cek `hermes --version` di terminal baru.
- **Pukul 23:53 user melaporkan sudah install Hermes sendiri**, dan bertanya: sudah benar atau belum, apa saja yang belum di-setup, dan apakah akunnya sama seperti di VPS.
- **Jawaban Bre: belum bisa dinilai tanpa melihat hasilnya**, karena install Hermes ada dua lapis — CLI-nya terpasang, dan provider/modelnya ter-set. Soal akun: **tidak otomatis sama seperti VPS.** Hermes di G480 adalah instalasi baru dengan config lokal sendiri. Yang menentukan sama/tidaknya adalah provider dan kunci yang dimasukkan saat `hermes setup` — kalau kuncinya sama seperti di VPS (Nous Portal / Gemini), akun dan kuotanya kepakai bareng; kalau provider/kunci lain, ya beda.
- **Yang diminta dari user untuk verifikasi:** output `hermes --version`, `ls ~/.hermes/`, dan `grep` provider/base_url di `~/.hermes/config.yaml`, plus jawaban dua hal: provider apa yang dipilih saat `hermes setup`, dan apakah token bot Telegram sudah dimasukkan.
- **Yang sudah diketahui belum di-setup di G480:** 9Router belum disambungkan ke Gemini, password dashboard masih default, Hermes belum diarahkan ke 9Router (`http://localhost:20128/v1`), dan gateway Telegram belum ada service-nya.

## Status — belum terverifikasi

- **Tes terakhir WiFi belum terkonfirmasi:** user diminta mencabut kabel USB tethering, lalu Bre mengecek apakah internet + Tailscale + SSH tetap jalan hanya lewat WiFi. Sampai ringkasan ini ditulis, belum ada konfirmasi dari user.
- **Setup Hermes belum terverifikasi** — menunggu user mengirim output tiga perintah di atas dan jawaban soal provider + token Telegram.
- **9Router belum disambung ke Gemini dan password dashboard belum diganti** dari default.
- **Passwordless sudo** masih menunggu keputusan user.

## Terkait

- [[2026-11-13 - Penyelamatan Data G480]] — pertanyaan keamanan port 22 dan asal port 20128/9119 yang jadi dasar rencana remote ini.
- [[2026-11-10 - Hermes Balik ke Gemini]] — setup 9Router/Hermes di VPS yang sedang direplikasi ke G480.

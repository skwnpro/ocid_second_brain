# Penyelamatan Data G480

Ringkasan 13 November 2026 — user mengajukan dua pertanyaan lanjutan soal rencana remote G480 via Tailscale yang diputuskan kemarin: keamanan port 22, dan asal-usul nomor port 20128 dan 9119 (side chat "Choose OS for 4GB laptop server", 13:07–13:43 WIB). Belum ada eksekusi — ini masih tanya-jawab konsep sebelum langkah pertama dijalankan.

## Port 22 kebuka itu bahaya atau tidak?

- **Yang bahaya: port 22 yang di-port-forward ke internet.** Bot di internet nyari port 22 yang kebuka 24 jam dan nyoba login pakai username umum (`root`, `admin`, `ubuntu`) dan password umum, ribuan kali sehari (brute force). Kalau password lemah atau SSH versi lama bercelah, bisa jebol.
- **Rencana user tidak seperti itu.** Di rencana Tailscale, port 22 G480 cuma kebuka di dua jaringan privat: LAN rumah dan Tailscale (IP `100.x.x.x`, bukan IP publik). Router tidak meneruskan apa-apa ke G480, jadi bot internet bahkan tidak tahu G480 ada.
- **Kalau mau lebih aman lagi, urut dari yang paling ngaruh:**
  1. Jangan port forwarding 22 — sudah dipenuhi dengan memilih Tailscale.
  2. Password kuat dan unik, jangan sama seperti akun lain.
  3. Pindah ke SSH key nanti kalau SSH sudah jalan (kunci publik di server, matikan login pakai password).
  4. Firewall (`ufw`) yang cuma mengizinkan SSH dari Tailscale.
  5. Fail2ban — satpam otomatis yang memblokir IP gagal login berkali-kali; lebih berguna kalau port kebuka ke internet, kurang perlu kalau cuma Tailscale.
- **Yang bukan keamanan beneran:** ganti port 22 ke port lain (misal 2222) cuma ngumpetin pintu — bot tetap bisa scan semua port dan menemukannya.

## Dari mana angka 20128 dan 9119, dan port itu kerjanya gimana?

- **Analogi port:** IP itu alamat rumah, port itu nomor kamar di dalam rumah itu. Satu komputer punya satu IP, tapi banyak layanan, masing-masing punya kamar sendiri. Alamat lengkap selalu `IP:port`, misal `100.x.x.x:20128`. Nomor port 0–65535; yang sudah jadi standar: 22 = SSH, 80 = HTTP, 443 = HTTPS.
- **Cara kerja:** sebuah program "duduk nunggu" (listening) di satu port; kalau ada yang ketok ke port itu, program itu yang jawab. Kalau tidak ada program yang nunggu, jawabannya connection refused. Dua cara program nunggu: `127.0.0.1` (localhost) = cuma dari mesin itu sendiri yang bisa ketok; `0.0.0.0` = dari mana saja bisa ketok.
- **20128 itu port default 9Router.** Yang bikin 9Router (decolua) yang menentukan angka itu di kodenya; README resminya menyebut dashboard di `http://localhost:20128` dan API di `http://localhost:20128/v1`. Angka yang sama sudah dipakai dan terbukti jalan di VPS `finance-bot`.
- **9119 itu port default dashboard Hermes.** Yang bikin Hermes yang menentukan; `hermes dashboard` defaultnya nunggu di `127.0.0.1:9119`, sudah dicek langsung kebuka halaman "Hermes Agent - Dashboard" di VPS.
- **Kenapa perlu SSH tunnel:** keduanya defaultnya nunggu di localhost saja, jadi dari PC tidak bisa langsung ketok `100.x.x.x:20128`. Tunnel-nya: `ssh -L 20128:localhost:20128 -L 9119:localhost:9119 skwn@g480-server` — kamar 20128 di PC disambungkan ke kamar 20128 di server lewat SSH. Setelah itu buka `http://localhost:20128` (9Router) dan `http://localhost:9119` (Hermes) di browser PC.

## Status — belum terverifikasi

- **User belum mengonfirmasi sudah login ke G480** (pakai keyboard USB) dan menjalankan langkah Tailscale — IP Tailscale G480 masih ditunggu.
- **BACKUP64 belum dicek `fsck`** — jangan dipercaya di Windows sebelum dicek.
- Dua pertanyaan hari ini tidak mengubah rencana kemarin; user masih di tahap memahami sebelum eksekusi.

## Terkait

- [[2026-11-12 - Penyelamatan Data G480]] — clean install selesai, keputusan remote via Tailscale, dan rencana lengkap 4 langkahnya.
- [[2026-11-10 - Hermes Balik ke Gemini]] — setup 9Router/Hermes di VPS yang akan direplikasi ke G480.

# Dashboard VPS dan G480

Ringkasan 15 November 2026 — user bertanya apa itu Space, lalu memesan dashboard buat memantau VPS dan server G480, dan dashboard itu langsung dibangun dan jalan malam itu juga (main chat, 00:39–23:58 WIB).

## Space itu apa

- Pukul 00:39 user bertanya "space itu fitur apa sih bre di muse". Jawaban asisten: Space itu halaman web atau aplikasi kecil yang dibuatkan buat user — bukan sekadar jawaban chat, tapi sesuatu yang interaktif dan bisa dibuka kapan saja dari sidebar atau Library. Contohnya tracker/planner, dashboard, atau game/proyek kreatif.

## Pesanan dashboard

- Pukul 01:05 user memesan: dashboard buat memantau VPS dan server G480, yang mau dilihat: (1) suhu CPU, (2) RAM usage, (3) storage usage, (4) aplikasi yang sedang berjalan — plus tambahan dari asisten kalau ada yang penting.
- **Dua hal jujur sebelum dibangun:** suhu CPU di VPS kemungkinan tidak kebaca (VM tidak punya sensor suhu fisik; di G480 bisa asal `lm-sensors` terpasang), dan supaya datanya live perlu script kecil di tiap mesin yang mengirim metrik ke dashboard tiap menit.
- Tambahan dari asisten: beban CPU (%), uptime, swap, 5 proses teratas, status service, jaringan, baterai, dan timestamp update terakhir — plus badge online/stale/offline dari umur snapshot dan auto-refresh.
- Pukul 01:07 user memerintahkan **"Pasangin langsung aja"** — perintah takeover langsung untuk tugas ini (mode guide-only yang biasa berlaku untuk G480 tidak dipakai di sini, atas perintah user).

## Dibangun dan live malam itu juga

- **Space "Dashboard VPS dan G480" jadi pukul 01:12**, datanya live pukul 01:13. Update otomatis tiap 5 menit, diam-diam tanpa spam chat; baru memberi kabar kalau ada yang gagal.
- **Script pengumpul** (Python stdlib-only) dipasang di kedua mesin dan dites sebelum dashboard jadi:
  - VPS: RAM 16%, disk 29%, 4 service jalan semua (9Router, Hermes gateway, bot finance recorder + advisor). Suhu CPU memang tidak ada sensornya, jadi di dashboard tampil "tidak tersedia" — bukan error.
  - G480: suhu 58°C, RAM 69%, disk 9%, 9Router jalan. Hermes gateway masih belum aktif saat itu.
- **Kunci instabox habis besok, 16 November.** Setelah itu data VPS berhenti update (tampil stale/offline) sampai user memberi token baru — sudah disampaikan ke user malam itu.

## RAM dan storage G480

- Pukul 01:22 user bertanya: RAM yang kepake di G480 cuma 2 GB ya, dan storage cuma sekitar 98 GB?
- **Jawaban asisten: iya, dua-duanya benar — dengan nuansa:**
  - **RAM: yang kebaca sistem cuma ~1,7 GB total, bukan 4 GB.** Kemungkinan yang terpasang memang cuma 2 GB, atau satu keping 2 GB lagi tidak kebaca. Yang terpakai ~1,3 GB, sisa available ~0,5 GB — lumayan mepet, Hermes makan banyak. Untuk memastikan keping fisiknya, user diminta menjalankan `sudo dmidecode -t memory | grep -E 'Size|Locator'` di G480 — belum terverifikasi.
  - **Storage: 98 GB itu partisi root-nya saja.** HDD-nya sendiri 320 GB; waktu clean install, root dibuat ~100 GB (LVM).

## G480 gelap sejak 01:33 — belum terverifikasi

- **Pukul 01:33 WIB metrik G480 berhenti masuk** — jalur SSH ke G480 putus, percobaan ulang langsung gagal. VPS aman, datanya tetap update.
- Pukul 09:14 VPS sempat cooldown (free plan, bangun lagi ~09:18), G480 masih gelap. Pukul 09:52 follow-up: G480 masih offline sejak ~01:30, semua polling gagal terus. Jalur SSH dari sisi asisten yang putus (proxy menutup koneksi), jadi kondisi G480-nya sendiri tidak bisa dipastikan — **kemungkinan besar G480-nya aman, cuma tunnel-nya yang mati (belum terverifikasi)**.
- Pukul 17:25 VPS cooldown lagi (bangun ~18:18), G480 masih gelap sejak 01:33.
- **Status terakhir malam ini (polling 23:58 WIB):** VPS OK lagi — metrik ter-ingest (CPU 0,7%, RAM 9,3%, disk 29,4%; 9Router, advisor-bot, cashflow-bot aktif, hermes-gateway tidak aktif). **G480 masih tidak terjangkau via SSH tunnel proxy** ("proxy closed without response"), gelap sejak 01:33 WIB — sudah lebih dari 22 jam.

## Terkait

- [[2026-11-14 - G480 Server]] — G480 jadi server sehari sebelumnya: WiFi, Tailscale, 9Router, dan install Hermes yang dashboard ini pantau.

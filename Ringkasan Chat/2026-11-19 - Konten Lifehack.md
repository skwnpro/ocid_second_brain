# Konten Lifehack

Ringkasan 19 November 2026 — di WhatsApp (16:28–16:43 WIB): Sprint 0 jalan lagi setelah user sempat sakit, dan dua hal yang tadinya menggantung akhirnya kekunci — alat voiceover-nya dan tempat render-nya.

## User balik setelah sakit, Sprint 0 lanjut (16:28–16:35)

- Pukul 16:28 user: "Sorry bre gw abis sakit, yok kita lanjut" — dia sempat sakit, sekarang balik dan mau lanjut Sprint 0.
- Pukul 16:30 user tanya voiceover-nya pakai apa. Pukul 16:35 dia memastikan: "berarti sprint0 ini jalan di local gw ya? Bukan di vps?" — iya, **Sprint 0 jalan lokal di PC desktop-nya, bukan di VPS** (VPS gratis gampang pause, G480 terlalu lemah buat render Remotion). Ini mengunci pembagian kerja yang sudah disepakati 17 Nov: Bre bikin kode/template, user render di PC-nya sendiri.

## Voiceover: ElevenLabs $1, tarif $0,08/1k karakter (16:41–16:43)

- Pukul 16:41 user: sudah langganan **ElevenLabs $1** (promo bulan pertama) — Bre memastikan bisa kepake dan kualitas suaranya lebih bagus. Ini menggantikan edge-tts yang tadinya pilihan pertama tapi WebSocket-nya gagal di sandbox VM Muse.
- Pukul 16:42 user melaporkan tarifnya: **$0,08 per 1.000 karakter**. Naskah 24 jam itu 969 karakter / 157 kata, jadi satu kali generate voiceover cuma **~$0,08** — generate ulang 10 kali pun baru $0,80.
- Kesimpulan Bre: buat Sprint 0, **biaya TTS praktis bukan masalah**. Yang beneran perlu diukur nanti adalah menit render + waktu user sendiri dari naskah sampai MP4 jadi.

## List kerja malam ini (16:43)

- User minta list apa saja yang perlu diberesin malam ini. Bre memberi list Sprint 0:
  - **Siapin dulu:** PC desktop nyala, cek Node.js + npm (`node -v`) dan FFmpeg (`ffmpeg -version`) — kalau belum ada, install dulu; file sudah di tangan (zip template Remotion + naskah 24 jam); satu track musik royalty-free buat backsound.
  - **Kerjain:** ElevenLabs — coba satu suara cowo + satu cewe, dengarkan, pilih yang paling enak; paste narasi lengkap, generate, download mp3-nya. Kirim mp3 ke Bre — Bre membalas file timing JSON (timestamp tiap kata buat caption). Copy folder template ke PC, `npm install`, masukkan timing JSON, render (`npm run render`), catat berapa menit render-nya. FFmpeg: gabung video bisu + voiceover + musik jadi MP4 final. Tonton hasilnya, nilai sendiri layak tayang atau belum. Catat tiga angka: menit render, biaya TTS (~$0,08), total waktu dari awal sampai MP4 jadi — bahan mutusin lanjut bangun factory atau nggak.
  - **Yang paling penting malam ini cuma voiceover dulu** — sisanya ngikutin.

## Langkah berikut

- Voiceover + timing JSON adalah satu-satunya bagian yang belum ada dari Sprint 0; template Remotion sudah jadi sejak 17 Nov. Kalau voiceover malam ini kelar, render dan ukur tiga angka di atas, lalu putusan go/no-go factory.

## Terkait

- [[2026-11-17 - Konten Lifehack]] — naskah episode 1 (aturan 24 jam) dan kesepakatan Sprint 0 yang dipakai malam ini.
- [[2026-11-17 - AI Content Factory]] — sistem factory dan Sprint 0 sebagai spike-nya.

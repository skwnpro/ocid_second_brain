# Video Motion What If

Ringkasan 12 November 2026 — user menghidupkan lagi rencana video motion graphic "what if" pakai Remotion (WhatsApp, 13:40 WIB).

## Yang terjadi

User menulis singkat: "Bikin video motion". Itu sinyal mulai — tapi tiga hal yang dari 6 November belum ditentukan masih tetap menggantung, dan asisten menanyakannya lagi sebelum mulai menulis kode:

1. **Topik "what if"-nya apa?** Misal "what if manusia bisa terbang", atau ada ide lain?
2. **Shorts atau video panjang?**
3. **Bahasa Indonesia atau Inggris?**

Jawaban atas ketiganya belum ada — user belum membalas pertanyaan itu.

## Pembagian kerja yang sudah disepakati (6 November, masih berlaku)

- **Asisten hanya menulis kode** (project Remotion lengkap, naskah, animasi, aset) — serapi mungkin biar enak dibaca dan diedit sendiri.
- **User yang render manual di PC-nya** — biar hemat token. Langkahnya: install Node LTS + ffmpeg, `npm install`, `npx remotion browser install chrome`, preview di Remotion Studio, lalu `npx remotion render`.
- Bedanya Chromium vs Chrome sudah dijelaskan 6 November: Chromium itu bahan mentah open-source (tanpa codec H.264/AAC, tanpa auto-update), Chrome itu produk jadi; buat render headless Remotion bedanya hampir tidak berpengaruh, pilih Chrome saja.
- Cek kesiapan 6 November: ffmpeg 8.1.2 dan Node 24 sudah ada di VM cloud; yang belum ada: project Remotion-nya dan Chrome headless. Render tetap di PC user, bukan di VM.

## Status — menunggu tiga jawaban

Belum ada kode yang ditulis malam ini. Begitu user menjawab topik, format, dan bahasa, asisten langsung menggarap template code-nya.

## Terkait

- [[2026-11-10 - Animasi Cacing]] — proyek video/konten lain user yang juga masih jalan.

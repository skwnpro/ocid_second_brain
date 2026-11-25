# Hidroponik

Ringkasan 24 November 2026 — di side chat "Design ESP32 pipe hydroponics" (16:22–16:23 WIB): lanjutan dari ringkasan 23 November. Pertanyaan kritis yang tertinggal kemarin terjawab, dan Bre mengirim gambar v2 + partlist final.

## Jawaban user: pipa 3/4" buat rangka DAN jalur air (16:22 WIB)

- User menjawab pertanyaan yang tertinggal di ringkasan kemarin: pipa 3/4" dipakai untuk rangka **dan** jalur air/manifold — bukan salah satu saja.
- Dengan jawaban ini, pertanyaan kritis dari 23 November **tertutup**.

## Ilustrasi v2 + partlist final

- Bre mengirim ilustrasi v2 sesuai jawaban itu: rak 160 × 60 × 120 cm, manifold 3/4" di atas, 4 pipa tanam 4" paralel, tandon 60 L di bawah, timer, genangan 15–20 mm.
- **Koreksi dari Bre atas gambarnya sendiri:** jalur balik di gambar digambar turun ke lantai — aslinya harus balik ke tandon, jangan ke lantai. Sisanya sudah benar.
- Partlist final v2:
  - **Pipa tanam:** PVC 4" × 4 m, 2 batang, potong jadi 4 × 1,5 m; end cap 4" × 8; netpot 7 cm × 32; rockwool 1 slab.
  - **Rangka + jalur air, semua 3/4" AW:** pipa 3/4" AW × 4 m sekitar 5 batang; elbow 3/4" sekitar 16; T 3/4" sekitar 20; end cap 3/4" buat ujung manifold yang dibuntu; kran 3/4" × 4, satu per pipa tanam buat menyamakan aliran.
  - **Tandon + pompa + timer:** box hitam 60 L × 1, isi 40–50 L; pompa celup 15–25 W, head 2 m; timer listrik analog/digital × 1; selang/pipa 3/4" buat naik dari pompa ke manifold.
  - **Nutrisi:** AB Mix sayur daun 1 kg, EC/TDS meter, pH meter, pH up/down.

## Syarat rangka 3/4" kuat

- 3/4" bisa dipakai buat rangka, tapi tidak bisa asal bikin seperti rangka besi. Lima syarat dari Bre:
  1. **Pakai PVC AW, bukan D.** AW dindingnya tebal, buat tekanan; yang D tipis, gampang melengkung.
  2. **Kaki tiap 50 cm, jangan cuma 4 kaki di pojok.** Rak 160 cm butuh 6–8 kaki total, jadi tiap pipa tanam 4" ditopang di 3 titik: ujung kiri, tengah, ujung kanan.
  3. **Kasih penguat silang X** di samping kiri-kanan sama belakang — ini yang bikin rangka tidak goyang.
  4. **Lem kering total 24 jam** sebelum dikasih beban air + tanaman. Kalau baru 2 jam sudah diisi, sambungan bisa geser.
  5. **Tandon jangan ditaruh di atas rangka.** Tandon tetap di lantai; rangka cuma menahan pipa tanam, sekitar 40–50 kg.
- Kalau lima syarat ini dilanggar, tengahnya pasti melengkung, air jadi mengumpul di tengah dan tidak mengalir.

## Timer + genangan (tetap seperti v2 kemarin)

- Genangan di dasar pipa **15–20 mm**, pakai standpipe di ujung buangan — itu yang bikin pompa tidak perlu nyala terus.
- Siang 07:00–17:00: ON 15 menit, OFF 30 menit. Malam: ON 15 menit, OFF 60 menit.
- Matahari 4–5 jam cukup buat selada, panen sekitar 35–40 hari. Taruh di spot matahari pagi; tandon tutup rapat biar air hujan tidak masuk dan mengacaukan EC.

## Belum terverifikasi / masih terbuka

- **Belum diputuskan user:** lanjut ke monitoring ESP32 sekarang, atau beresin rangka dulu — Bre sudah menanyakan, belum dijawab sampai malam ini.
- Ilustrasi v2 dan partlist di atas adalah rancangan Bre berdasarkan jawaban user; belum ada yang dibeli atau dirakit — belum terverifikasi di lapangan.

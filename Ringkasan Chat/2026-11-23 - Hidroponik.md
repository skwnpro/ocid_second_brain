# Hidroponik

Ringkasan 23 November 2026 — di side chat "Design ESP32 pipe hydroponics" (15:08–20:16 WIB): user mau bikin sistem hidroponik yang bisa dimonitoring lewat HP pakai ESP32. Fokus dulu ke hidroponiknya (sistem, partlist, teknik), monitoring belakangan. Mau yang pakai pipa biar gampang dimonitor.

## Desain awal: NFT pipa, 32 lubang

- Teknik yang dipilih: NFT (Nutrient Film Technique) pakai pipa PVC — air nutrisi ngalir tipis di dasar pipa, akar kecipratan air tapi tetap kena udara. Jalur air cuma satu (tandon → pompa → pipa → balik ke tandon), jadi sensor nanti cukup taruh di tandon.
- Skala starter: 4 pipa PVC 4 inch × 1,5 m, masing-masing 8 lubang, total 32 lubang, buat selada / pakcoy, panen 30–35 hari. Rak 2 tingkat (2 pipa atas, 2 bawah), footprint sekitar 160 × 60 cm.
- Pipa 4 inch dipilih bukan 3 inch karena akar selada dewasa lebat dan gampang mampet di pipa kecil.
- Alur: tandon 60 L warna gelap di bawah → pompa celup 15–25 W, head 1,5–2 m, flow 1000–1200 L/h → manifold → dibagi ke 4 pipa pakai kran kecil → air ngalir miring 2–3 cm per 1,5 m → balik ke tandon. Aliran per pipa 1–2 liter/menit, lapisan air di dasar pipa cuma 1–2 mm.
- Partlist: pipa PVC 4" (2 batang 4 m dipotong jadi 4 × 1,5 m), end cap 4" × 8, hole saw 65 mm, jarak lubang 20 cm, pipa 1/2" + selang PE 16 mm buat inlet, kran 1/2" × 4, pipa 3/4" buat jalur balik, netpot 7 cm × 32, rockwool 1 slab, AB Mix sayur daun 1 kg, EC/TDS meter + pH meter pen, pH up/down. Estimasi di luar rak sekitar Rp700 ribu–1 juta; rak baja ringan nambah Rp200–300 ribu.
- Teknik tanam: semai di rockwool 1–2 hari gelap sampai kecambah, langsung kena cahaya biar tidak kutilang; pindah tanam umur 10–14 hari (3–4 daun). Nutrisi: 0–7 hari air biasa, 7–14 hari EC 0,8–1,0, 14–28 hari EC 1,2–1,6, dewasa sampai panen EC 1,4–1,8; pH 5,5–6,5. Ganti total air nutrisi tiap 2 minggu.
- Trik anti-mati-listrik: lubang buangan ujung pipa dinaikin 5 mm, jadi selalu ada genangan tipis cadangan — akar tahan 1–2 jam kalau pompa mati.

## Horizontal vs vertikal: horizontal menang

- User sempat tanya vertikal vs horizontal. Jawaban Bre: horizontal tetap mending buat kasus ini — cahaya merata, nutrisi merata, monitoring ESP32 simpel (1 set sensor di tandon), pompa kecil cukup, bersihin pipa gampang.
- Vertikal cuma menang hemat lahan (60–80 lubang di 1 m² yang sama), tapi tanaman bawah keteduhan dan panen telat 3–5 hari, EC bisa beda per level, pompa harus lebih kuat (head 2,5–3 m), dan kalau satu jalur mampet satu tower mati semua. Jalan tengah: A-frame.

## Sketsa user: rangka pipa + kolam ikan + tangki atas

- User mengirim 3 foto sketsa (19:34 WIB): rangka full pipa, kolam ikan di bawah, pompa dorong air ke tangki atas, lalu air ngalir gravitasi seri dari pipa atas ke pipa bawah dan balik ke kolam ikan.
- Plus yang diakui Bre: rangka pipa murah, tidak karatan, gampang bongkar pasang; kolam ikan = aquaponik, sekali jalan dua panen; tangki atas + gravitasi = aliran stabil dan pompa lebih awet.
- Minus yang kritis:
  - "Pompa nyala sekali doang" hampir tidak mungkin kalau tangki kecil — 4 pipa butuh 4–8 liter/menit, tangki 20 liter habis dalam 3–5 menit. Pompa tetap harus nyala berkali-kali, kecuali tangki atas 200 liter (terlalu berat buat rangka pipa).
  - Rangka pipa belum tentu kuat: 4 pipa isi air + tanaman sekitar 40–50 kg, ditambah tangki atas isi air 20–30 kg, total 70–100 kg. PVC gampang melengkung kalau dibebani lama, apalagi kena panas matahari; sambungan lem rawan lepas kalau goyang. Perlu penguat silang.
  - Ini sudah aquaponik, bukan hidroponik biasa — jauh lebih ribet: amonia ikan racun dan belum bisa langsung dimakan tanaman, butuh bakteri pengubah amonia jadi nitrat + filter/media bakteri; nutrisi ikan lebih encer dari AB Mix, sayur tumbuh lebih pelan.
  - pH ikan vs tanaman tabrakan: ikan mau 6,8–7,5, tanaman mau 5,5–6,5; kompromi 6,8–7,0 bikin tanaman kurang maksimal nyerap nutrisi.
  - Aliran seri bikin tanaman bawah dapat sisa nutrisi + oksigen lebih sedikit; kalau satu pipa mampet, semua kena.
- Ikan yang disarankan kalau tetap aquaponik: nila (paling gampang buat pemula, tahan banting, cepat besar), lele (paling bandel soal oksigen rendah, tapi kotoran dan amonianya tinggi), koi/komet cuma hias dan paling sensitif — tidak cocok buat percobaan pertama.
- Bre minta jawaban 8 hal sebelum revisi desain (diameter/tebal pipa rangka, ukuran kolam, volume + tinggi tangki atas, detail ujung pipa/stopper di gambar, arah aliran, tanaman, filter/aerator, lokasi).

## Keputusan user: v2 — hidroponik full, paralel, tandon bawah (20:16 WIB)

- User setuju: hidroponik saja full (bukan aquaponik), paralel (bukan seri). Ikan dan tangki atas gugur.
- Pipa 3/4" dilem sampai kering total biar tidak gerak; tandon di bawah saja, dibikin agak tergenang biar tidak harus selalu mengalirkan air, pakai timer; sambungan T ada bagian yang ditutup/dibuntu buat mengarahkan aliran; tanaman tetap selada/pakcoy 32 lubang; lokasi outdoor tapi matahari cuma 4–5 jam.
- Rangkuman Bre atas desain v2 (NFT-DFT hybrid): tandon bawah → pompa → manifold → dibagi ke 4 pipa masing-masing lewat T → tiap pipa balik sendiri-sendiri ke tandon. Kalau 1 pipa mampet, 3 lainnya tetap jalan; tanaman atas-bawah dapat nutrisi yang sama.
- Ujung buangan tiap pipa dinaikin pakai standpipe sekitar 15–20 mm, jadi dasar pipa selalu ada genangan segitu; akar tetap kecelup walau pompa mati.
- Timer awal: siang 07:00–17:00 ON 15 menit / OFF 30 menit; malam 17:00–07:00 ON 15 menit / OFF 60 menit. Kalau tanaman layu, pendekin OFF; kalau akar busuk / air keruh, panjangin OFF.
- Sambungan T yang dibuntu itu buat manifold — ujung manifold ditutup end cap, cabang T yang tidak kepakai juga ditutup. Tiap cabang ke pipa dikasih kran kecil biar aliran 4 pipa sama rata; jangan bikin cabang buntu yang panjang (air mati, lumutan); kasih 1 kran drain di titik paling rendah buat nguras.
- Matahari 4–5 jam masih cukup buat selada, cuma panen lebih lambat (35–40 hari, bukan 30 hari); pakcoy masih oke; kangkung jangan dulu (butuh matahari lebih banyak). Taruh di spot kena matahari pagi 07:00–11:00; tandon harus ketutup rapat, kalau tidak air hujan masuk dan EC langsung ngaco.

## Belum terverifikasi / masih terbuka

- **Kritis, belum dijawab user: pipa 3/4" itu buat rangka atau buat jalur air/manifold?** Kalau buat jalur air, 3/4" itu pas (manifold, T, kran 3/4", cocok buat pompa 1000 L/h). Kalau buat rangka, 3/4" itu kekecilan — rangka harus nahan 40–50 kg bentang 1,5 m, pipa 3/4" PVC bakal melengkung di tengah walau sudah dilem kering total; lem bikin sambungan tidak gerak, tapi tidak bikin pipanya lebih kaku nahan beban. Kalau rangka full pipa, minimal 1,5" atau 2" plus kaki penyangga di tengah tiap 50–75 cm dan penguat silang; atau rangkanya tetap besi hollow/baja ringan, pipanya cuma buat jalur air.
- Lem PVC: 2 jam baru bisa dipegang, 24 jam baru kering total buat nahan tekanan air — jangan dites isi air dulu.
- Setelah pertanyaan 3/4" dijawab, Bre janji bikinin gambar v2 + partlist final. Monitoring ESP32 (tahap 2) belum dibahas.

# AI Content Factory

Ringkasan 17 November 2026 — di WhatsApp (12:47–13:54, 14:17–14:19, 18:37 WIB): user mengupload 2 spek + 10 file mentah dan minta dibandingkan, dicari celahnya, dan dipilih workflow paling efisien. Bre memberi verdict, lalu menjelaskan alurnya pelan-pelan atas permintaan user. Disepakati: **Sprint 0 manual dulu buat mengukur biaya** — dan malam harinya template Remotion-nya sudah dikirim.

## Dua spek yang dibandingkan (12:47)

- User mengupload **Blueprint Perbaikan Arsitektur** dan **Consolidated Spec V11**, lalu 10 file mentah: V2, V3, V4, V5, V6.1, Sprint 1, V8, V9, V10, Master Spec. Permintaan: "Bandingkan 2 ide itu, cari celah dan workflow paling efisien."
- Provenance (12:52, dari user): **Blueprint hasil Gemini Plus (berbayar $4,99/bulan), V11 hasil Claude gratis.**

## Verdict: V11 menang telak

- Ini bukan dua ide yang bersaing — **V11 itu evolusi dari Blueprint**. Blueprint itu bahan mentah hasil analisa V2–V10, V11 itu hasil konsolidasinya. Yang dipakai V11, Blueprint jadi arsip + sumber yang diambil bagusnya.
- **Bukti dari file mentah (Bre mengecek satu-satu):** tabel konflik §24 V11 benar semua lawan file aslinya — urutan Opencode sebelum Narrator (ada di V2 stage 5–6 dan V3); retry Master (Cue 2x, Render/FFmpeg 5x) vs V5 (tanpa Cue); Express (V2) vs Fastify (V9); shared/ (V4/Master) vs packages/ (Sprint 1); 9 sprint (Master) vs 14 (V8); Master 12 queue vs V4 10 queue tanpa storyboard. V11 juga jujur: yang belum terverifikasi ditandai [VERIFY]/[OPEN], angka kapasitas dilabeli "BUKAN data", dan dia tidak mengarang versi library atau harga.
- **Cacat Blueprint:** celah no. 6 setengah salah — dia bilang tabel `users` belum ada, padahal Master Spec §7 sudah menyebut `users` di daftar tabel; yang benar: tabelnya disebut tapi kolomnya tidak pernah didefinisikan. Tabel `prompts` hilang dari DDL-nya, padahal ada di V6.1 dan Master §7. State machine-nya memotong state `storyboard_approved`/`design_approved` yang ada di V4, dan tidak punya state approval-pending. Yang paling fatal: **Muse tetap jadi QA** — Blueprint tidak menangkap masalah bias (Muse menilai karyanya sendiri); V11 menangkap dan memisahkan LLM-judge.
- **Yang Blueprint masih menang:** DDL SQL lengkap yang langsung bisa dipakai (V11 cuma mendeskripsikan tabel, tidak ada skemanya), dan lebih gampang dibaca (analogi rumah produksi).

## Yang V11 perbaiki dari Blueprint (ini yang paling ngaruh ke efisiensi)

1. **Codegen per video → template.** Blueprint menyuruh Opencode menulis kode Remotion baru tiap video — mahal, lambat, gampang error. V11: 5–7 template tetap, Hermes cuma memilih template + mengisi props.
2. **Sprint 0 dulu, bukan bangun infra dulu.** Blueprint langsung 4 minggu membangun Docker, database, bot — video pertama baru keluar minggu ke-4, dan belum tahu bagus apa nggak. V11: spike vertikal dulu, 1 video end-to-end manual/semi-otomatis, ukur kualitas + biaya + waktu render, baru diputus go/no-go.
3. **Muse tidak boleh jadi QA.** Blueprint menyuruh Muse meng-approve karyanya sendiri — bias. V11: QA deterministik dulu, terus LLM-judge terpisah dari Muse.
4. **Human gate.** Blueprint tidak menyebut gate manusia, publish otomatis. V11: Gate A (approve naskah) dan Gate B (approve preview) default ON — penting buat konten keuangan, salah info bisa fatal.
5. **Sandbox + keamanan.** Blueprint render dan eksekusi kode di host yang sama dengan database. V11: render + Opencode di sandbox terpisah, tanpa secrets, tanpa akses DB. Prompt injection dari hasil riset Cue juga ditangani.
6. **Retry berdasar klasifikasi error + budget guard.** Blueprint cuma membatasi revisi QA 2x. V11: error transient boleh retry, error permanen langsung gagal, ada dead-letter, idempotency, outbox, dan hard cap biaya per project.

## Caveat penting: V11 itu redesign, bukan sekadar konsolidasi

- **Sprint 0, template-based Remotion, human gates ON, sandbox, dan classified-error retry adalah keputusan redesign Claude, BUKAN isi file mentah** — file mentah bilang codegen per video, tidak ada Sprint 0, dan bilang "tanpa campur tangan manusia". Jadi V11 itu redesign yang bagus, bukan ringkasan setia dari mentahnya. Ini hasil verifikasi Bre lawan 10 file mentah, bukan klaim V11 sendiri.

## Celah yang masih ada di dua-duanya

- **VPS user tidak cocok.** Spek minta VPS always-on 4 vCPU/8GB. VPS instabox gratis pause tiap 30 menit idle / 8 jam jalan. Factory tidak bisa jalan 24/7 di situ. Spike bisa di desktop/G480, tapi produksi butuh keputusan hosting (Oracle free tier atau VPS berbayar).
- **Biaya LLM vs gratisan.** Spek mengasumsikan provider LLM berbayar. Kunci Gemini user free tier 20 request/hari — satu video saja butuh panggilan Muse + Cue + Hermes berkali-kali. Budget guard V11 jadi wajib, bukan opsional.
- **TTS Indonesia belum terverifikasi.** Kokoro kemungkinan tidak mendukung id-ID. Harus blind test 3 provider dulu (itu bagian Sprint 0).
- **Lisensi Remotion + publish otomatis belum terverifikasi.** V11 jujur menandai [VERIFY] — Blueprint malah langsung mengasumsikan publish otomatis jalan. Jangan bangun publish dulu; manual upload sampai terverifikasi.
- **Niche dan target pendapatan belum diputus di spek** — tapi buat user ini sudah keputus di luar spek: lifehack → affiliate, video pertama naskah 24 jam yang sudah jadi.

## Workflow paling efisien (rekomendasi Bre, diterima user)

1. **Sprint 0 dulu, pakai naskah 24 jam yang sudah ada.** Jangan bangun factory-nya dulu. Spike: TTS test → timestamp → 2–3 template → render → FFmpeg → MP4. Ukur waktu render + biaya. Kalau hasilnya jelek atau mahal, ketahuan sekarang, bukan setelah 4 minggu membangun infra.
2. **V11 jadi single source of truth.** Blueprint diarsip, DDL-nya diambil dan disesuaikan: tabel `analytics` diganti `cost_events`, tambah tabel `prompts` + `approvals`, `job_status` ditambah `cancelled`/`dead`, `projects` ditambah `budget_usd_cap` + `spent_usd`.
3. **Template-based dari hari pertama.** Jangan pernah codegen per video.
4. **Gate A dan B ON.** User approve naskah + preview via Telegram — cocok dengan aturan review-first-nya.
5. **Tunda yang belum perlu:** auto-publish, dashboard, analytics loop, Opencode bikin template baru. Itu Sprint 5–9, bukan sekarang.

## Walkthrough V11, pelan-pelan (13:54)

- User: "V11 ini sebenernya udah oke yaa, tapi gw gatau flow nya. Jelasin pelan pelan dong. Data dari mana kemana, komunikasi tiap agent gimana, kalo kerjaan agent selesai simpen dimana yang lanjutin siapa."
- Bre menjelaskan pakai **analogi rumah produksi (PH) video**: user = klien yang memesan ("buatin video soal aturan 24 jam"); **Mandor (Orchestrator)** = manajer PH — bukan yang membuat video, tapi yang mengatur: menerima pesanan, mencatat di buku besar, lalu menyuruh tim satu-satu sesuai urutan; **Tim (agent)** = tim riset (Cue), sutradara (Muse), desainer (Hermes), tukang suara, tukang render — **mereka tidak pernah ngobrol langsung satu sama lain**, semua ngomongnya cuma ke mandor; **Buku besar (PostgreSQL)** = catatan utama PH — semua pesanan, status, dan hasil kerja dicatat di sini, jadi kalau ada yang sakit atau ganti orang, penggantinya tinggal membaca buku ini terus melanjutkan.
- **Tiga aturan yang membuat semuanya jalan (yang terlihat di penjelasan):**
  1. **Agent tidak pernah ngobrol langsung.** Muse tidak memanggil Cue, Cue tidak memanggil Hermes. Semua lewat mandor: mandor menulis surat tugas (JobRequest — isinya proyek mana, tugas apa, datanya apa) lalu menaruhnya di papan antrean (Redis + BullMQ). Agent yang sedang menganggur mengambil surat itu, bekerja, lalu mengembalikan hasilnya ke mandor (JobResult). Kalau satu agent error, yang lain tidak ikut kena — mandor tinggal memberikan suratnya ke agent lain atau menyuruh ulang.
  2. **Hasil kerja selalu disimpan dulu sebelum lanjut.** Begitu satu agent selesai, hasilnya TIDAK langsung dilempar ke agent berikut. Hasilnya disimpan dulu di buku besar (PostgreSQL): tulisan/JSON-nya masuk tabel `agent_outputs`, file-nya (audio, video, subtitle) masuk tabel `assets` dan disimpan di storage. Baru setelah tersimpan dan dicek (satpamnya bernama Zod — dia mengecek "formulirnya lengkap nggak, formatnya bener nggak"), mandor membuat surat tugas buat agent berikutnya. Ini penting: kalau listrik mati di tengah jalan, tidak ada yang hilang.

## Keputusan: Sprint 0 manual dulu (14:17–14:19)

- User: "Oke kita jalanin manual dulu kan? Buat ngukur biaya?" Bre: iya, manual dulu — itu intinya Sprint 0. Jangan bangun factory sebelum tahu satu video itu biayanya berapa dan hasilnya layak nggak.
- **Langkah selanjutnya, urut:**
  1. **Voiceover dulu (user).** Naskah 24 jam yang sudah jadi dilempar ke AI lain buat TTS. Yang dibutuhkan: file suara + timestamp tiap kata (kapan tiap kata diucapkan) — ini harus duluan, karena template video nanti mengikuti durasi suara asli. Kalau bisa, test 2–3 provider TTS biar kedengaran mana yang paling natural bahasa Indonesianya; kabari providernya apa + biayanya berapa (kalau bayar).
  2. **Template Remotion (Bre).** Begitu suara + timestamp sudah ada, Bre membuatkan 3 template: TitleHook (buat hook), Steps (buat 3 langkah aturannya), CTA (buat penutup). Scene contoh cowo/cewe bisa pakai template Steps juga. Bre menulis kodenya, user yang render di PC-nya.
  3. **Render + gabung (user).** Render video bisu di PC user, terus FFmpeg menggabungkan: video + suara + musik + subtitle. Musik pakai library royalty-free dulu, jangan Suno/Mubert — lisensinya belum terverifikasi.
  4. **Ukur + nilai (bareng).** Yang dicatat: berapa menit render-nya, berapa biaya TTS-nya, berapa lama waktu user dari awal sampai MP4 jadi, dan yang paling penting — menurut user videonya layak tayang nggak? Kalau layak dan biayanya masuk akal → go, baru dibangun Sprint 1 (fondasi factory). Kalau nggak → dibenerin dulu biang masalahnya (misal TTS jelek → ganti provider, render lemot → kurangi template).
- User: "Oke nanti malem kita gas" (14:19).

## Template Sprint 0 dikirim malam harinya (18:37)

- Bre mengirimkan template **TitleHook, Steps, CTA** duluan sebagai file zip **sprint0-remotion.zip** (di workspace/your_files/) — tinggal copy ke PC user, npm install, terus render.
- **Tips TTS dari Bre: coba edge-tts dulu** — gratis, suara Indonesianya lumayan, dan dia memberi timestamp per kata. Kalau AI TTS user tidak bisa memberi timestamp, kirim audionya saja, nanti Bre align pakai Whisper. Video tetap jalan.

## Yang masih menggantung

- Menunggu user: voiceover + timestamp kata dari naskah 24 jam, lalu render di PC user + ukur biaya/waktu → keputusan go/no-go Sprint 1.
- Yang belum terverifikasi: TTS id-ID (blind test 3 provider, bagian Sprint 0), lisensi Remotion, publish otomatis, keputusan hosting produksi (Oracle free tier vs VPS berbayar).

## Terkait

- [[2026-11-17 - Konten Lifehack]] — naskah 24 jam yang dipakai sebagai topik spike Sprint 0.

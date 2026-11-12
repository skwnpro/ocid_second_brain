# Claude Pro

Ringkasan 12 November 2026 — user bertanya berapa jatah token Claude Pro yang $20 (WhatsApp, 13:37 WIB). Lanjutan dari pembahasan 10 November soal rencana langganan Pro dan tools manajemen memory.

## Jawabannya: tidak ada jatah token tetap

Anthropic tidak pernah mempublikasikan angka token untuk Claude Pro. Limitnya bukan "sekian token per bulan", tapi dua jendela pemakaian: **limit 5 jam** (rolling, mulai hitung dari pesan pertama user) dan **limit mingguan**. Pro itu minimal 5x jatah Free per sesi 5 jam.

Patokan yang paling sering dikutip: **~45 pesan per 5 jam untuk Sonnet**. Tapi itu bukan angka kaku — yang dihitung di belakang layar itu token, bukan jumlah pesan. Pesan panjang, file lampiran, dan output yang panjang semuanya menghabiskan jatah lebih cepat. Dan tiap pesan baru, Claude membaca ulang seluruh isi percakapan dari awal, jadi chat yang makin panjang makin boros per pesannya. Model juga berpengaruh: Opus paling boros, Haiku paling hemat. Ada estimasi yang menyebut ~90 pesan per 5 jam setelah limitnya digandakan Mei 2026, tapi itu **bukan angka resmi** — belum terverifikasi.

## Harga dan catatan pemakaian

- **$20/bulan**, atau **$17/bulan kalau bayar tahunan** ($200/tahun).
- **Claude Code memakai pool yang sama dengan chat** — jadi kalau user coding seharian pakai Claude Code, jatah chat ikut kemakan.
- Buat chat biasa, Pro umumnya cukup. Buat coding berat seharian, bisa mentok juga.

## Status — belum diputuskan

User belum menjawab pertanyaan lanjutannya: Pro ini rencananya buat coding atau chat biasa? Jawaban itu menentukan rekomendasi tool memory-nya (lihat file 10 November). Langganan belum dibeli — masih tahap tanya-tanya.

## Terkait

- [[2026-11-10 - Claude Pro dan Manajemen Memory]] — perbandingan tools memory (Memory bawaan, Projects, claude-mem, projectmem) dan rekomendasi awal.

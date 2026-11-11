# Claude Pro dan Manajemen Memory

Ringkasan 10 November 2026 — user berencana langganan Claude Pro dan bertanya ada tidaknya tools manajemen memory biar hemat token (side chat "Claude Pro memory management tools", 14:13 WIB).

## Yang perlu diluruskan dulu

- Limit Claude Pro itu **bukan bayar per token, tapi jatah pemakaian per 5 jam**. Jadi memory yang bagus bukan bikin Pro jadi unlimited, tapi bikin jatah lebih awet karena tidak buang-buang buat mengulang konteks.
- Analogi yang dipakai: memory AI seperti catatan tempel di kulkas — ditulis sekali, dibaca tiap kali masak. Hemat karena tidak perlu mengulang cerita dari nol; boros kalau tempelannya kebanyakan, karena tiap kali masak dia harus baca semuanya dulu.

## Yang sudah gratis di dalam Pro (paling worth)

1. **Memory bawaan claude.ai** — di kiri bawah ada Edit memory; isi sekali identitas, preferensi, gaya kerja; tiap chat baru otomatis kebawa. Diklaim bisa hemat 20–30% token yang terbuang buat pengulangan.
2. **Projects** — upload file sekali, dipakai berkali-kali di semua chat dalam project itu tanpa biaya upload ulang. Buat user ini paling masuk akal: file `.md` cloud brain di GitHub tinggal dijadikan Projects. Klaimnya bisa menambah efektif budget bulanan 30–50% kalau sering upload ulang file yang sama.
3. **Pilih model + /compact di Claude Code** — Pro $20 sudah termasuk Claude Code. Haiku buat tugas receh, Sonnet buat harian, Opus cuma buat yang ribet. Ada auto-compact bawaan, dan Anthropic mengklaim context editing mereka memotong pemakaian token 84% di evaluasi 100-turn.

## Tools luar (khusus coding di Claude Code)

- **claude-mem — paling populer, tapi hati-hati.** Plugin yang otomatis mencatat apa yang dikerjakan, dikompres, lalu disuntik lagi ke sesi berikutnya (SQLite + Chroma, klaim hemat sampai ~10x dibanding load semua history mentah). Masalah nyata: issue #618 — ada user Pro yang jatah 5 jamnya habis dalam <10 pesan setelah mengaktifkannya, apalagi kalau Endless Mode nyala. Kalau pakai ini, matikan Endless Mode, pakai provider lokal.
- **projectmem — yang paling cocok buat user.** Local-first, open-source, menyimpan memory sebagai log di dalam repo sendiri. Support Claude Code, Cursor, sampai Antigravity — nyambung sama setup user sekarang. Klaim hemat 50%+ token, dan ada mode `pjm wrap` yang menyuntik memory dengan budget token yang diatur sendiri (500–2000 token per sesi).
- **Mem0 / OpenMemory + Letta — tidak direkomendasikan buat user.** Mem0 klaim hemat >90% token, tapi ada trade-off akurasi: di benchmark LoCoMo, full-context biasa malah menang ~73% vs Mem0 ~68,5%. Letta (dulu MemGPT) sudah geser jadi produk coding-agent sendiri, server memory V1-nya diarsip. Buat kerjaan coding/Brush Kart yang butuh detail akurat, ini berisiko.
- **Headroom** — proxy kompresi buat output tool/log sebelum sampai ke model (contoh vendor: prompt 55.957 token dikompres jadi 24.340, sekitar 56% lebih hemat). Lebih kepakai kalau bayar API per token, bukan Pro.

## Penilaian jujur buat kasus user

- User itu sudah punya sistem memory paling benar: file `.md` di `ocid_second_brain`. Itu externalized memory, bisa dibaca Muse, Claude, Gemini, siapa saja. Tools seperti Mem0 malah mengunci ke satu ekosistem.
- Urutan yang disarankan: (1) kalau cuma chat di web, cukup Memory + Projects bawaan, tidak usah install apa-apa; (2) kalau coding, tambah `CLAUDE.md` yang tipis di tiap repo + Skills; (3) kalau masih kurang, baru coba projectmem karena lokal dan repo-based.
- Yang paling sering bikin boros itu bukan tidak adanya memory, tapi memory yang kegedean — ingatan tetap harus dikirim ulang tiap turn, jadi makin gendut memory, makin makan jatah. Pruning lebih penting dari menambah tool.

## Masih pending

- User belum menjawab pertanyaan penutup: Claude Pro ini mau dipakai buat apa dulu — coding Brush Kart / finance bot, atau chat umum saja? Jawabannya menentukan tool yang beda.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — soal yang sama dari sisi lain: kenapa tiap balasan agent mahal token dan cap compaction 256K yang dipilih user.

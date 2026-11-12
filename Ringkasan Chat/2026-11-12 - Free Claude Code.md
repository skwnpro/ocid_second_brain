# Free Claude Code

Ringkasan 12 November 2026 — user minta dibacakan repo https://github.com/Alishahryar1/free-claude-code, lalu bertanya soal token, langganan, dan bedanya dengan 9Router (WhatsApp, 16:10–16:17 WIB).

## Intinya

Free Claude Code (FCC) adalah alat open-source untuk memakai **coding agent seperti Claude Code tanpa langganan Claude**. Cara kerjanya: jadi **proxy lokal** di komputer user (`fcc-server`, jalan di `127.0.0.1:8082`). User menjalankan agent lewat FCC — misal `fcc-claude` untuk Claude Code, `fcc-hermes` untuk Hermes — lalu FCC meneruskan request ke model dari provider lain yang dipilih, termasuk provider gratis. Agent-nya tetap Claude Code (atau Codex, OpenCode, Cline, Aider, dll. — total 12 harness didukung), tapi otak di belakangnya bukan model Anthropic.

## Yang ditawarkan

- **61 provider**, termasuk yang gratis — klaimnya total 1,3 miliar token gratis per bulan (jatah gratis itu aturan masing-masing provider, bisa berubah sewaktu-waktu).
- **Fallback otomatis** — kalau satu provider/model mati atau kena rate limit, pindah sendiri ke provider lain. Plus retry otomatis dan multi-API-key.
- **Hemat token** — klaim sampai 90% lebih hemat pakai RTK + Headroom.
- Bisa dipakai dari terminal, VS Code/JetBrains, desktop app, sampai HP via Discord/Telegram (voice note juga bisa, pakai Whisper lokal).

## Tiga jawaban buat user

1. **Tokennya dari mana?** Bukan dari FCC — FCC cuma router, tidak memberi token sama sekali. Token datang dari **provider yang disambungkan** ke FCC, 4 jenis: gratis (misal NVIDIA NIM API key gratis, OpenRouter model free, Groq free tier); API key berbayar (isi saldo, bayar per token); langganan (misal ChatGPT atau GitHub Copilot, via OAuth); lokal (model jalan di mesin sendiri, misal Ollama, tanpa provider sama sekali).
2. **Harus langganan?** Tidak. FCC-nya sendiri **gratis**, open-source. Yang mungkin bayar itu providernya — dan itu pun pilihan user. Mau nol rupiah, pakai yang gratis saja.
3. **Bedanya sama 9Router?** Ide dasarnya sama: dua-duanya proxy lokal yang menyambungkan user ke banyak provider lewat satu endpoint. Bedanya di fokus: **9Router itu router umum** — buat apa saja (Hermes user, chat, dll.), ada dashboard dan katalog model; **FCC itu khusus coding agent** — dia mengerti harness-nya (Claude Code, Hermes, dll.), jadi streaming, tool use, dan thinking tetap jalan native. Plus fallback otomatis, penghemat token, Admin UI + model picker yang nyambung ke Claude Code. Bonus: keduanya **bisa ditumpuk** — FCC bisa mendaftarkan provider custom, jadi 9Router user (yang sudah nyambung Gemini) bisa didaftarkan di FCC sebagai provider.

## Relevansi buat user

- Ini fungsi utamanya: **coding agent**. Hermes user bisa jalan lewat FCC juga (`fcc-hermes`), jadi providernya tidak harus cuma Nous/Gemini — bisa fallback ke provider gratis lain kalau yang satu mati.
- **Claude Code bisa dicoba tanpa Pro $20** — tapi ingat, model di belakangnya bukan Claude beneran, jadi kualitas codingnya tergantung model gratis yang dipilih. Buat tugas ringan-sedang oke, buat yang kompleks jangan harap setara Opus/Sonnet.
- **Antigravity CLI juga didukung** (`fcc-agy`), nyambung sama setup KiCad/FreeCAD user.
- FCC jalan lokal di mesin user (Python), bukan layanan cloud.

## Catatan jujur

- Gratisnya itu jatah dari providernya masing-masing, bukan dari FCC — kalau provider gratisnya menutup keran, ya habis. Bukan jaminan abadi.
- Belum ada perintah install dari user — baru tanya fungsinya dulu. Kalau mau coba install, tinggal bilang.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — 9Router user yang jadi jalur Hermes ke Gemini, pembanding langsung buat FCC.
- [[2026-11-11 - Browser untuk AI Agent]] — tooling agen AI lain yang dibahas minggu ini.

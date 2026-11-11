# OpenMausBot

Ringkasan 11 November 2026 — user minta dibacakan dan dijelaskan repo github.com/milind-soni/OpenMausBot (main chat, 17:40 WIB).

## Intinya

Aplikasi chat gaya Telegram, tapi tiap kontak di sidebar itu **bot AI beneran** — bukan chatbot biasa. Tiap bot punya kepribadian, model, komputer, dan aplikasi terhubung sendiri. Proyek open-source (Apache 2.0, ~21 ribu star, ~2.900 fork), terinspirasi Grok Bot-nya xAI, diposisikan sebagai alternatif terbuka untuk Grok Bot, Meta Muse, OpenAI dots, Cue (Manus), dan Instinct.

## Cara kerjanya

- **Otaknya bawa sendiri.** Bot jalan di atas CLI `claude`, `codex`, atau `grok` yang sudah terinstal di komputer user — pakai login/subscription yang sudah ada. Tiap bot bisa pakai model berbeda, bisa diganti di tengah percakapan.
- **Tiap bot punya komputer.** Bisa desktop cloud (lewat Boat), VM lokal yang terisolasi, atau komputer user sendiri. Layar live bisa dilihat saat bot bekerja, atau diambil alih dari browser.
- **Bot bertanya dulu sebelum bertindak.** Perintah shell, edit file, dan pertanyaan muncul sebagai kartu approval — Allow / Deny / jawab di chat.
- **Aplikasi terhubung lewat Composio:** Gmail, Slack, GitHub, Notion, Linear, dan ratusan lagi. OAuth sekali, semua bot bisa pakai.
- **Channel terpisah** untuk kerja, pribadi, dan tiap proyek — masing-masing dengan transcript, instruksi, dan daftar bot sendiri.
- **Tim instan:** satu file Markdown bisa menginstal satu tim lengkap (bot, channel, playbook, rutinitas); satu departemen bisa di-share sebagai satu file.
- **Bisa bicara** (ElevenLabs, Fish Audio, Grok, atau suara bawaan Mac), **trigger** berupa jadwal dan webhook, serta bisa dikendalikan lewat MCP server atau CLI `openmausbot`.
- **Menjalankannya:** ada build jadi untuk macOS, Windows, dan Ubuntu 24.04 (.deb atau AppImage); dari source butuh Node 24+ dan pnpm plus minimal satu CLI agent yang sudah login. Server jalan lokal di `127.0.0.1`, transcript dan key disimpan di `~/.openmausbot`, bukan di cloud.

## Catatan jujur buat user

- **Engine-nya cuma Claude, Codex, dan Grok — Gemini tidak disebut sama sekali.** Jadi Gemini Plus user tidak bisa dipakai di sini; butuh login salah satu dari tiga itu.
- Ubuntu masih status **beta**, dan kontrol komputer lokal di **Wayland dimatikan** (hanya jalan di Xorg, itu pun harus opt-in eksplisit).
- Komputer cloud-nya (Boat) adalah layanan berbayar setelah trial — bukan gratis selamanya.

## Relevansi buat user

Ini pada dasarnya **versi open-source dari setup Hermes multi-agent yang sedang user bangun** — hanya bentuknya aplikasi chat desktop, bukan grup Telegram, dan otaknya memakai CLI agent yang sudah ada di mesin user.

## Terkait

- [[2026-11-10 - Grup Telegram Multi-Agent]] — setup multi-agent Hermes user yang sedang berjalan, pembanding langsung untuk OpenMausBot.

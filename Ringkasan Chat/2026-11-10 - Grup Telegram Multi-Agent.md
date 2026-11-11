# Grup Telegram Multi-Agent

Ringkasan 10 November 2026 — user mau meniru setup multi-agent Telegram dari sebuah screenshot: satu grup forum dengan banyak topik, satu bot per persona, rekap malam (side chat "Free Instabox VPS agent ideas", 16:32–16:45 WIB).

## Yang mau ditiru dari screenshot

- **Grupnya forum group (Topics):** sidebar berisi topik HQ, Kas, Threads & Leads, General, Absensi, Studio Konten, Server & Bot. Satu grup, dipecah jadi banyak thread.
- **Tiga bot beda persona** (di screenshot bernama LALISA, RUKA, JENNIE): di Telegram satu bot cuma bisa punya satu nama + satu foto, jadi tiap persona = satu bot sendiri, satu token sendiri dari BotFather. Tulisan hijau "Divisi 1 / Divisi 2" di bawah nama itu custom title admin grup — bot dijadikan admin lalu diberi title.
- **Rekap jam 21:00** itu cron — tiap agen punya jadwal sendiri buat posting rekap ke topiknya.

## Arsitektur yang disarankan pakai Hermes

- Satu install Hermes bisa pegang banyak profile; satu profile = satu agen (config, SOUL, memory, skill, provider/model sendiri-sendiri) = satu bot Telegram dengan token sendiri.
- Langkah bangunnya: bikin bot-bot di BotFather (nama + foto profil), di VPS bikin satu profile Hermes per bot, masukkan semua bot ke grup, jadikan admin, kasih custom title, tiap profile dikasih cron buat rekap malamnya.
- **Dua catatan jujur yang disampaikan:** ini bukan Hermes-nya FNATIX (studio di screenshot) — agen mereka bisa jadi framework lain, tapi polanya sama persis dan bisa ditiru pakai Hermes. Dan VPS gratisan user tidur kalau idle 30 menit, jadi rekap jam 21:00 cuma kekirim kalau VPS lagi bangun pas jadwalnya.
- **Privacy mode bot:** default-nya bot di grup cuma dengar kalau di-mention atau di-reply. Kalau mau bot nyautin obrolan bebas, privacy dimatikan via BotFather — konsekuensinya bot membaca semua pesan di grup itu.
- **Teman user harus di-allowlist:** kemungkinan bot cuma mau menjawab user yang diizinkan; butuh user ID Telegram temannya buat didaftarkan. Grup Telegram saja — Hermes tidak bisa dimasukkan ke grup WhatsApp.

## Yang sudah dilakukan user, dan yang mentok

- User sudah membuat grupnya (nama berawalan "Ocid r...") dan bertanya cara bikin grupnya (16:35 WIB) — langkahnya dijelaskan: Telegram → ikon pensil → New Group → pilih 1 anggota dulu → beri nama → Create; lalu nyalakan Topics (tap nama grup → ikon pensil/Edit → toggle Topics), bikin topik-topiknya, tambahkan bot sebagai member lalu promote jadi admin dan isi Custom Title.
- **Mentok di toggle Topics (16:45 WIB):** user membuka menu titik tiga tapi isinya cuma Unmute, Search, Boost, Clear History — bukan itu tempatnya. Yang benar: tutup menu itu, tap **nama grup di atas**, tap ikon **pensil (Edit)**, scroll ke bawah, cari **Topics** dan nyalakan toggle-nya. Kalau di halaman Edit tidak ada "Topics" sama sekali, dua kemungkinannya: Telegram user perlu di-update dari Play Store, atau grupnya masih basic group — cari "Group Type" di halaman Edit yang sama, convert ke supergroup dulu baru toggle Topics muncul. Belum ada konfirmasi user apakah sudah ketemu.

## Masih pending (belum dijawab user)

1. **Berapa agen + peran masing-masing** (ikut contoh screenshot atau peran sendiri?).
2. **Username/ID Telegram temannya** (buat allowlist dan tambah ke grup).
3. **Bot-botnya dibuat dulu di BotFather** — ini user yang buat sendiri, tokennya nanti dikasih pas wiring.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — Hermes yang dipakai sebagai mesin tiap agen, dan batas VPS gratisannya.

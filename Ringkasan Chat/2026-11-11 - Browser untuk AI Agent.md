# Browser untuk AI Agent

Ringkasan 11 November 2026 — user minta dicarikan info soal browser untuk AI agent yang ada di vault (side chat "Free Instabox VPS agent ideas", 09:18 WIB). Hasilnya: tiga sumber relevan ditemukan di vault.

## Yang ditemukan di vault

1. **Claude in Chrome** — `Ringkasan Chat/2026-11-10 - Claude in Chrome.md`
   Extension resmi Anthropic: Claude membaca halaman lalu klik, ketik, isi form, dan pindah tab sendiri, memakai login Chrome user. GA sejak 26 Agustus 2026 untuk semua paket Claude berbayar. Batasannya: butuh Claude Pro ($20/bln, user hanya punya Gemini Plus), hanya jalan di Chrome desktop, dan risiko prompt injection-nya nyata — Anthropic sendiri bilang proteksinya "aren't foolproof", jangan dipakai untuk banking/data kesehatan, red-team mode otonom pernah tembus 23,6%. CAPTCHA juga masih jadi tembok.

2. **Lightpanda** — `Threads/doomscroll dump/2026-11-01.md`
   Browser headless yang ditulis dari nol pakai Zig, bukan fork Chromium. Klaimnya: 16x lebih hemat memori dan 9x lebih cepat dari Chrome headless, ada mode agent + server MCP bawaan sehingga AI agent bisa mengendalikannya langsung, halaman bisa di-dump jadi Markdown/PNG/PDF. Saat post itu ditulis repo-nya sudah 39,2 ribu stars. Catatan: angka 16x/9x itu klaim penulis post — uji dulu di satu workload nyata sebelum mengganti tool yang sudah jalan. Tapi ini paling nyambung dengan mesin terbatas user (VPS gratis / G480).

3. **Claude mengoperasikan Gemini lewat browser** — `Threads/doomscroll dump/2026-11-06.md`
   Post soal nge-prompt AI pakai AI: Claude mengendalikan browser, browser mengendalikan Gemini. Catatan: rantainya banyak titik gagal (login, perubahan UI, biaya token Claude) — coba dulu untuk satu aset, bandingkan waktunya dengan menulis prompt manual. Hanya layak untuk batch besar.

## Yang tidak ada di vault

- **Vercel Labs agent-desktop** yang user tanyakan 31 Oktober lalu — itu hanya dibahas di chat, belum pernah disimpan jadi catatan. Asisten menawarkan untuk membuatkan ringkasannya lalu menyimpan ke vault; user belum menjawab tawaran itu.

## Terkait

- [[2026-11-10 - Claude in Chrome]] — detail lengkap Claude in Chrome, salah satu dari tiga sumber di atas.

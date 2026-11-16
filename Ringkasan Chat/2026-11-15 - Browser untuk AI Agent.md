# Browser untuk AI Agent

Ringkasan 15 November 2026 — user mengirim link repo GitHub dan minta dibaca (WhatsApp, ~14:58 WIB): Vercel Labs **agent-browser**, saudara kembar agent-desktop yang ditanyakan 31 Oktober lalu.

## agent-browser: CLI Rust buat otomasi browser khusus AI agent

- **Cara kerja:** loop pendek lewat shell — `agent-browser open example.com`, lalu `agent-browser snapshot -i` mengembalikan halaman sebagai teks accessibility tree (tiap elemen punya ref seperti `@e1`), lalu `agent-browser click @e1`, `agent-browser get url` untuk cek hasil. Browser tetap terbuka antar command; di belakang layar ada daemon yang mengendalikan browser lewat Chrome DevTools Protocol. Daemon mati sendiri kalau 1 jam tidak ada perintah (state tersimpan kalau persistence nyala).
- **Bukan screenshot-guessing.** Snapshot-nya teks, jadi hemat token dan ref-nya stabil. Klik gagal beneran kalau target ketutupan banner atau di luar viewport — bukan klaim sukses palsu.
- **Ada skill bawaan dan MCP server.** `agent-browser skills get core` memberi instruksi loop-nya ke agent, dan `agent-browser mcp` bisa dicolok ke Claude Code, Hermes, atau harness lain.
- **Fleksibel.** Default-nya Chrome for Testing, tapi bisa nempel ke Chrome yang sudah jalan (`--cdp`), ganti engine (Lightpanda, Obscura), Mobile Safari di iOS Simulator, sampai cloud provider seperti Browserbase dan Browserless.
- **Command-nya lengkap.** Bukan cuma klik-isi: ada screenshot, record, PDF, cookies, network, console error, profiler, sampai React/a11y inspector.
- **Security dipikirin, tapi tidak hilang risikonya.** Ada auth vault, domain allowlist, dan action policy — tapi risiko klasik browser agent (prompt injection dari isi web) tidak hilang cuma gara-gara ada allowlist.

## Angka dan perbandingan

- Repo Vercel Labs, lisensi Apache-2.0, dibuat Januari 2026, sekarang **51 ribu star, 3.500-an fork**, terakhir update hari ini (masih aktif). Sebagai perbandingan, agent-desktop kemarin sekitar 3.300 star.

## Buat user

- Nyambung langsung dengan riset browser buat AI agent tanggal 11 November — ini jawaban paling konkretnya. Jalan di Linux (PC user dan VPS), install-nya `npm install -g agent-browser` lalu `agent-browser install`.
- Kepakai kalau user mau Hermes atau agent lain bisa nge-browse beneran, misal riset referensi video viral atau cek dashboard.
- Catatan jujur asisten: ini tool, bukan agent jadi — dia memberi "tangan" ke browser, otaknya tetap agent-nya.
- **Belum diputuskan:** asisten menawarkan mau di-install/dicoba atau sekadar disimpan dulu buat referensi — user belum menjawab tawaran itu.

## Terkait

- [[2026-11-11 - Browser untuk AI Agent]] — tiga sumber sebelumnya (Claude in Chrome, Lightpanda, Claude mengoperasikan Gemini) dan catatan agent-desktop yang belum tersimpan di vault.

# Hermes Balik ke Gemini

Ringkasan 10 November 2026 — job sekali jalan jam 15:00 WIB mengembalikan Hermes dari Nous ke Gemini setelah kuota gratisan reset (side chat "Free Instabox VPS agent ideas", laporan job 15:01 WIB).

## Yang terjadi

- **VPS sempat tidur, dibangunin dulu.** `finance-bot` dalam keadaan suspended saat job jalan; di-resume sampai status running sebelum lanjut.
- **Tes kuota pertama gagal, penyebabnya bukan kuota.** 9Router menjawab 404 "No active credentials for provider: gemini" — ternyata **kedua koneksi Gemini di 9Router nonaktif**. Koneksi yang pakai kunci API `.env` ternyata auto-deactivated jam 10:40 WIB setelah kena 429 pagi tadi. Koneksi OAuth yang mati pagi tadi juga tetap nonaktif.
- **Perbaikannya:** koneksi `.env`-key diaktifkan lagi saja, 9Router di-restart, tes ulang ke `gemini/gemini-3.6-flash` balik HTTP 200 dengan jawaban "4" — kuota sudah reset.
- **Switch-back dieksekusi:** config Hermes dipulihkan dari backup `~/.hermes/config.yaml.bak-before-nous-switch` (provider custom, `gemini-3.6-flash`, cap compaction 256K tetap), gateway di-restart (`hermes-gateway.service`), verifikasi `hermes -z "jawab dengan satu angka saja: 2+2"` → "4", gateway active.
- **Catatan teknis dari eksekusi:** tabel API key 9Router namanya `apiKeys` (bukan `api_keys` seperti tertulis di body job), dan sqlite3 CLI tidak ada di VPS jadi dibaca pakai python3. Tidak ada key/token yang tercetak di log mana pun.

## Status akhir

- Hermes kembali jalan di Gemini dengan jatah fresh 20 request/hari — habis lagi sekitar besok jam 15:00 WIB kalau dipakai terus-terusan.
- **Fallback otomatis masih belum terpasang** (butuh `hermes fallback add` interaktif dari terminal user sendiri). Jadi kalau Gemini mati lagi, Hermes tidak pindah sendiri ke Nous.
- Kunci akses instabox CLI user kedaluwarsa 16 November 2026 — setelah itu membangunkan VPS butuh persetujuan login ulang dari user.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — kejadian Gemini/Nous pagi harinya dan kenapa kuota habis.

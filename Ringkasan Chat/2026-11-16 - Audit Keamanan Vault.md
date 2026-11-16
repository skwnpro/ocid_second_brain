# Audit Keamanan Vault

Ringkasan 16 November 2026 — di WhatsApp (13:44–14:02 WIB): user mau menjadikan vault `ocid_second_brain` public supaya Gemini, Claude, dan GPT gratisannya bisa membacanya sebagai context. Sebelum flip, Bre mengaudit seluruh isi repo untuk mencari info rahasia.

## Permintaan dan cakupan audit

- User: "cek repo second brain, ada info rahasia gw ga? Gw mau jadiin public biar gemini, claude, gpt gratisan gw bisa baca dan jadiin context itu catatan gw. Cek dulu, list info yang rawan."
- **Cakupan:** 83 file di HEAD terbaru (commit terakhir semalam jam 00:34), dibaca satu per satu — termasuk folder `.trash`, `.obsidian`, dua folder finance (yang satu typo `finnance_assistant`), sampai riwayat commit lama. Yang dicari: token/key/password, nomor HP, email, alamat, NIK/NIP, saldo dompet, utang-piutang, info keluarga, IP server, sampai URL yang ada tokennya.
- **Catatan awal Bre:** public artinya semua orang di internet bisa baca, bukan cuma AI user — jadi daftar ini penting sebelum flip.
- Audit sempat di-stop user sekali ("Stop proses"), lalu dilanjutkan atas perintah user: "Baca aja, jangan lakukan hal lain. Cek ada key atau password yang disana, kalo gaada bilang."

## Temuan: 2 password beneran, tidak ada API key/token

- **Password login + sudo user G480** — tertulis di `Ringkasan Chat/2026-11-14 - Penyelamatan Data G480.md` (baris 8), masih hidup. Ironisnya, di file yang sama tertulis "Passwordnya tidak disimpan di mana-mana" — padahal file itu sendiri yang menyimpannya.
- **Password dashboard 9Router di G480** — tertulis di `Finance Assistant/9Router_Hermes_Setup.md` (baris 99–104, 232, 236), termasuk contoh curl yang memakai nilai itu, total 6 tempat. Yang di VPS sudah diganti 9 November kemarin, tapi yang di **G480 masih bawaan/default** sampai sekarang. Dashboard-nya localhost-only, jadi risikonya lebih kecil — tapi nilainya tetap ada di repo.
- **Yang bersih:** tidak ada API key / token beneran. Semua `TELEGRAM_BOT_TOKEN`, `GEMINI_API_KEY`, `DB_PASSWORD` di file-file Finance Assistant hanya placeholder (`<redacted>`, `PASSWORD-KUAT`, `GANTI-DENG...`). Tidak ada pola `nvapi-`, `ghp_`, `sk-ant-`, private key, atau URL publish instabox yang ada tokennya. Insiden kunci NVIDIA + token Nous bocor 14 Nov kemarin hanya *diceritakan* di `Ringkasan Chat/2026-11-15 - Hermes.md` — nilainya tidak ikut tertulis. Kunci 9Router hanya ada ID + prefix-nya, kunci lengkapnya di `~/.9router/db/data.sqlite`, bukan di repo. Folder `.obsidian` dan `.trash/Finance Assistant.md` juga bersih (catatan: `.trash` ikut ke-commit walau ada di `.gitignore`, jadi kalau repo jadi public dia ikut terbuka).

## Dibersihkan dan di-push (14:02)

- User: "oke hapus password". **Dua-duanya dihapus dari repo dan sudah ke-push (commit `dd4d7b72`)**, Bre memverifikasi dengan baca ulang dari GitHub langsung:
  - `Ringkasan Chat/2026-11-14 - Penyelamatan Data G480.md` — nilai password G480 sudah tidak ada.
  - `Finance Assistant/9Router_Hermes_Setup.md` — password dashboard 9Router hilang dari 6 tempat, diganti placeholder.

## Yang wajib dilakukan SEBELUM repo jadi public (belum beres)

- **Ganti password user G480.** Hanya menghapus dari file tidak cukup — commit lama di history Git tetap menyimpan password itu. Yang membuat aman itu rotasi, bukan hapus.
- **Ganti password dashboard 9Router di G480 dari nilai bawaannya.**
- **History Git belum di-scan** — audit hanya mengecek file saat ini. Kalau mau, itu cek lanjutan.
- **5 link Google Drive di `Threads/doomscroll dump/2026-10-28.md`** — bukan kredensial, tapi kalau file Drive-nya di-share "anyone with the link", link itu ikut terbuka bareng repo.

## Terkait

- [[2026-11-14 - Penyelamatan Data G480]] — file yang menyimpan password G480.
- [[2026-11-16 - Hermes]] — password dashboard 9Router G480 dan VPS yang masih harus diganti; rotasi kunci NVIDIA + token Nous juga masih menggantung.

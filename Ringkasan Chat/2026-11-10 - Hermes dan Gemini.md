# Hermes dan Gemini

Ringkasan 10 November 2026 — semua kejadian hari ini soal Hermes, Gemini, dan 9Router. Bukan panduan instalasi (itu sudah ada di `Finance Assistant/9Router_Hermes_Setup.md`).

## Kuota Gemini gratisan habis dua kali

- **Pagi, 08:15 WIB:** koneksi Gemini di 9Router yang dipasang pakai token login Google (OAuth) kena 429 "You exceeded your current quota" jam 08:19, lalu 9Router menonaktifkannya. Hermes error `404 No active credentials for provider: gemini`. (Side chat "Free Instabox VPS agent ideas", ~08:22 WIB)
- **Kunci API `.env` ternyata sehat.** Tes langsung ke Google pakai `gemini-3.6-flash` balik 200, jawab "4". Jadi yang habis itu kuota token login OAuth, bukan kuncinya. Solusinya: koneksi Gemini baru dibuat di 9Router pakai kunci API dari `.env`, 9Router di-restart, `hermes -z` balik "4". Koneksi OAuth yang mati dibiarkan nonaktif. Hermes sendiri tidak perlu diubah.
- **Siang, ~09:00 WIB:** giliran kunci `.env` sendiri yang kena limit free tier Google — 20 request/hari untuk `gemini-3.6-flash`, reset sekitar jam 15:00 WIB (tengah malam Pasifik). Pemakaian Hermes jam 08:28–08:59 menghabiskan sisanya. Tiap balasan Hermes = 1 request, dan kalau gagal dia retry 3 kali, jadi satu obrolan gagal bisa makan 3–4 jatah sekaligus.

## Pindah sementara ke Nous

- Hermes dipindah sementara ke Nous `upstage/solar-mini4:free` (settingan awal sebelum 9Router), dites jawab "4", gateway jalan. User sempat memilih "Biarin di Nous dulu" (09:05 WIB), lalu satu menit kemudian minta balik ke Gemini jam 15:00 (09:06 WIB).
- **Job sekali jalan `hermes-switch-back-gemini-1500`, Selasa 10 Nov 15:00 WIB:** bangunkan VPS kalau lagi pause, tes kuota Gemini dulu lewat 9Router, balik ke Gemini hanya kalau tesnya 200; kalau kuota belum reset, tetap di Nous dan lapor di chat. (Saat ringkasan ini ditulis, job belum jalan.)
- **Fallback otomatis BELUM terpasang.** `hermes fallback add` itu interaktif, hanya bisa dari terminal user sendiri. Disarankan pilih Nous `solar-mini4:free` biar kalau Gemini mati lagi, Hermes pindah sendiri.

## Kenapa bot pencatatan awet, Hermes boros

- Limit 20/hari berlaku sama untuk keduanya — kunci sama, model sama (`gemini-3.6-flash`), bot pencatatan tidak punya privilege apa-apa. Bedanya di frekuensi panggilan: bot pencatatan hanya manggil Gemini untuk teks bebas dan foto struk; perintah `/saldo`, `/riwayat`, `/utang`, `/dompet`, `/audit`, `/batal` cuma baca database (nol panggilan). Hermes sebaliknya: tiap balasan = 1 request.
- Bot pencatatan juga pernah kena tembok yang sama: 8 kali error Gemini di log, terakhir 29 Oktober. Bedanya, saat Gemini mati dia balas "AI parser sedang bermasalah, coba lagi sebentar" dan transaksinya gagal dicatat.

## Langganan AI Plus tidak menaikkan kuota API

- Google AI Plus ($4,99/bln) hanya untuk aplikasi Gemini, Gmail, dan kawan-kawan — **tidak termasuk API**. API itu jalur terpisah dengan free tier sendiri; Plus cuma dapat akses AI Studio "Limited". Bahkan naik ke Pro ($19,99) pun API tetap terpisah (cuma dapat kredit Google Cloud $10/bln). Satu-satunya jalan menaikkan kuota API: aktifkan Cloud Billing di project Google Cloud (bayar per token). (Side chat "Free Instabox VPS agent ideas", 10:35 WIB)

## Jalan buntu: Nvidia dan OpenCode

- **Nvidia lewat 9Router buntu:** semua 7 model chat di katalog sudah mati — 5 kena 410 end-of-life (Juli–Sep 2026), 2 lagi 404 tidak di-deploy untuk akun user.
- **OpenCode:** koneksinya tidak tersimpan sama sekali di 9Router; model `oc/*` butuh API key opencode.ai sendiri (401 Missing API key).

## Hemat token: cap compaction 256K

- Tiap balasan agent mengirim ulang seluruh konteks ke model. Bukti di instalasi sendiri: perintah "2+2" saja kena 2085 prompt token sebelum satu kata riwayat pun. Compaction Hermes jalan di 50% context window (model 1 juta token → 524 ribu) plus safety net gateway di 85%.
- User memilih cap **256K** ("Oke 256k aja amannya", 08:49 WIB): `compression.threshold_tokens: 256000` + `idle_compact_after_seconds: 1800` (thread Telegram yang diam 30 menit dimampatkan dulu sebelum dibalas). Config lama di-backup ke `config.yaml.bak-before-compaction-cap`, gateway di-restart, `hermes -z` balik "4". Penjelasan 128K vs 256K disampaikan pakai analogi buku catatan: makin tebal bukunya, makin mahal tiap jawaban; cap itu batas tebal buku sebelum Hermes merapikannya jadi ringkasan.

## Dashboard Hermes dipublish

- User minta link dashboard Hermes seperti 9Router kemarin (10:51 WIB). Dashboard jalan di port **9119**, dinyalakan manual via nohup (bukan service — mati kalau VPS ke-pause/restart), lalu dipublish private seperti 9Router.
- Link lengkap (yang ada tokennya) hanya muncul sekali saat publish dan tersensor di Bre — user mengambil sendiri dari terminalnya (Termux/PC) dengan `instabox port publish finance-bot 9119 --private --rotate-token`. Ambil link = dari luar VPS (CLI instabox); cek/nyalakan dashboard = dari terminal VPS (`hermes dashboard --status`, atau nyalakan lagi dengan `nohup hermes dashboard --port 9119 --host 127.0.0.1 --no-open`).
- **Peringatan keamanan:** dashboard ini tidak ada loginnya — karena di-bind ke loopback, gerbang auth-nya tidak aktif, jadi token di link itu satu-satunya pelindung. Siapa pun yang pegang link bisa buka dan utak-atik config, API keys, dan sessions. Jangan dishare; unpublish kalau selesai (`instabox port unpublish finance-bot 9119`).

## Terkait

- [[2026-11-10 - Lubuntu untuk Laptop G480]] — laptop G480 disiapkan jadi server yang nantinya menjalankan Hermes/9Router/n8n.
- [[2026-11-10 - Peringatan PAT GitHub]] — Hermes/Cue sempat meminta PAT GitHub, dijawab dengan peringatan keamanan.

# Bug Driver Nvidia

Ringkasan 26 November 2026 — di chat WhatsApp (06:35 WIB): user mengirim reel Instagram soal bug driver Nvidia dan minta diingatkan jam 21:00 malam ini. Bre sudah menyetel pengingatnya.

## Isi reel (dari @elevenbyte)

- **Bug di driver Nvidia 621.14 / 621.24:** proses `NVDisplay.Container.exe` menulis ulang file log `nvlog` terus-terusan, sehingga SSD ditulis **3–30+ GB per jam** walau PC sedang nganggur.
- **Solusinya:** update ke driver **621.40 WHQL**, atau rollback ke **621.01** buat RTX 40 ke bawah.
- **Cara ceknya:** buka Resource Monitor (resmon) → tab Disk → urutkan kolom Write, lihat apakah `NVDisplay.Container.exe` menulis besar terus-menerus.
- Sumber: reel Instagram https://www.instagram.com/reel/DgJrlDtOX08/ (dari @elevenbyte), dikirim user 06:35 WIB. Bre mengonfirmasi sudah menyetel pengingat jam 21:00 malam ini.

## Belum terverifikasi / masih terbuka

- Info bug, nomor versi driver, dan solusinya **hanya dari reel** — Bre belum memverifikasi secara independen dari sumber lain.
- **Belum diketahui** apakah PC user memakai driver versi yang kena bug (621.14/621.24) atau tidak — user belum mengecek via resmon.
- Pengingat jam 21:00 malam ini sudah disetel; tindak lanjutnya (cek resmon, update/rollback driver) menunggu user.

# Lubuntu untuk Laptop G480

Ringkasan 10 November 2026 — menyiapkan Lubuntu buat laptop Lenovo G480 (RAM 4 GB, masih HDD, Windows 7 rusak, data di partisi D harus selamat).

## Tutorial Startup Disk Creator: buat bikin USB, bukan install langsung

- User mengirim tutorial Ubuntu "create a USB stick" dan bertanya apakah bisa install Lubuntu pakai itu, atau sekadar try saja (main chat, 10:12 WIB). Dijelaskan: tutorial itu untuk **membuat USB installer** via Startup Disk Creator, bukan install langsung.
- **"Try Lubuntu" bisa dan itu langkah yang benar:** boot dari USB, pilih Try Lubuntu — jalan live dari USB/RAM tanpa menyentuh harddisk sama sekali. Dari live session itu dicek dulu D masih kebaca atau tidak, lalu file penting di-copy ke media backup, baru install.
- Aturan tetap saat install nanti: partisi manual, format C saja, jangan sentuh D.

## Download resmi: lubuntu.me 404, pakai cdimage

- User minta link download resmi; link `lubuntu.me/downloads` dikembalikan 404 di sisinya (main chat, 10:14 WIB). Penggantinya: server resmi Ubuntu (cdimage.ubuntu.com), diverifikasi langsung dan masih hidup saat dicek.
- Versi yang dipakai: **Lubuntu 26.04.1 LTS Desktop 64-bit** (Resolute Raccoon), support sampai April 2029. Bukan 26.10 (interim, support hanya sampai Juli 2027).
- Link langsung ISO (3,7 GB): `https://cdimage.ubuntu.com/lubuntu/releases/26.04.1/release/lubuntu-26.04.1-desktop-amd64.iso` — torrent dan SHA256SUMS ada di folder rilis yang sama. Flashdisk minimal 8 GB.

## Bikin bootable di Ubuntu: Startup Disk Creator

- User sudah download ISO dan bertanya cara bikin bootable di Ubuntu, karena Rufus hanya ada di Windows (side chat "Choose OS for 4GB laptop server", 13:10 WIB). Jawaban: pakai **Startup Disk Creator** bawaan Ubuntu (`sudo apt install usb-creator-gtk` kalau belum ada). Caranya: colok flashdisk (backup dulu isinya), pilih ISO di "Source disc image", pilih flashdisk di "Disk to use", klik Make Startup Disk, masukkan password, tunggu sampai selesai tanpa dicabut. Jangan pakai `dd` kalau belum yakin — salah ketik nama disk bisa menghapus harddisk sendiri.

## Insiden /dev/sda: salah duga, lalu dikoreksi

- Percobaan pertama gagal (13:17 WIB). Bre awalnya salah menduga `/dev/sda` itu harddisk internal mini PC dan memperingatkan jangan lanjut — dugaan itu **salah**.
- Setelah user kirim screenshot Disks, koreksinya jelas: `/dev/sda` **memang flashdisk** 63 GB USB DISK 3.0 (58,6 GB di Startup Disk Creator — beda hitungan GB vs GiB, drive yang sama). Harddisk internal itu yang 256 GB di atasnya. Jadi target yang dipilih user sudah benar.
- **Penyebab gagal sebenarnya:** partisi flashdisk masih ter-mount di `/media/exedy/Chapter Us`, dan Startup Disk Creator tidak bisa menulis ISO ke disk yang partisinya masih dipakai. Perbaikannya: di Disks, klik tombol unmount sampai tertulis Not Mounted, lalu ulangi dari Startup Disk Creator.
- Isi flashdisk "Chapter Us" (~2 GB, 3,1% terpakai) akan terhapus semua saat ISO ditulis — backup dulu kalau ada file penting.
- **Jalan lain kalau masih gagal:** di Disks, pilih drive 63 GB, menu titik tiga → Restore Disk Image, pilih ISO Lubuntu, Start Restoring. Fungsinya sama, sering jalan saat Startup Disk Creator ngadat.
- Screenshot terakhir dikonfirmasi benar (ISO Lubuntu 26.04.1 LTS 3,7 GB + USB DISK 3.0 /dev/sda 58,6 GB) → tinggal klik Make Startup Disk, masukkan password, tunggu, eject aman, colok ke G480, tekan F12, pilih USB, lalu Try Lubuntu.
- Status akhir: belum ada konfirmasi user bahwa pembuatan USB selesai atau live session sudah jalan.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — server yang disiapkan di laptop ini nantinya menjalankan Hermes/9Router/n8n.

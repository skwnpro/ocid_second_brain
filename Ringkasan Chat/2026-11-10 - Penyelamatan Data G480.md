# Penyelamatan Data G480

Ringkasan 10 November 2026 — sesi malam di installer Ubuntu Server: menyelamatkan data partisi D laptop Lenovo G480 sebelum install, lalu mulai menyalin Foto Mila (main chat, 20:50–22:45 WIB; plus konfirmasi Rufus MBR di side chat "Choose OS for 4GB laptop server", 20:47 WIB).

## Rufus: mode tulis dan skema partisi

- **Mode tulis ISO (20:50 WIB):** Rufus menanyakan ISO Image mode vs DD Image mode karena ISO Ubuntu itu ISOHybrid. Dipilih **ISO Image mode (Recommended)** — menyalin file-file ISO ke flashdisk seperti copy biasa, flashdisk masih kebaca normal di Windows. DD Image mode hanya cadangan kalau USB gagal boot. Catatan: yang dipilih ini ISO **Ubuntu Server**, bukan Lubuntu — Server tidak ada mode "Try" live desktop, dia langsung masuk installer. Buat menyelamatkan data D dulu, tetap pakai USB Lubuntu dari tadi pagi; USB Server ini buat langkah install setelah D aman.
- **MBR (20:47 WIB):** screenshot Rufus menunjukkan Partition scheme MBR, Target system BIOS or UEFI — itu sudah benar buat G480. MBR itu skema lama buat BIOS legacy (maksimal disk 2TB, 4 partisi utama); G480 laptop 2012 dengan BIOS legacy dan HDD di bawah 2TB, jadi MBR yang cocok. GPT itu buat UEFI modern — kalau dipilih di Rufus, targetnya jadi UEFI saja dan G480 kemungkinan tidak bisa boot dari USB itu. Setting MBR ini hanya buat USB-nya, bukan buat harddisk G480 (partisi harddisk urusan installer Ubuntu Server, beda lagi).

## Jangan format seluruh disk: D harus diselamatkan dulu

- User masuk installer dan mau memindahkan beberapa file, tapi hanya bisa pakai Ubuntu Server via CLI (20:50 WIB). Rencana: boot USB Server, di installer pilih Help → Enter shell, mount partisi D (NTFS) read-only biar aman, mount media backup, copy pakai `rsync`/`cp`, verifikasi dulu sebelum install.
- **STOP di layar "Use an entire disk" (21:03 WIB):** pilihan itu akan memformat seluruh disk 298GB termasuk partisi D. User disuruh tekan Back, lalu Help → Enter shell, dan menjawab dulu: file D mau di-copy ke mana (flashdisk/harddisk eksternal lain, atau kirim lewat jaringan ke mini PC).
- **Pilih Custom, D masih utuh (21:05 WIB):** user memilih Custom lalu Done — benar, D masih kelihatan utuh di list. Isi harddisk (ST320LT020, 298GB): partition 1 NTFS 73GB = C (Windows lama, ini yang nanti diformat jadi `/`); **partition 2 NTFS 149GB = D (data, jangan diformat, jangan di-mount di installer)**; partition 3 vfat 30GB = kemungkinan partisi recovery (jangan sentuh); partition 4 NTFS 500MB = boot Windows (jangan sentuh); Lexar 14,5GB = USB installer (jangan sentuh).

## Mount D: salah nama disk, lalu ketemu

- Percobaan pertama mount gagal (21:07 WIB) — penyebabnya salah nama disk: Bre menebak `sda`, ternyata dari `lsblk` terbukti `sda` = USB installer (UBUNTU-SERV), **`sdc` = harddisk internal** — `sdc1` itu C, **`sdc2` itu D**, `sdc3` partisi DATA, `sdc4` System Reserved. Perintah yang benar: `mount -o ro /dev/sdc2 /mnt/d`, lalu `du -sh /mnt/d` dan `ls /mnt/d`.
- **D kebaca, datanya masih utuh (21:09 WIB):** isinya folder DATA MILA, FILM, dokumen kuliah IPB (KIP Kuliah, Essai, Praktikum), sama file-file Word. Tapi total isi D **90GB**, sementara USB installer cuma sisa 11,8GB — copy semuanya ke situ tidak muat.
- User memutuskan: punya flashdisk 64GB, mau buka data Mila dan pindahkan satu-satu ke situ (21:11 WIB). Perintah awal: `ls -la "/mnt/d/DATA MILA"`, `du -sh "/mnt/d/DATA MILA"`, colok flashdisk 64GB, `lsblk -f`, foto hasilnya.
- **Isi DATA MILA: 75GB** (21:14 WIB) — lebih besar dari flashdisk 64GB (aslinya cuma ~59GB yang kepake). Jadi tidak bisa copy semuanya; harus pilih-pilih subfolder, sesuai rencana user "satu-satu". Isinya 4 folder: Dokumen, Foto Mila (57GB), MUSIK, VIDEO. Perintah buat melihat ukuran tiap subfolder: `du -sh "/mnt/d/DATA MILA"/* | sort -rh`.

## Flashdisk 64GB diformat jadi BACKUP64

- Flashdisk 64GB itu ternyata masih berformat ISO Lubuntu (21:16 WIB) — makanya di Windows kebaca 2 partisi F dan G yang tidak bisa dibuka atau diformat, dan terasa "write protected". Format ISO itu dari sononya read-only, bukan berarti flashdisk rusak.
- Perbaikan dari shell (21:22 WIB): cek `blockdev --getro /dev/sdd` (harus 0), lalu `umount /mnt/usb64`, `wipefs -a /dev/sdd`, `printf 'label: dos\n, , c\n' | sfdisk /dev/sdd`, `mkfs.vfat -F 32 -n BACKUP64 /dev/sdd1`, `lsblk -f /dev/sdd`. Peringatan keras saat itu: target harus **`/dev/sdd`** (USB Lubuntu) — `sda` itu USB installer yang lagi jalan, `sdc` itu harddisk laptop; salah ketik bisa fatal.
- **Hasil akhir (21:29 WIB):** `sdd1` vfat FAT32 BACKUP64, kosong, siap diisi. Langkah berikut: `mkdir -p /mnt/backup`, `mount /dev/sdd1 /mnt/backup`, `df -h /mnt/backup`, `du -sh "/mnt/d/DATA MILA/Foto Mila"`.

## Copy Foto Mila: jalan, satu file error

- **Muat, tapi mepet (21:31 WIB):** Foto Mila 57GB, ruang kosong 59GB, sisa ~2GB — masuk asal jangan ditambah file lain. Perintah: `cp -a "/mnt/d/DATA MILA/Foto Mila" /mnt/backup/`. Diperkirakan lama (57GB, bisa 1,5–2 jam lewat USB 2.0); layar diam itu normal. Jangan cabut flashdisk, jangan matikan laptop, tunggu sampai prompt `root@ubuntu-server:/#` balik lagi. Verifikasi sesudahnya: `du -sh /mnt/backup/"Foto Mila"` harus 57G juga, lalu `sync` dan `umount /mnt/backup`.
- **Satu jam kemudian masih jalan (22:41 WIB):** user mengirim foto layar yang tidak berubah — copy masih jalan, bukan stuck; buktinya prompt belum balik. Satu-satunya error sejauh ini: `cp` tidak bisa membuat `/mnt/backup/Foto Mila/WhatsApp/IMG-20~5.JPG` karena **File exists** — tabrakan nama pendek (short-name) di sistem FAT32 pada foto-foto WhatsApp yang namanya mirip-mirip. `cp` me-skip file itu saja dan lanjut menyalin yang lain; file itu dibetulkan nanti setelah copy selesai.
- **Cara mengecek progres (22:41 WIB):** pindah ke konsol kedua dengan **Ctrl+Alt+F2**, login sebagai `root` (password kosong, langsung Enter), ketik `du -sh /mnt/backup/"Foto Mila"`. Balik ke layar copy dengan **Ctrl+Alt+F1**.
- **Berapa konsol (22:45 WIB):** biasanya **6** (F1–F6); F1 yang sedang menjalankan copy, F2–F6 kosong dan independen seperti 6 layar terminal. Ada juga sistem yang sampai 12 (F1–F12), tapi yang aktif umumnya 6 itu.
- Status akhir malam ini: **copy belum selesai, belum terverifikasi** — hasil `du` dan foto verifikasi masih ditunggu.

## RAM G480 dan perbandingan dengan VPS

- **RAM (21:46 WIB):** DDR3, SODIMM 204-pin, 1600MHz (PC3-12800). Sekarang 4GB, slot ada 2, maksimal 8GB resmi.
- **2x8GB bisa atau tidak (21:47 WIB):** resminya tidak bisa — manual Lenovo bilang maks **8GB** (2x4GB DDR3-1600). Tapi di lapangan CPU Ivy Bridge sebenarnya support 16GB dan ada laporan 2x8GB jalan di G480 — kemungkinan besar bisa, **tidak dijamin**: kalau BIOS rewel, bisa tidak kebaca. Buat Ubuntu Server 8GB sudah lebih dari cukup, 16GB overkill; kalau mau nambah nanti, 2x4GB saja yang aman.
- **G480 + 8GB vs VPS Instabox (21:49 WIB):** G480 menang di **nyala terus** (kelemahan terbesar VPS gratis: pause sendiri setelah 30 menit idle atau 8 jam jalan) dan **biaya** (cuma tambah listrik sekitar 25–40 ribu per bulan, estimasi). VPS menang di **tenaga mentah** (4 vCPU modern + SSD lawan i3/i5 Ivy Bridge 2012 + HDD putar), **keandalan + akses** (datacenter, IP publik, gampang diakses dari mana saja; G480 di belakang NAT rumah, harus lewat Tailscale, dan HDD 14 tahun itu bom waktu). Verdict buat kebutuhan user (bot Telegram, Hermes, n8n — kerjaan ringan yang butuh nyala terus): **G480 malah lebih cocok**, 8GB sudah lega, bahkan 4GB sekarang pun sebenarnya cukup. Bukan menggantikan VPS, tapi bagi tugas: G480 buat yang harus standby terus, VPS buat yang butuh tenaga atau diakses publik.

## Terkait

- [[2026-11-10 - Lubuntu untuk Laptop G480]] — persiapan USB Lubuntu dan insiden /dev/sda tadi pagi; laptop yang sama.
- [[2026-11-10 - Hermes dan Gemini]] — server yang disiapkan di laptop ini nantinya menjalankan Hermes/9Router/n8n.

# Penyelamatan Data G480

Ringkasan 11 November 2026 — lanjutan pemindahan folder VIDEO dari DATA MILA ke BACKUP64; flashdisk ternyata berganti huruf device dan copy gagal dengan I/O error (side chat "Choose OS for 4GB laptop server", 21:27–21:40 WIB).

## Yang terjadi

- **Huruf device berganti.** BACKUP64 kemarin terdeteksi sebagai `sdd1`, hari ini jadi `sde1` — huruf device di Linux tidak tetap, berubah tiap boot tergantung urutan colok dan port USB. User sempat me-mount `sdd1` yang ternyata perangkat lain (kemungkinan USB live yang read-only), sehingga `cp` gagal.
- **Copy VIDEO gagal: Input/output error.** Pesan `cp: cannot create directory '/mnt/backup/VIDEO': Input/output error` — ini sinyal dari hardware/filesystem, bukan "tidak muat" atau "tidak ada izin" (kalau tidak muat pesannya akan "No space left"). Nyambung dengan tanda kemarin: BACKUP64 di Windows tidak bisa dibuka tapi di Ubuntu bisa — filesystem-nya sudah dicurigai kotor atau flashdisk mulai bermasalah, apalagi habis ditulis 57GB sampai hampir penuh.
- **Mount ganda sempat terjadi.** Percobaan mount kedua gagal dengan "already mounted on /mnt/backup" — mount lama yang salah masih menempel, harus dilepas dulu (`sudo umount /mnt/backup`) sebelum mount yang benar.
- **Perbaikan yang disarankan:** lepas mount lama, mount ulang ke `/dev/sde1`, verifikasi dengan `ls` (harus muncul Foto Mila) dan `df` (sisa sekitar 2GB). Supaya tidak ketukar lagi, mount pakai label: `sudo mount LABEL=BACKUP64 /mnt/backup`.
- **Sebelum copy, cek ukuran dulu.** BACKUP64 tinggal sekitar 2GB kosong habis Foto Mila; perintah yang disarankan: `du -sh "/mnt/d/DATA MILA/VIDEO"` dan `df -h /mnt/backup` — kalau VIDEO lebih kecil dari sisa ruang, lanjut `cp -a "/mnt/d/DATA MILA/VIDEO" /mnt/backup/`, lalu verifikasi `du -sh /mnt/backup/VIDEO` dan `sync`.
- **Langkah diagnosis kalau masih error:** `lsblk -f` dan `dmesg | tail -n 50`, cari baris `I/O error`, `reset`, `FAT-fs`, atau `device not accepting address`. Saran lain: coba port USB lain langsung ke laptop (tanpa hub), dan tes `mkdir /mnt/backup/test` — kalau itu saja I/O error, jangan lanjut copy VIDEO.

## Status akhir — belum terverifikasi

- User terakhir bertanya "udah aman bre ini langsung cp atau gimana" (21:40 WIB) dan dijawab dengan instruksi cek ukuran + `cp` di atas. **Belum ada konfirmasi apakah copy VIDEO sudah jalan atau berhasil.** Foto Mila yang sudah di-copy sebelumnya juga disarankan dicek ulang ukurannya, jangan dianggap aman dulu — apalagi kalau penyebabnya flashdisk yang mulai rusak, bukan sekadar filesystem kotor.
- Satu-satunya hal yang pasti berubah dari kemarin: huruf device BACKUP64 sekarang `sde1`, bukan `sdd1` — selalu cek `lsblk -f` / label sebelum mount.

## Terkait

- [[2026-11-10 - Penyelamatan Data G480]] — copy Foto Mila 57GB kemarin, satu file ter-skip karena tabrakan nama pendek FAT32, dan BACKUP64 yang tidak bisa dibuka di Windows.

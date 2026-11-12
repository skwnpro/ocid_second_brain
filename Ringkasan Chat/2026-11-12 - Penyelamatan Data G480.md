# Penyelamatan Data G480

Ringkasan 12 November 2026 — clean install Ubuntu Server di Lenovo G480 selesai, lalu user memutuskan remote server itu dari PC via Tailscale (side chat "Choose OS for 4GB laptop server", 05:34–15:36 WIB).

## Clean install: sempat dikira disk hilang, ternyata cuma mount nyangkut

- **Installer bilang "no disks".** Setelah error lalu disuruh ulang, installer tidak melihat harddisk internal sama sekali. User sempat khawatir harddisknya mati.
- **Penyebab sebenarnya: partisi E masih ke-mount.** Screenshot boot log membuktikan harddisk sehat: `ST320LT020-9YG142` terdeteksi, `sdc` dengan partisi C, D, DATA, System Reserved lengkap. Yang bikin installer tidak mau probing adalah mount `/mnt/e` yang masih nempel dari shell sebelumnya. Perbaikannya: `umount /mnt/e`, `umount /mnt/d`, `umount /mnt/backup`, cek `lsblk -f` sampai bersih, baru tekan Refresh di installer.
- **Installer sempat crash.** Layar "Sorry, the installer has restarted because of an error" dengan `ProblemType: Bug`, project `subiquity` (nama installer Ubuntu Server). Kemungkinan besar crash saat probing disk yang masih ke-mount. Saran: jangan pilih Send to Canonical; ulangi tanpa buka shell/mount dulu. Kalau crash lagi di tempat yang sama, ganti ke Ubuntu Server 24.04 LTS (lebih stabil di hardware 2012) — catatan ini belum terverifikasi karena install akhirnya berhasil di percobaan berikutnya.
- **Layout clean install terkonfirmasi.** Target benar: `ST320LT020 298GB`, `/boot/efi` 1GB, `/boot` 2GB, root `/` 100GB ext4, sisa 195GB free di ubuntu-vg (normal — Ubuntu sengaja pakai 100GB dulu). Lexar USB 14.5GB itu installernya, jangan disentuh. User menekan Done → Continue (format), lalu isi profile.
- **Isian profile yang penting:** yang dipakai buat SSH cuma username + password. Nama (misal `Skwn`), nama server (`g480-server`), username (`skwn`, huruf kecil, tanpa spasi). Peringatan yang sudah disampaikan: **jangan pakai huruf f atau c di username/password**, karena keyboard PC user tombol F dan C-nya rusak. Password dicatat user sendiri di file pass-nya, jangan dikirim di chat.
- **Satu temuan dari log:** `FAT-fs (sde1): Volume was not properly unmounted` — itu BACKUP64, dicabut tanpa eject bersih. Datanya kemungkinan masih aman, tapi **harus dicek pakai `fsck` dulu sebelum dibuka di Windows** — belum dilakukan.

## Login pertama: keyboard F7 nyangkut

- Setelah install dan reboot, layar login kebanjiran `[[18~` berulang-ulang — itu kode tombol **F7** yang ketahan/nyangkut dan mengirim terus. Bukan Ubuntu rusak.
- Solusi: tekan-tekan F7 dan semua tombol F lain untuk cek yang nyangkut; matikan paksa (tahan power 5 detik), coba lagi; atau colok keyboard USB dan login pakai itu. Kalau keyboard internal memang rusak, solusinya cabut kabel fleksibel keyboard internalnya atau disable di Ubuntu — buat server tidak masalah, nanti dikelola lewat SSH.
- **Cara shutdown yang benar dari terminal:** `sudo shutdown now` (atau `sudo reboot` buat restart). Tekan Ctrl+C dulu buat membersihkan input yang kebanjiran `[[18~`. Jangan matikan pakai tahan tombol power kalau masih bisa pakai command — filesystem bisa kotor lagi seperti BACKUP64 kemarin.

## Keputusan baru: remote via Tailscale, tanpa port forwarding

User (15:36 WIB): mau setup server G480 supaya bisa di-remote dari PC, dan 9Router + Hermes nanti juga punya dashboard — **pakai Tailscale saja biar ringkas, tanpa port forwarding**. Rencana yang diberikan:

1. **Di G480, login lokal sekali dulu** (pakai keyboard USB kalau F7 masih spam): install openssh-server, `systemctl enable --now ssh`, install Tailscale (`curl -fsSL https://tailscale.com/install.sh | sh`, `sudo tailscale up`), buka link auth di PC, login akun Tailscale yang sama, approve device `g480-server`, catat IP `100.x.x.x`-nya.
2. **Dari PC, remote pakai SSH:** `ssh skwn@g480-server` atau `ssh skwn@100.x.x.x`, passwordnya password server yang dibuat pas install.
3. **Dashboard 9Router + Hermes lewat SSH tunnel:** keduanya jalan di `localhost` G480 (9Router port 20128, Hermes dashboard 9119). Dari PC: `ssh -L 20128:localhost:20128 -L 9119:localhost:9119 skwn@g480-server`, biarkan terminal terbuka, lalu buka `http://localhost:20128` dan `http://localhost:9119` di browser PC. Servicenya tetap ngumpet di localhost server, cuma user yang bisa buka lewat tunnel.
4. **Nanti install 9Router + Hermes di G480** sama seperti di VPS: `npm install -g 9router`, Hermes diarahkan ke `http://localhost:20128/v1`, model `gemini/gemini-3.6-flash`. Catatan: kalau pakai key Gemini yang sama seperti bot finance, kuota 20/hari itu berbagi — Hermes di G480 dan bot finance rebutan kuota yang sama.

## Status — belum terverifikasi

- **Belum ada konfirmasi apakah user sudah berhasil login ke G480** (pakai keyboard USB) dan menjalankan langkah Tailscale di atas. Asisten masih menunggu kabar IP Tailscale-nya.
- **BACKUP64 belum dicek `fsck`** — jangan dipercaya di Windows sebelum dicek.
- Copy VIDEO dari DATA MILA ke BACKUP64 (tertunda sejak 11 Nov) belum ada konfirmasi berhasil — kemungkinan sudah tidak relevan lagi karena semua file sudah dipindah dan user memilih clean install, tapi belum dikonfirmasi eksplisit.

## Terkait

- [[2026-11-11 - Penyelamatan Data G480]] — BACKUP64 berganti device sdd1→sde1, copy VIDEO gagal I/O error.
- [[2026-11-10 - Penyelamatan Data G480]] — copy Foto Mila 57GB, satu file ter-skip karena tabrakan nama pendek FAT32.
- [[2026-11-10 - Hermes Balik ke Gemini]] — setup 9Router/Hermes di VPS yang akan direplikasi ke G480.

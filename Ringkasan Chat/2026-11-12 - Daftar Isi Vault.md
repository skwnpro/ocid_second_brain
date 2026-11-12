# Daftar Isi Vault

Ringkasan 12 November 2026 — user merevisi file `Daftar Isi Vault.md` (indeks seluruh vault) sampai bentuk finalnya: nested dropdown per topik plus tabel tools GitHub terpisah (main chat, 14:32–15:21 WIB).

## Yang diminta user

- **Bukan per project.** Indeks awal dikelompokkan per folder (Threads, Ringkasan Chat, dst.) — user menolak: maunya per **isi/subjek**, kayak daftar isi di file paling luar (root vault, semacam readme).
- **Bisa di-dropdown.** Pas file dibuka, yang muncul hanya topik utama; klik satu topik → sub-topiknya muncul; klik sub-topik → isinya muncul, begitu seterusnya.
- **Tanpa deskripsi, tanpa link.** Semua penjelasan di bawah heading dihapus, semua link `[[#...]]` dihapus. File murni daftar isi.
- **GitHub Tools jadi topik utama sendiri**, isinya dibaca dari catatan harian atau Threads dump. Sub-topiknya diserahkan ke asisten: Video Generator, Skills Audio, AI Coding, Tools PC dan Utilitas, Otomasi dan Workflow.
- **Rekomendasi GitHub dan tools PC dibuatkan list/tabel sendiri** di bawah, kolomnya: Topic | Nama Tools | Fungsi (singkat) | Link GitHub. Alasannya: buat mengamankan ingatan user, tools itu penting dan bakal sering dibuka ulang.

## Hasil akhirnya

- **Format: nested callout Obsidian.** Tiap topik utama satu callout `> [!note]- <Topik>` (collapsed); sub-topik nested callout `> > [!note]- <Sub>`; daunnya bullet biasa. Tidak ada wrapper callout luar, tidak ada heading, tidak ada link, tidak ada deskripsi.
- **11 topik utama:** AI dan Robotik; PC, Server, dan Infrastruktur; Otomasi dan Bot; Keuangan, Saham, dan Trading; Karir dan Kerja; Konten dan Media; Properti dan Rumah; Penyelamatan Data G480; Game Nusantara; GitHub Tools; Catatan Pribadi.
- **Koreksi user yang dimasukkan:** KIPK/IPB, arsip foto, dan Foto Mila itu konten **Penyelamatan Data G480** (data yang diselamatkan dari partisi D), bukan topik Edukasi/Keluarga — dua topik itu dihapus. Catatan Pribadi (Sholat Jumat + Braind Dump) dibiarkan sebagai penampung.
- **Tabel Tools GitHub:** 20 baris, semua link diverifikasi dari file vault atau hasil search. Dua catatan: WinOptimizer tidak ada linknya (sumber aslinya memang tidak memberi link); Ninite linknya ke ninite.com (situs resmi, bukan GitHub). Lima tools sengaja tidak dimasukkan karena link repo terverifikasinya belum ketemu: Nango, youtube-viral-clipper, Agentic Inbox, Fincept Terminal, Flowsint.
- **Draf awal yang dibuang:** versi pertama file ini (dibuat subagent pagi harinya, 6 topik, 64 file, format daftar isi + heading dengan link anchor) diganti total mengikuti revisi user di atas.

## Status — belum di-push

- User menyetujui update tiap hari, tapi versi ini **ditahan push-nya** ("nanti aja di pushnya"). File tetap lokal di `~/workspace/second_brain/Daftar Isi Vault.md`.
- Job malam merawat file ini tiap malam (topik baru dari ringkasan + baris tabel baru dari tools yang ada linknya), tapi tidak ikut ke-push sampai user memerintahkan di chat.

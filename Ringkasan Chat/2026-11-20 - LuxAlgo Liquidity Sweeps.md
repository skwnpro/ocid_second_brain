# LuxAlgo Liquidity Sweeps

Ringkasan 20 November 2026 — di WhatsApp (~09:23 WIB): user mengirim script Pine Script v5 lengkap (LuxAlgo Liquidity Sweeps, lisensi CC BY-NC-SA 4.0), minta dijadikan satu file, lalu minta dibaca dan dijelaskan. Bre menyimpan script itu apa adanya dan menjelaskan cara kerjanya.

## Yang dikirim user

- Script lengkap **Liquidity Sweeps [LuxAlgo]** untuk TradingView, Pine Script v5.
- Disimpan verbatim sebagai `~/workspace/your_files/liquidity-sweeps-luxalgo.pine` (belum dites compile di TradingView — kalau pas di-paste ada error, user diminta kabarin).

## Cara kerja indikatornya

- **Nyari "kolam" likuiditas dulu:** deteksi swing high/low (puncak/lembah 5 bar kiri-kanan, bisa diubah di setting "Swings"). Di atas swing high dan di bawah swing low itu tempat stop loss banyak trader ngumpul.
- **Nandain sweep wick:** kalau harga nembus ke atas swing high tapi tutup candle-nya balik ke bawah (atau sebaliknya di swing low), indikator menggambar kotak + garis putus-putus di level itu. Artinya: ada yang menyapu likuiditas di situ, harga ketolak balik.
- **Nandain breakout + retest:** kalau harga nembus swing high dengan close di atasnya, terus balik turun ngetes level itu dan ketahan (close balik ke atas), indikator menggambar kotak hijau. Artinya: level itu sudah jebol dan sekarang jadi support.
- Ada 3 mode di setting: cuma wick, cuma breakout + retest, atau dua-duanya. Kotaknya bisa memanjang sampai 300 bar (atau sampai harga close menembus kotak itu, berarti areanya sudah tidak valid).

## Yang penting: ini bukan sinyal buy/sell

- Indikator ini cuma menandai zona — harga habis menyapu likuiditas belum tentu balik arah. Tetap butuh konfirmasi lain + aturan trade user sendiri (lot size, kill switch, jangan nambah posisi rugi).
- Cara pakai: buka TradingView → Pine Editor → paste isi file → Add to chart.
- Nyambung ke [[2026-11-17 - Trading Bot]]: sama seperti post martingale kemarin, ini bahan belajar membaca struktur chart, bukan sistem siap pakai — dan tetap tunduk ke disiplin trade-rules.md (tidak ada martingale, lot dihitung dari risk%, kill switch harian).

## Belum terverifikasi

- Script belum dites compile di TradingView; kemungkinan error pas paste belum diketahui.

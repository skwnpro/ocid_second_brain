# Trading Bot

Ringkasan 17 November 2026 — di side chat "Trading bot, EA, and quant trading" (06:58 WIB): user mengirim satu post Threads dan minta dijelaskan. Bre membedah klaimnya dan membandingkannya dengan disiplin trade-rules.md user sendiri.

## Post yang dibahas

- Post dari **@deactivated911__** di Threads (16 Nov 2026, 26 likes, 0 komentar, follower-nya cuma 5 orang, tidak verified), menjawab pertanyaan orang lain: *"JUJUR AJA: Apa 'cheat code' soal duit yang pernah kalian temuin, yang rasanya kayak curangin sistem tapi 100% legal?"*
- Strategi yang diklaim author: trading forex di akun **Exness raw, leverage 1:2000**, pakai **EA martingale lot tetap 0.01**. Kalau EA itu sudah masuk 4 entry terus harga memantul, dia masuk **manual lot 3**. Terus dia pakai **EA kedua yang tugasnya cuma take profit di $200**. Kalau harga beneran memantul dan entry manualnya profit, dia membiarkannya sampai kena TP $200 itu. Dia memantau **RSI** untuk memastikan itu beneran memantul atau malah lanjut terus. Klaimnya: **profit $200–600 per hari × 20 hari kerja**. Ditutup dengan "persetan orang gak percaya."

## Bedah Bre: ini bukan cheat code

1. **Ini martingale + rescue manual, bukan sistem.** Martingale itu strategi yang menambah posisi pas lagi loss, berharap harga balik. Dia bilang lot-nya fixed 0.01, tapi terus masuk manual lot 3 — **lot 3 itu 300x lipat dari 0.01**. Jadi pas EA-nya sudah loss 4 entry, dia malah memasang taruhan super besar dengan harapan harga memantul. Analoginya: kalah 4x berturut-turut pakai taruhan kecil, terus all-in di ronde kelima dengan harapan pasti menang. Kalau beneran memantul, menang besar. Kalau harga malah lanjut terus, habis di situ juga.
2. **RSI tidak bisa memastikan memantul atau lanjut.** RSI itu indikator yang mengikuti harga yang sudah terjadi, bukan peramal. RSI oversold bukan berarti harga pasti naik, bisa saja dia tetap turun terus. Jadi "pantau RSI untuk mempertegas" itu kedengaran teknis, tapi tidak menghilangkan risiko utamanya.
3. **Klaim profitnya tidak masuk akal kalau dihitung.** $200–600 per hari × 20 hari = **$4.000–12.000 per bulan**. Dia tidak menyebut modalnya berapa. Kalau modalnya $1.000, itu 400–1.200% per bulan. Kalau modalnya $10.000, itu masih 40–120% per bulan, terus-terusan. Strategi yang beneran bisa begitu tidak bakal dishare gratis di Threads sambil menantang orang tidak percaya.
4. **Tidak ada bukti apa-apa.** Tidak ada Myfxbook, tidak ada history trade, tidak ada drawdown, tidak ada modal awal. Cuma klaim di caption.
5. **Leverage 1:2000 membuat ini makin bahaya.** Gerakan harga kecil saja bisa menghabiskan margin, apalagi pas sedang memegang lot 3. Exness raw account memang spread-nya kecil, tapi itu tidak menolong kalau arahnya salah dan lot-nya besar.

## Hubungannya dengan roadmap user: kebalikan total

- Ini persis kebalikan dari yang sedang user bangun di trade-rules.md: di roadmap user, lot dihitung dari risk%, ada kill switch max loss harian, tidak ada martingale/grid, tidak ada menambah posisi pas loss, dan aturan dibekukan selama backtest/demo.
- Post ini malah: martingale, intervensi manual lot besar, tidak ada SL/kill switch yang disebut, dan TP-nya $200 tetap tanpa menyebut modal dan drawdown. Kalau pola ini dimasukkan ke EA user, backtest-nya mungkin kelihatan profit halus berbulan-bulan, terus satu tren panjang langsung habis. Itu yang sudah Bre warning dari awal.
- Post yang sama juga tersimpan di Threads hari ini (Threads/doomscroll dump/2026-11-17.md) dengan catatan Explore yang sama: jangan ditradingkan; kalau penasaran dengan logikanya, cukup backtest di atas kertas, jangan pernah dengan uang asli.

## Kesimpulan

- Bukan strategi yang bisa dicopy mentah. Ini contoh bagus buat belajar cara membaca klaim trading di sosmed: cari modalnya, cari drawdown-nya, cari bukti live-nya. Kalau tiga-tiganya tidak ada, anggap saja cerita, bukan sistem.
- **Belum terverifikasi:** klaim profit $200–600/hari (tidak ada bukti); apakah author benar-benar trading seperti itu.
- **Tidak ada perubahan pada roadmap** — Fase 0 masih menunggu user memutuskan pair, timeframe, dan entry/exit manual.

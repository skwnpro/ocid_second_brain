# Flow Diagram — Finance Assistant

Alur data dan flowchart sistem pencatat keuangan: recorder bot,
advisor bot, database, dan lifecycle VPS. Diagram memakai Mermaid
(ter-render langsung di Obsidian).

Lihat juga: [[install-instabox]] (instalasi di VPS),
[[postgresql]] (perintah dan skema database).

## Arsitektur sistem

Dua bot berbagi satu database. Recorder menulis, advisor hanya
membaca. Gemini dipakai keduanya, tapi dengan peran berbeda: recorder
memakainya untuk parsing transaksi, advisor hanya untuk memahami
pertanyaan — angka selalu dari SQL, tidak pernah dari AI.

```mermaid
flowchart LR
    U[User\nTelegram] -->|pesan teks / foto| R[Recorder bot\nsrc/]
    U -->|pertanyaan| A[Advisor bot\nadvisor/]
    R -->|parse transaksi| G[Gemini API]
    A -->|klasifikasi intent| G
    R -->|INSERT / UPDATE / DELETE| DB[(PostgreSQL\nfinance_assistant)]
    R -->|tulis audit| DB
    A -->|SELECT saja| DB
    G -.->|tidak pernah menghitung angka| DB
```

Aturan arsitektur yang tidak boleh dilanggar:

- Satu token = satu instance polling. Dua instance dengan token sama
  → 409 Conflict.
- Advisor tidak punya jalur tulis: user database-nya SELECT-only
  dan koneksi disetel read-only.
- Semua tanggal transaksi = WIB, dihitung di Python
  (`today_wib()`), bukan di database.

## Flow recorder: pesan teks

```mermaid
flowchart TD
    S[Mulai: pesan teks masuk] --> Auth{ID Telegram\nterdaftar?}
    Auth -- Tidak --> Tolak[Balas ID user\n+ minta didaftarkan]
    Auth -- Ya --> Cmd{Perintah?}
    Cmd -- /start /saldo /riwayat /utang /audit /dompet --> Jalankan[Jalankan perintah\nbaca database]
    Cmd -- /batal --> Undo[Undo batch terakhir\nDELETE + tulis audit undo]
    Cmd -- teks biasa --> Gemini[Gemini parse\nmax 10 transaksi]
    Gemini --> Valid{Hasil valid?}
    Valid -- Tidak --> Minta[Balas error + contoh\npenulisan yang benar]
    Valid -- Ya --> Loop[Untuk tiap item:\nresolve dompet & tanggal]
    Loop --> Ambig{Dompet ambigu?}
    Ambig -- Ya --> Tanya[Balas: sebutkan nama\ndompet lengkap]
    Ambig -- Tidak --> Eksekusi[Eksekusi dalam\n1 transaksi DB:\nINSERT tx + audit]
    Eksekusi --> Balas[Balas ringkasan batch]
    Jalankan --> Balas
    Undo --> Balas
```

Catatan penting:

- Ambiguitas dompet **tidak** disimpan sebagai state. Bot hanya
  membalas meminta nama lengkap, user mengirim ulang pesan dengan
  nama dompet lengkap. Cocokkan: persis → partial unik → ambigu
  (tanya) → tidak dikenal (error).
- `/batal` meng-undo **seluruh batch terakhir**, bukan satu item.
- State `_last_action` (untuk undo) hidup di memori: survive pause
  mesin, hilang kalau service restart.
- Arah hutang ditentukan subjek kalimat: `[NAMA] bayar` = uang
  masuk (piutang dibayar), `bayar hutang ke [NAMA]` = uang keluar.
  Salah arah = error pintar yang menampilkan kondisi hutang aktual
  + contoh penulisan ulang.

## Flow recorder: OCR struk

```mermaid
flowchart TD
    P[Foto struk masuk] --> OCR[Gemini Vision\nbaca per baris]
    OCR --> Review[Tampilkan review:\nmerchant, total, tanggal,\nitem per baris]
    Review --> Koreksi{User kirim\nkoreksi?}
    Koreksi -- "no 2 = groceries" --> Cat[Ubah kategori item 2]
    Koreksi -- "no 1 = 10000" --> Harga[Ubah harga item 1]
    Koreksi -- kata konfirmasi --> Dompet[Minta dompet]
    Koreksi -- input lain --> Review
    Cat --> Review
    Harga --> Review
    Dompet --> Simpan[Catat semua item\n+ crosscheck total]
    Simpan --> Cek{Selisih item vs\ntotal > Rp1.000?}
    Cek -- Ya --> Warn[Balas + peringatan\nselisih]
    Cek -- Tidak --> Selesai[Balas ringkasan]
```

Aturan OCR yang lahir dari bug nyata:

- Prompt membaca **total tercetak per baris**, tidak pernah
  menghitung `qty × harga`. (Kasus nyata: `2 ... 10000` pernah
  dihitung jadi 20000.)
- Pending OCR disimpan di disk — survive restart service.
- Input yang tidak dikenali saat review **tidak** menghapus
  pending; bot menampilkan ulang review.
- Crosscheck: jumlah item vs total struk, peringatan kalau selisih
  > Rp1.000.

## Flow advisor: tanya-jawab

```mermaid
flowchart TD
    Q[Pertanyaan masuk] --> Auth{ID terdaftar?}
    Auth -- Tidak --> Tolak[Balas ID user]
    Auth -- Ya --> Klas[Gemini klasifikasi\nintent + parameter]
    Klas --> OK{Hasil valid?}
    OK -- Tidak --> Bantuan[Balas contoh\npertanyaan /start]
    OK -- Ya --> Unknown{Intent unknown?}
    Unknown -- Ya --> Bantuan
    Unknown -- Tidak --> Per[Resolve periode WIB\n+ dompet + kategori]
    Per --> Ambig{Dompet ambigu /\nkategori tak dikenal?}
    Ambig -- Ya --> Klar[Balas klarifikasi]
    Ambig -- Tidak --> SQL[Jalankan SQL\n deterministik]
    SQL --> Templat[Template jawaban\nangka dari SQL]
    Templat --> Jawab[Balas ke user]
```

Intent yang didukung V1:

| Intent | Query | Filter opsional |
|---|---|---|
| `expense_total` | SUM cashflow | dompet, kategori |
| `expense_by_category` | SUM per kategori, GROUP BY | dompet |
| `income_vs_expense` | SUM income vs SUM cashflow | — |
| `top_expenses` | ORDER BY amount DESC LIMIT | dompet, kategori |
| `balances` | wallets + savings + total | — |
| `debts` | debt WHERE status <> 'paid' | — |
| `monthly_summary` | gabungan 4 query di atas | — |

Periode yang dipahami: hari ini, minggu ini, bulan ini, bulan lalu,
bulan bernama (`oktober` = kemunculan terakhir bulan itu di masa
lalu), tahun ini, selama ini. Perintah cepat: `/ringkasan`,
`/saldo`, `/utang` (melewati klasifikasi).

## Relasi antar tabel

```mermaid
erDiagram
    wallets ||--o{ cashflow : "wallets_id"
    wallets ||--o{ income : "destination_wallets"
    wallets ||--o{ moneyflow : "source/destination wallet"
    wallets ||--o{ adjustment : "wallets_id"
    savings ||--o{ moneyflow : "source/destination saving"
    categories ||--o{ cashflow : "category_name"
    wallets {
        int id
        string wallets_name
        numeric current_balance
    }
    savings {
        int id
        string saving_name
        numeric current_balance
    }
    income {
        int id
        date date
        string source
        numeric amount
    }
    cashflow {
        int id
        date date
        string category
        string item
        numeric amount
    }
    moneyflow {
        int id
        date date
        string source_type
        int source_id
        string destination_type
        int destination_id
        numeric amount
    }
    debt {
        int id
        string person_name
        string type
        numeric original_amount
        numeric remaining_amount
        date due_date
        string status
    }
    adjustment {
        int id
        date date
        numeric amount
        string adjustment_type
    }
    audit_log {
        int id
        bigint user_id
        string action
        string kind
        int row_id
        jsonb detail
    }
```

Catatan relasi:

- `cashflow.wallets_id` NULL = pengeluaran dari tabungan (skema
  tidak punya kolom sumber tabungan di `cashflow`).
- `moneyflow` memakai pasangan `source_type`/`source_id` dan
  `destination_type`/`destination_id` karena sumber/tujuan bisa
  dompet **atau** tabungan.
- `debt` tidak terhubung ke dompet: hutang baru dan pembayaran
  bisa dicatat tanpa dompet; saldo dompet hanya bergerak kalau
  dompet disebut.
- `audit_log` tidak punya foreign key — sengaja, supaya jejak
  tetap ada meskipun baris transaksi dihapus undo.

## Lifecycle VPS dan systemd

```mermaid
flowchart TD
    Boot[Mesin start / reboot] --> Systemd[systemd jalan]
    Systemd --> Enable{Service enable?}
    Enable -- Ya --> Start[Start cashflow-bot\n+ advisor-bot]
    Enable -- Tidak --> Diam[Tidak ada bot jalan]
    Start --> Poll[Polling Telegram\n+ PostgreSQL up]
    Poll --> Pause{Mesin pause?\n30 menit idle / 8 jam}
    Pause -- Ya --> Beku[Proses dibekukan\nmemori dipertahankan]
    Beku --> Resume[Resume manual:\nSSH / dashboard / port]
    Resume --> Poll
    Pause -- Tidak --> Poll
    Poll --> Pesan[Pesan masuk\nsaat pause?]
    Pesan -- Ya --> Antre[Telegram antre\nkirim saat resume]
    Antre --> Poll
```

Perintah yang menggerakkan diagram ini:

```bash
sudo systemctl enable cashflow-bot.service advisor-bot.service
sudo systemctl start cashflow-bot.service advisor-bot.service
sudo systemctl status cashflow-bot.service advisor-bot.service
sudo systemctl restart cashflow-bot.service
sudo journalctl -u cashflow-bot.service -n 30
```

Update kode: `git pull` lalu restart service yang berubah.
Pause/resume tidak butuh start ulang service — proses dilanjutkan
dari titik beku.

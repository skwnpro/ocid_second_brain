# finance_assistant

> Dokumentasi Second Brain — Project Documentation, Technical Documentation, User Guide,
> Development Journal, Learning Notes, Knowledge Base, Maintenance Reference, Historical Record.
> Terakhir diperbarui: 2026-10-19. Status: **V1 — fondasi stabil** (commit `62694fe`).

---

## Ringkasan

**finance_assistant** adalah asisten keuangan pribadi self-hosted: bot Telegram yang mencatat
transaksi keuangan lewat bahasa Indonesia natural (contoh: `beli bensin 50k bni ocid`,
`gaji 5jt masuk bca`, kirim foto struk). Pesan diurai oleh AI (Gemini) menjadi data terstruktur,
lalu disimpan ke PostgreSQL dengan saldo dompet yang terupdate otomatis.

- **Apa project ini:** Bot Telegram pencatat keuangan (recorder) + rencana bot advisor dan dashboard.
- **Tujuan:** Mencatat pengeluaran, pemasukan, transfer antar dompet, hutang/piutang, dan
  koreksi saldo semudah chatting — tanpa buka aplikasi/Sheets.
- **Masalah yang diselesaikan:** Pencatatan manual itu ribet dan gampang lupa; struk belanja
  menumpuk tidak tercatat; saldo antar dompet tidak terpantau.
- **Siapa penggunanya:** Pribadi (Skwn), single-user, akses dibatasi via `ALLOWED_USER_IDS`.
- **Status:** 🟢 V1 live — recorder bot jalan 24/7 sebagai systemd service di Raspberry Pi,
  backup harian otomatis, audit log aktif.

---

## Metadata

| Field | Value |
|---|---|
| Nama Project | finance_assistant |
| Tanggal Dibuat | 2026-10-15 (goal) / 2026-10-16 (kode pertama) |
| Versi | V1 — acuan terakhir commit `62694fe` (belum ada versioning semver) |
| Status | Aktif dikembangkan |
| Owner | Skwn (`skwnpro`) |
| Repository | `github.com/skwnpro/finance_assistant` (private) |
| Deployment | Raspberry Pi 4, Ubuntu 24.04.5, Python 3.12 venv, systemd service `cashflow-bot` |
| Teknologi Utama | Python, PostgreSQL 16, `python-telegram-bot` v20+, Gemini API (`google-genai`), Gemini Vision (OCR) |

---

## Latar Belakang

Kebutuhan personal: punya banyak dompet/rekening (bank, e-wallet, cash) dan ingin tahu ke mana
uang pergi tanpa effort pencatatan manual. Solusi awal yang direncanakan: dua bot Telegram —

1. **Recorder bot** — mencatat transaksi (pengeluaran, pemasukan, transfer, hutang). ✅ Live
2. **Advisor bot** — menjawab pertanyaan analitik (total pengeluaran, tren kategori, dsb). ⏳ Belum

Ditambah milestone: OCR struk (difasilitasi Gemini Vision ✅), audit log ✅, backup otomatis ✅,
dan dashboard Streamlit (⏳ belum).

---

## Tujuan

### Tujuan Utama
- Mencatat transaksi keuangan via chat bahasa Indonesia natural dengan akurasi tinggi.
- Menyimpan semua data terstruktur di PostgreSQL (9 tabel).
- Saldo tiap dompet selalu sinkron otomatis dengan transaksi.
- Mendukung transaksi susulan (backdate) dan foto struk (OCR).
- Setiap mutasi data terlacak (audit log) — tidak ada perubahan diam-diam.

### Tujuan Sekunder
- Cek saldo/riwayat/hutang/audit langsung dari Telegram.
- Undo pencatatan terakhir (`/batal`).
- Arsip foto struk.
- Backup harian otomatis dengan retensi.

### Non-Goals
- Multi-user publik (single-user by design, auth via Telegram user ID).
- Integrasi otomatis mutasi bank / e-wallet (Open Banking).
- Aplikasi mobile native.
- Microservice / arsitektur terdistribusi (ditolak eksplisit).

---

## Fitur

### 1. Pencatatan pengeluaran (`expense`)
- **Tujuan:** Catat belanja harian.
- **Cara kerja:** Pesan → Gemini ekstrak (barang, nominal, kategori, dompet, tanggal) → validasi
  Python → `INSERT` ke `cashflow` → saldo dompet berkurang → baris audit `create`.
- **Input:** `beli nasi uduk 10k BNI`, `10/10 beli skincare 70k bni ocid`.
- **Output:** `Tercatat. PENGELUARAN | nasi uduk | Rp10.000 | eating_out | Tanggal: ...`
- **Dependensi:** Gemini API, tabel `cashflow` + `wallets` + `audit_log`.

### 2. Pencatatan pemasukan (`income`)
- **Input:** `gaji 5jt masuk bca`, `gaji 20jt masuk bni ocid`.
- **Catatan:** Kalau dompet tidak disebut, default ke wallet `cash`.

### 3. Transfer antar dompet (`moneyflow`)
- **Input:** `tf dari bni ocid ke jago ocid 2jt`, `tarik tunai mandiri 100k` (→ otomatis ke `cash`),
  `topup gopay 100rb dari bca`.
- **Catatan penting:** Transfer **ke orang** (mis. `tf buat ayah 500k`) BUKAN moneyflow —
  dicatat sebagai `expense` kategori `gift`.

### 4. Hutang/piutang (`debt_new`, `debt_payment`)
- **Input baru:** `saya ngutang ke mila 500ribu` (hutang), `wahid ngutang 500k dibalikin bulan depan`
  (piutang + jatuh tempo).
- **Input bayar:** `beto bayar hutang 300k` (dia bayar ke saya), `bayar hutang ke mila 200k` (saya bayar).
- **Cara kerja:** Arah ditentukan dari subjek kalimat (aturan eksplisit di prompt). Pembayaran
  dialokasikan FIFO ke hutang terbuka (tertua dulu), status `unpaid → partial → paid`.
  Nama dinormalisasi (`Mas wahid` → `wahid`). Jatuh tempo deterministik
  (`bulan depan`/`minggu depan`/`tanggal 15`).
- **Catatan:** Dompet BOLEH tidak disebut (saldo tidak bergerak); kalau disebut tapi tidak
  cocok → error (dugaan typo), tidak ditebak diam-diam.

### 5. Penyesuaian saldo (`adjustment`)
- **Input:** `saldo awal bni ocid 200k`, `koreksi saldo cash jadi 250k`.
- **Cara kerja:** Saldo di-SET absolut; selisih dicatat di `adjustment` (`initial`/`correction`).
- **Batasan:** Khusus wallet. **Filosofi:** adjustment = obat (benerin angka), audit log = diagnosis
  (cari penyebab). Adjustment sendiri ikut tercatat di audit log — tanpa blind spot.

### 6. Multi-transaksi per chat
Satu pesan multi-baris = beberapa transaksi independen. Yang gagal tidak menggagalkan yang
sukses. `/batal` membatalkan seluruh batch sekaligus.

### 7. Backdate (transaksi susulan)
Format deterministik (Python, bukan tebakan AI): `10/10`, `3/10/2026`, `10 oktober`,
`kemarin`, `kemarin lusa`. Tanpa tanggal = hari ini (WIB).

### 8. OCR struk (Gemini Vision) — dengan crosscheck berlapis
- **Cara kerja:** Foto → Vision ekstrak per baris (nama, harga, kategori) → bot tampilkan
  daftar untuk **crosscheck** → user koreksi harga (`no 1 = 10000`) atau kategori
  (`no 2 = groceries`) → konfirmasi (`ya`) → tanya dompet → tiap item = 1 expense.
- **Pertahanan berlapis (defense in depth):**
  1. Prompt: baca total tercetak, JANGAN hitung qty×harga sendiri.
  2. Crosscheck otomatis: jumlah harga item vs total struk → warning ⚠️ kalau selisih > Rp1.000.
  3. Review user sebelum dicatat.
- **Detail:** State pending disimpan ke disk (tahan restart bot); input tak dikenal tidak
  menghapus struk (ditampilkan ulang + hint); `/batal` membatalkan sesi; caption foto
  dipakai sebagai jawaban pertama; foto diarsipkan di `receipts/`.

### 9. Audit log
- Setiap `create` (semua 6 jenis transaksi, termasuk adjustment) dan setiap `undo`
  menulis ke `audit_log` **dalam transaksi DB yang sama** (atomik — tidak ada jejak palsu).
- Detail JSON menyimpan pergerakan saldo (`moves`) dan perubahan hutang (`debt_changes`).
- Tabel dibuat otomatis saat bot start (`ensure_audit_table`, fail-fast kalau DB mati).

### 10. Perintah Telegram
| Perintah | Fungsi |
|---|---|
| `/start` | Bantuan + contoh (ID Telegram hanya untuk user belum diizinkan) |
| `/dompet` | Daftar wallet & saving |
| `/saldo` | Saldo semua wallet/saving + total |
| `/riwayat [n]` | n transaksi terakhir gabungan (default 10, maks 30) |
| `/utang` | Hutang/piutang terbuka + sisa + jatuh tempo |
| `/batal` | Undo pencatatan terakhir (atau batalkan sesi OCR) |
| `/audit [n]` | Jejak perubahan data terakhir (default 10, maks 30) |

### 11. Klarifikasi dompet ambigu (anti-tebak)
Sebutan seperti `bni` (cocok ke 3 wallet) TIDAK ditebak — bot menampilkan pilihan dan
meminta nama lengkap. Hanya kecocokan tepat/satu-satunya yang dipakai langsung.

### 12. Operasional otomatis
- **systemd service** (`cashflow-bot`): auto-start saat boot, auto-restart saat crash.
- **Backup harian** (`scripts/backup_db.sh` via cron jam 03:00): `pg_dump` + gzip,
  retensi bertingkat (harian 7, mingguan 4, bulanan 6).
- **Rotasi log**: `RotatingFileHandler` (5MB × 5 arsip) — `bot.log` tidak membesar tanpa batas.

---

## User Flow

```mermaid
flowchart TD
    A[User kirim pesan teks] --> B{Authorized?}
    B -- tidak --> Z[Tolak]
    B -- ya --> C{Ada sesi OCR menggantung?}
    C -- ya --> D{Jawaban dompet?}
    D -- ya, cocok --> E[Catat item struk + audit]
    D -- koreksi no/kategori/harga --> F[Tampilkan ulang list]
    D -- bukan --> A2[Tampilkan ulang + hint]
    C -- tidak --> G[Gemini parse → array JSON]
    G --> H{Valid per item?}
    H -- tidak --> I[Lapor gagal per item]
    H -- ya --> J[Resolusi dompet anti-tebak]
    J --> K{Ambigu?}
    K -- ya --> L[Minta klarifikasi]
    K -- tidak --> M[Simpan ke DB + geser saldo + audit]
    M --> N[Balas ringkasan + /batal]

    P[User kirim foto struk] --> Q[Gemini Vision ekstrak item]
    Q --> R[Tampilkan list + kategori untuk crosscheck]
    Q --> R2{Jumlah item ≠ total struk?}
    R2 -- ya --> R3[Tampilkan warning]
    R --> S{User koreksi?}
    S -- ya --> R
    S -- tidak / ya --> T[Tanya dompet]
    T --> D
```

---

## Arsitektur Sistem

```mermaid
flowchart LR
    TG[Telegram] <-->|long polling| BOT[src/bot.py]
    BOT -->|parse teks & vision| GEM[Gemini API]
    BOT <-->|psycopg2| PG[(PostgreSQL)]
    BOT --> LOG[logs/bot.log<br/>rotasi otomatis]
    BOT --> ARS[receipts/]

    subgraph Raspi [Raspberry Pi 4 — Ubuntu 24.04.5]
        SYS[systemd<br/>cashflow-bot.service]
        BOT
        PG
        CRON[cron 03:00<br/>backup_db.sh]
        CRON --> BK[~/backups/]
    end
```

### Modul (`src/` — package, 12 file)

| Modul | Tanggung jawab | ~Baris |
|---|---|---|
| `bot.py` | Entry point tipis: registrasi handler + `main()` | 90 |
| `config.py` | `.env`, konstanta, logging (rotasi), state global | 140 |
| `db.py` | Koneksi & query dasar | 55 |
| `normalize.py` | Deterministik: nominal, tanggal, jatuh tempo, nama, match dompet | 230 |
| `ai.py` | Prompt & panggilan Gemini (`google-genai`), OCR vision, validasi | 230 |
| `records.py` | 6 fungsi simpan transaksi | 190 |
| `undo.py` | `/batal` + jejak audit undo | 65 |
| `executor.py` | Resolusi dompet + orkestrasi 1 transaksi + audit create | 165 |
| `commands.py` | `/start /dompet /saldo /riwayat /utang /batal /audit` | 260 |
| `handlers.py` | Handler pesan teks | 85 |
| `ocr.py` | Alur foto struk (review → koreksi → dompet) | 270 |
| `audit.py` | Tabel `audit_log` + `log_audit()` | 60 |

- **Prinsip:** AI hanya mengusulkan JSON; validasi, normalisasi angka/tanggal/dompet
  dikerjakan deterministik di Python. Ketergantungan antar modul satu arah (tanpa cycle).
- **Cara jalan:** `python -m src.bot` dari folder project (bukan `python src/bot.py`).
- **Database:** PostgreSQL 16, 9 tabel (8 domain + `audit_log`).
- **AI Services:** `google-genai` SDK (migrasi dari `google-generativeai` yang deprecated).
- **External:** Telegram Bot API (pin IPv4 di `/etc/hosts` karena IPv6 tethering HP rusak;
  dukungan `TELEGRAM_PROXY` sebagai fallback).

### Skema Database

8 tabel domain sesuai `schema.md` (`wallets`, `savings`, `categories`, `income`,
`cashflow`, `moneyflow`, `debt`, `adjustment`) + tabel audit:

```sql
CREATE TABLE IF NOT EXISTS audit_log (
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id BIGINT NOT NULL,
    action TEXT NOT NULL,          -- 'create' | 'undo'
    kind TEXT NOT NULL,            -- expense|income|moneyflow|debt_new|debt_payment|adjustment
    row_id INTEGER,
    label TEXT NOT NULL,
    detail JSONB                   -- moves & debt_changes
);
```

> ⚠️ Histori: tabel live sempat meleset dari skema (`ammount`, `destination_type` hilang,
> `current_ballance`, `wallet_id`). Semua disejajarkan via `ALTER TABLE` (Oktober 2026).

---

## Struktur Folder

```text
finance_assistant/
├── src/
│   ├── __init__.py
│   ├── bot.py          # entry point (main)
│   ├── config.py       # env, konstanta, logging, state
│   ├── db.py           # koneksi & query dasar
│   ├── normalize.py    # normalisasi deterministik
│   ├── ai.py           # Gemini parser & OCR
│   ├── records.py      # 6 fungsi simpan
│   ├── undo.py         # /batal
│   ├── executor.py     # orkestrasi 1 transaksi
│   ├── commands.py     # perintah Telegram
│   ├── handlers.py     # handler pesan teks
│   ├── ocr.py          # alur struk
│   └── audit.py        # audit log
├── config/
│   ├── .env            # RAHASIA — tidak di-commit!
│   └── .env.example
├── sql/
│   └── seed_wallets.sql  # 12 wallet
├── scripts/
│   └── backup_db.sh    # backup harian (cron)
├── systemd/
│   └── cashflow-bot.service
├── logs/
│   ├── bot.log         # rotasi otomatis 5MB x 5
│   └── backup.log
├── receipts/           # arsip foto struk
├── requirements.txt
└── .gitignore
```

---

## Panduan Pengguna

### Operasional harian (via systemd)

```bash
sudo systemctl restart cashflow-bot.service  # restart
sudo systemctl status cashflow-bot.service   # cek status
journalctl -u cashflow-bot.service -f        # log live
```

### Variabel `.env`
`TELEGRAM_BOT_TOKEN`, `GEMINI_API_KEY`, `GEMINI_MODEL` (=`gemini-3.5-flash`),
`DB_HOST/DB_PORT/DB_NAME/DB_USER/DB_PASS`, `ALLOWED_USER_IDS`, `TELEGRAM_PROXY` (opsional).

### Contoh pemakaian

```
beli bensin 10k cash
gaji 5jt masuk bca
tf dari bni ocid ke jago ocid 2jt
tarik tunai mandiri 100k
saya ngutang ke mila 500ribu
beto bayar hutang 300k masuk bni ocid
saldo awal cash 200k
10/10 beli skincare 70k bni ocid
[kirim foto struk] -> review -> "no 1 = 10000" -> "ya" -> "cash"
/saldo   /riwayat 20   /utang   /audit   /batal
```

### Aturan penulisan nominal
`10k`/`50rb` = ribu · `5jt`/`2juta` = juta · `200.000`/`25000` = literal.
⚠️ `m` TIDAK didukung (di Indonesia `m` = miliar — terlalu berisiko) → tulis `jt` eksplisit.

### Koreksi struk saat review
- `no 2 = groceries` → betulkan kategori item 2
- `no 1 = 10000` / `No 1 10.000` → betulkan harga item 1
- `ya` → lanjut ke pertanyaan dompet; atau langsung jawab nama dompet
- Nilai dibedakan otomatis (angka = harga, kata = kategori); typo kategori → error yang mengajari

---

## Jurnal Development

| Tanggal | Kejadian |
|---|---|
| 2026-10-15 | Goal "build a budget and track spending" dipilih; intake terhenti di pivot project. |
| 2026-10-16 | Push awal: hardened `bot.py` (parser Gemini ketat + validasi Python, saldo atomik, auth user ID, `/dompet`, `/batal`), `requirements.txt`, `.env.example`, `seed_wallets.sql`. |
| 2026-10-16 | **Insiden keamanan:** `config/.env` sempat ter-commit saat repo publik → repo di-private, file dihapus dari tracking; token & API key dirotasi. Pelajaran: jangan pernah commit secrets. |
| 2026-10-16 | **Jaringan:** IPv6 via tethering HP rusak → pin IPv4 di `/etc/hosts` (`149.154.166.110`). `TELEGRAM_PROXY` sebagai fallback. |
| 2026-10-16 | **Model pensiun:** `gemini-1.5-flash` 404 → `gemini-2.5-flash` → `gemini-3.5-flash` (aktif). |
| 2026-10-16 | Bug: Gemini kembalikan key `source` bukan `item` → validasi toleran + prompt ditegaskan. |
| 2026-10-16 | **Anti-tebak dompet** (`match_holders()`): ambigu → tanya user, bukan ditebak diam-diam. |
| 2026-10-16 | **Backdate** deterministik: `10/10`, `3/10/2026`, `10 oktober`, `kemarin`, `kemarin lusa`. |
| 2026-10-16 | Transfer antar dompet sendiri = `moneyflow`; transfer ke orang = `expense`/`gift`. |
| 2026-10-16 | **Skema live vs kode:** `income.amount`, `moneyflow`, `savings.current_balance`, `adjustment` diperbaiki via `ALTER TABLE`. |
| 2026-10-16 | **Multi-transaksi per chat** (array JSON, eksekusi per item, `/batal` batch). |
| 2026-10-16 | **Bug audit:** saldo saving tidak kegeser (+ `/batal` merusaknya) → fix; `date.today()` → `today_wib()` (Asia/Jakarta); suffix `m` dihapus (risiko miliar). |
| 2026-10-19 | Hutang tanpa dompet; `normalize_person`; `due_date` deterministik. |
| 2026-10-19 | Perintah baca: `/saldo`, `/riwayat [n]`, `/utang`. |
| 2026-10-19 | `/riwayat`: UNION gagal (`DatatypeMismatch`) → 3 query terpisah digabung di Python. |
| 2026-10-19 | **Arah debt_payment** berbasis subjek (`[NAMA] bayar`→in, `bayar hutang ke [NAMA]`→out) + error cerdas. |
| 2026-10-19 | **Fitur `adjustment`**: `saldo awal` / `koreksi saldo ... jadi ...` (absolut, selisih tercatat). |
| 2026-10-19 | **OCR struk (Vision):** per-baris (nama/harga/kategori) → review → koreksi → dompet → 1 expense/item. |
| 2026-10-19 | **5 bug fix OCR** (audit otomatis): pending tahan restart (disk), input tak dikenal tidak hapus struk, crosscheck jumlah vs total, caption sebagai jawaban, warning timpa struk. |
| 2026-10-19 | **Split monolit** → package `src/` 12 modul; jalan via `python -m src.bot`. Sempat ada bug `@contextmanager` terpotong → fix. |
| 2026-10-19 | **Backup harian** (`scripts/backup_db.sh`, cron 03:00, retensi 7/4/6) + **rotasi log** (5MB×5). |
| 2026-10-19 | **systemd service** `cashflow-bot`: auto-start, auto-restart (aman untuk tethering lambat). |
| 2026-10-19 | **Audit log** (`audit_log` + `/audit`): semua create/undo tercatat atomik, termasuk adjustment. |
| 2026-10-19 | **Migrasi `google-generativeai` → `google-genai`** (deprecated). |
| 2026-10-19 | **Koreksi harga struk** (`no 1 = 10000`): nilai dibedakan otomatis angka vs kategori. |
| 2026-10-19 | **Prompt OCR dipertegas:** baca total tercetak, JANGAN hitung qty×harga (kasus nyata: `2 ... 10000` dibaca 20000). |

---

## Referensi Teknis

### Aturan parser Gemini (ringkas)
- Output HANYA array JSON valid, maks 10 object; pesan user = DATA bukan instruksi.
- `amount_raw` disalin persis; `wallet_source/dest` disalin persis (sistem yang mencocokkan).
- `tx_type`: `expense` | `income` | `moneyflow` | `debt_new` | `debt_payment` | `adjustment`.

### Kategori valid
`groceries`, `eating_out`, `transport`, `housing_utilities`, `selfcare`, `health`,
`education_learning`, `entertainment`, `shopping`, `investment`, `debt`, `gift`, `miscellaneous`.

### Pencocokan dompet (`match_holders`)
1. Persis (case-insensitive) → pakai. 2. Satu kandidat → pakai.
3. Banyak kandidat → tanya user. 4. Tidak ada → "tidak dikenal".
Pengecualian: `tarik tunai <bank>` → sisi kosong diisi wallet `cash`.

### Dompet terdaftar (12)
Mandiri ocid, BNI ocid, Jago ocid, Jago cadangan, Shopeepay ocid, BCA mila,
BNI Mila 1, BNI Mila 2, ago cash buffer, Jago kantong utama, Shopeepay mila, cash.

### Audit log
- Ditulis di dalam transaksi yang sama dengan mutasi (atomik).
- `detail` JSON: `{"moves": [[tipe, id, delta]], "debt_changes": [...]}`.
- `/audit [n]` menampilkan `created_at` dalam WIB.

---

## Maintenance

### Operasional
| Perintah | Fungsi |
|---|---|
| `sudo systemctl restart cashflow-bot.service` | Restart bot |
| `sudo systemctl status cashflow-bot.service` | Cek status |
| `journalctl -u cashflow-bot.service -f` | Log live |
| `crontab -l` | Cek jadwal backup |

### Backup & restore
- Lokasi: `~/backups/finance_assistant/` (`daily_*`, `weekly_*`, `monthly_*`).
- Tes restore berkala ke database terpisah (lihat Installation Guide §10).
- Backup ada di Pi yang sama — copy manual ke laptop/HP sesekali untuk proteksi SD card.

### Error umum & penanganan
| Gejala | Penyebab | Solusi |
|---|---|---|
| `409 Conflict` | 2 instance jalan | Matikan yang manual; systemd hanya 1 |
| `UndefinedColumn` | kolom live ≠ skema | Bandingkan `\d` di psql dengan skema, `ALTER TABLE` |
| `AI parser sedang bermasalah` | Gemini error / model pensiun | Cek `GEMINI_MODEL` & key, `logs/bot.log` |
| Telegram timeout | IPv6 tethering rusak | Cek pin IPv4 di `/etc/hosts` (IP bisa berubah) |
| `Database tidak bisa dihubungi` | PostgreSQL mati / kredensial salah | `systemctl status postgresql`, cek `.env` |
| `ModuleNotFoundError: No module named 'src'` | Salah working directory | Jalan dari folder project: `python -m src.bot` |

---

## Informasi yang Belum Terdokumentasi

- [TODO] **Akun saving**: daftar & saldo awal tabungan (tabel `savings` masih kosong).
- [TODO] **Rotasi password DB** + **revoke GitHub PAT** (status belum dikonfirmasi).
- [TODO] **Advisor bot (bot 2)**: desain query analitik — belum dibangun.
- [TODO] **Dashboard**: rekap visual — belum dibangun.
- [TODO] **Category learning**: bot belajar kategori dari histori (diusulkan, belum disetujui).
- [TODO] **`/koreksi`**: edit nominal transaksi tercatat (diusulkan, belum disetujui).
- [TODO] **Versioning semver** (saat ini acuan = hash commit).

---

## Pembelajaran (Learning Notes)

1. **Jangan percaya skema live = skema dokumen.** Selalu verifikasi dengan
   `information_schema` sebelum menuduh kode.
2. **AI tidak boleh menebak hal material.** Ambigu → tanya. Pola ini berulang:
   dompet, arah hutang, kategori struk.
3. **Deterministik > LLM** untuk angka, tanggal, dan pencocokan nama.
4. **Bahasa natural itu ambigu** — aturan subjek eksplisit di prompt + pesan error
   yang mengajari user lebih baik daripada tebakan diam-diam.
5. **Timezone adalah bug yang menunggu.** Selalu pin zona waktu user di kode.
6. **Singkatan lokal berbahaya** (`m` = miliar). Validasi harus konservatif.
7. **OCR tidak akan pernah 100% benar** — desain berlapis: prompt yang baik +
   crosscheck otomatis + review user + koreksi mudah. Salahnya ketahuan > tidak salah.
8. **Audit log melengkapi adjustment, bukan menggantikannya.** Adjustment = obat,
   audit = diagnosis. Bug saldo saving dulu tidak akan ketahuan polanya tanpa jejak.
9. **Refactor termurah adalah sekarang.** Split 1500→12 modul sebelum todo membesar;
   verifikasi dengan differential test, bukan cuma compile.
10. **Slicing file harus menyertakan decorator** di atas `def` — nyaris lolos tanpa
    terdeteksi (`@contextmanager` terpotong → semua query DB gagal).
11. **Fail-fast untuk dependensi kritis.** `ensure_audit_table()` di startup:
    lebih baik crash-loop yang kelihatan daripada jalan tanpa audit yang sunyi.

# Panduan Instalasi — finance_assistant

> Dari nol sampai bot bisa dipakai: environment, tools, source code,
> database, konfigurasi, sampai operasional harian.
> Terakhir diperbarui: 2026-10-19. Berlaku untuk commit `62694fe` ke atas.

---

## 0. Gambaran & Prasyarat

**Yang dibangun:** Bot Telegram pencatat keuangan pribadi (Python + PostgreSQL +
Gemini API) yang jalan 24/7 di Raspberry Pi sebagai systemd service.

**Yang perlu disiapkan sebelum mulai:**

| Kebutuhan | Keterangan |
|---|---|
| Raspberry Pi 4 (RAM 4GB+) | OS Ubuntu 24.04, konek internet (tethering HP bisa) |
| Akun GitHub | Punya akses ke repo `skwnpro/finance_assistant` (private) |
| Bot Telegram | Buat via @BotFather (lihat Langkah 7) |
| Gemini API key | Gratis via [aistudio.google.com](https://aistudio.google.com) |
| Telegram user ID kamu | Didapat via `/start` (lihat Langkah 7) |
| Akses `sudo` di Raspberry Pi | Untuk install package & systemd |

**Waktu estimasi:** 45–90 menit untuk instalasi penuh pertama kali.

---

## 1. Install Tools Sistem

```bash
sudo apt update
sudo apt install -y git python3-venv postgresql postgresql-contrib
```

Verifikasi:

```bash
git --version
python3 --version        # harus 3.12+
psql --version           # harus 16.x
```

---

## 2. Setup PostgreSQL

Buat user database dan database-nya:

```bash
sudo -u postgres psql -c "CREATE USER finance_admin WITH PASSWORD 'GANTI-DENGAN-PASSWORD-KUAT';"
sudo -u postgres psql -c "CREATE DATABASE finance_assistant OWNER finance_admin;"
```

> ⚠️ Password ini dipakai bot untuk konek ke database. Simpan baik-baik —
> nanti dimasukkan ke `config/.env` sebagai `DB_PASS`.

Tes koneksi (akan diminta password):

```bash
psql -h localhost -U finance_admin -d finance_assistant -c "SELECT 1;"
```

Kalau keluar `1`, berarti beres.

---

## 3. Clone Source Code

Repo bersifat **private** — clone via HTTPS (nanti diminta username + Personal
Access Token GitHub sebagai password) atau pakai SSH key yang sudah terdaftar.

```bash
mkdir -p ~/project
git clone https://github.com/skwnpro/finance_assistant.git ~/project/finance_assistant
cd ~/project/finance_assistant
```

---

## 4. Buat Skema Database

Simpan DDL di bawah sebagai `sql/schema.sql`, lalu jalankan. (**Catatan:**
tabel `audit_log` TIDAK perlu dibuat manual — bot membuatnya otomatis saat start.)

```sql
-- 1. wallets
CREATE TABLE IF NOT EXISTS wallets (
    id SERIAL PRIMARY KEY,
    wallets_name VARCHAR(100) NOT NULL,
    account_number VARCHAR(50),
    current_balance NUMERIC(15, 2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. savings
CREATE TABLE IF NOT EXISTS savings (
    id SERIAL PRIMARY KEY,
    saving_name VARCHAR(100) NOT NULL,
    current_balance NUMERIC(15, 2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. categories
CREATE TABLE IF NOT EXISTS categories (
    id SERIAL PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL,
    sub_category_name VARCHAR(50)
);

INSERT INTO categories (category_name, sub_category_name) VALUES
('groceries', 'Beras'), ('groceries', 'Telur'), ('groceries', 'Sayur'),
('groceries', 'Minyak'), ('groceries', 'Gula'), ('groceries', 'Kopi'), ('groceries', 'Susu'),
('eating_out', 'Bakso'), ('eating_out', 'Mie Ayam'), ('eating_out', 'Starbucks'),
('eating_out', 'Ayam Geprek'), ('eating_out', 'GrabFood'),
('transport', 'Pertalite'), ('transport', 'Parkir'), ('transport', 'Tol'),
('transport', 'Bus'), ('transport', 'Kereta'), ('transport', 'servis'),
('housing_utilities', 'listrik'), ('housing_utilities', 'air'),
('housing_utilities', 'internet'), ('housing_utilities', 'kontrakan'),
('selfcare', 'Potong Rambut'), ('selfcare', 'Sabun'), ('selfcare', 'Shampoo'),
('selfcare', 'Skincare'), ('selfcare', 'Pasta Gigi'),
('health', 'Dokter'), ('health', 'Obat'), ('health', 'Vitamin'), ('health', 'BPJS'),
('education_learning', 'Udemy'), ('education_learning', 'Course'),
('education_learning', 'Buku'), ('education_learning', 'Gemini'), ('education_learning', 'ChatGPT'),
('entertainment', 'Steam'), ('entertainment', 'Game'), ('entertainment', 'Netflix'),
('entertainment', 'Spotify'), ('entertainment', 'Youtube Premium'),
('shopping', 'Baju'), ('shopping', 'Sepatu'), ('shopping', 'Tas'), ('shopping', 'Aksesoris'),
('investment', 'Bibit'), ('investment', 'Stockbit'), ('investment', 'Reksadana'),
('investment', 'Saham'), ('investment', 'Emas'), ('investment', 'Crypto'),
('debt', 'Bayar cicilan'), ('debt', 'Bayar hutang'),
('gift', 'kirim orang tua'), ('gift', 'sedekah'), ('gift', 'kirim adik'),
('miscellaneous', 'Lain-lain');

-- 4. income
CREATE TABLE IF NOT EXISTS income (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL DEFAULT CURRENT_DATE,
    source VARCHAR(100) NOT NULL,
    amount NUMERIC(15, 2) NOT NULL,
    destination_wallets INT REFERENCES wallets(id),
    notes TEXT
);

-- 5. cashflow
CREATE TABLE IF NOT EXISTS cashflow (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL DEFAULT CURRENT_DATE,
    category VARCHAR(50) DEFAULT 'miscellaneous',
    sub_category VARCHAR(50),
    item VARCHAR(150) NOT NULL,
    amount NUMERIC(15, 2) NOT NULL,
    wallets_id INT REFERENCES wallets(id),
    notes TEXT
);

-- 6. moneyflow
CREATE TABLE IF NOT EXISTS moneyflow (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL DEFAULT CURRENT_DATE,
    source_type VARCHAR(20) CHECK (source_type IN ('wallet', 'saving')),
    source_id INT NOT NULL,
    destination_type VARCHAR(20) CHECK (destination_type IN ('wallet', 'saving')),
    destination_id INT NOT NULL,
    amount NUMERIC(15, 2) NOT NULL,
    notes TEXT
);

-- 7. debt
CREATE TABLE IF NOT EXISTS debt (
    id SERIAL PRIMARY KEY,
    person_name VARCHAR(100) NOT NULL,
    type VARCHAR(20) CHECK (type IN ('debt', 'receivable')),
    original_amount NUMERIC(15, 2) NOT NULL,
    remaining_amount NUMERIC(15, 2) NOT NULL,
    installment NUMERIC(15, 2),
    due_date DATE,
    status VARCHAR(20) DEFAULT 'unpaid'
);

-- 8. adjustment
CREATE TABLE IF NOT EXISTS adjustment (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL DEFAULT CURRENT_DATE,
    wallets_id INT REFERENCES wallets(id),
    amount NUMERIC(15, 2) NOT NULL,
    adjustment_type VARCHAR(20),
    notes TEXT
);
```

Jalankan:

```bash
psql -h localhost -U finance_admin -d finance_assistant -f sql/schema.sql
```

Lalu seed daftar dompet bawaan (12 wallet):

```bash
psql -h localhost -U finance_admin -d finance_assistant -f sql/seed_wallets.sql
```

Verifikasi tabel terbentuk:

```bash
psql -h localhost -U finance_admin -d finance_assistant -c "\dt"
psql -h localhost -U finance_admin -d finance_assistant -c "SELECT wallets_name FROM wallets;"
```

> Harus muncul 8 tabel (+ `audit_log` akan muncul sendiri setelah bot pertama
> kali jalan), dan 12 nama wallet.

---

## 5. Setup Python Environment

```bash
cd ~/project/finance_assistant
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

> Selalu aktifkan venv (`source venv/bin/activate`) sebelum menjalankan
> perintah Python apa pun terkait project ini.

---

## 6. Konfigurasi `.env`

```bash
cp config/.env.example config/.env
nano config/.env
```

Isi setiap variabel:

| Variabel | Cara dapat | Contoh |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Chat @BotFather → `/newbot` → ikuti instruksi → copy token | `8702745579:AAG...` |
| `GEMINI_API_KEY` | [aistudio.google.com](https://aistudio.google.com) → Get API Key | `AIza...` |
| `GEMINI_MODEL` | Nama model Gemini yang dipakai | `gemini-3.5-flash` |
| `DB_HOST` | Host PostgreSQL | `localhost` |
| `DB_PORT` | Port PostgreSQL | `5432` |
| `DB_NAME` | Nama database | `finance_assistant` |
| `DB_USER` | User database | `finance_admin` |
| `DB_PASS` | Password dari Langkah 2 | — |
| `ALLOWED_USER_IDS` | ID Telegram kamu (lihat bawah) | `123456789` |
| `TELEGRAM_PROXY` | Opsional, kosongkan jika tidak perlu | — |

**Cara dapat Telegram user ID:** setelah bot jalan (Langkah 8), kirim `/start`
ke bot — ID kamu ditampilkan karena belum terdaftar. Masukkan ke
`ALLOWED_USER_IDS`, restart bot.

> 🔒 **JANGAN PERNAH** commit `config/.env` ke git. File ini sudah ada di
> `.gitignore`. Kalau tidak sengaja ter-commit saat repo publik, anggap semua
> secret bocor dan rotasi semuanya (buat ulang token/key/password).

---

## 7. Perbaiki Akses Jaringan ke Telegram (khusus tethering HP)

Via tethering, jalur IPv6 ke `api.telegram.org` sering rusak (hang), sedangkan
IPv4 normal. Tes dulu:

```bash
curl -4 -s -o /dev/null -w "%{http_code}\n" --max-time 10 https://api.telegram.org
```

- Kalau cepat keluar `302`/`200` → aman, lewati langkah ini.
- Kalau hang/timeout → pin IPv4 di `/etc/hosts`:

```bash
echo "149.154.166.110 api.telegram.org" | sudo tee -a /etc/hosts
```

> ⚠️ IP Telegram bisa berubah sewaktu-waktu. Kalau bot tiba-tiba tidak bisa
> konek padahal internet jalan, cek ulang IP ini (`nslookup api.telegram.org`)
> atau hapus barisnya dan andalkan DNS normal.

---

## 8. Tes Jalan Manual (sebelum systemd)

```bash
cd ~/project/finance_assistant
source venv/bin/activate
python -m src.bot
```

Harus muncul log `Bot cashflow aktif (model=...)` tanpa error. Lalu dari Telegram:

1. `/start` → bot membalas (catat user ID kamu untuk `ALLOWED_USER_IDS`)
2. Isi `ALLOWED_USER_IDS` di `.env`, `Ctrl+C`, jalankan lagi
3. `/saldo` → daftar wallet & saldo
4. `beli bensin 10k cash` → tercatat
5. `/batal` → pencatatan dibatalkan (saldo kembali)
6. Kirim foto struk → alur review item muncul

Kalau semua lolos, `Ctrl+C` untuk berhenti, lanjut ke systemd.

> ⚠️ Jangan pernah jalankan 2 instance bersamaan — Telegram menolak dengan
> error `409 Conflict`. Satu bot = satu proses.

---

## 9. Install Systemd Service (jalan 24/7)

Unit file sudah tersedia di repo: `systemd/cashflow-bot.service`.
Sesuaikan `ExecStart` jika path venv kamu berbeda.

```bash
cd ~/project/finance_assistant
git pull --rebase

# WAJIB: matikan bot manual dulu (kalau tidak: 409 Conflict)
pkill -f "src.bot"; sleep 2

sudo cp systemd/cashflow-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable cashflow-bot.service   # auto-start saat boot
sudo systemctl start cashflow-bot.service

# Verifikasi
sudo systemctl status cashflow-bot.service   # harus "active (running)"
```

Lihat log live:

```bash
journalctl -u cashflow-bot.service -f
```

Perintah operasional baru (pengganti `pkill` + `nohup`):

| Perintah | Fungsi |
|---|---|
| `sudo systemctl restart cashflow-bot.service` | Restart bot |
| `sudo systemctl stop cashflow-bot.service` | Stop bot |
| `sudo systemctl status cashflow-bot.service` | Cek status |
| `journalctl -u cashflow-bot.service -f` | Log live |
| `journalctl -u cashflow-bot.service --since "1 hour ago"` | Log 1 jam terakhir |

---

## 10. Setup Backup Harian

```bash
cd ~/project/finance_assistant
chmod +x scripts/backup_db.sh
./scripts/backup_db.sh                      # tes manual sekali
ls -lh ~/backups/finance_assistant/         # pastikan file .gz terbentuk
```

Kalau OK, pasang cron (jalan tiap jam 3 pagi, retensi otomatis:
harian 7, mingguan 4, bulanan 6):

```bash
crontab -e
```

Tambahkan baris:

```
0 3 * * * /home/exedy/project/finance_assistant/scripts/backup_db.sh >> /home/exedy/project/finance_assistant/logs/backup.log 2>&1
```

Verifikasi: `crontab -l`.

**Tes restore** (penting — backup yang tidak pernah dites = tidak ada backup):

```bash
sudo -u postgres psql -c "CREATE DATABASE finance_assistant_test OWNER finance_admin;"
gunzip -c ~/backups/finance_assistant/daily_YYYYMMDD_HHMMSS.sql.gz | \
  psql -h localhost -U finance_admin -d finance_assistant_test -f -
psql -h localhost -U finance_admin -d finance_assistant_test -c "SELECT COUNT(*) FROM cashflow;"
sudo -u postgres psql -c "DROP DATABASE finance_assistant_test;"
```

> Catatan: backup tersimpan di Raspberry Pi yang sama. Ini melindungi dari
> kasus paling umum (salah hapus, migrasi gagal). Sesekali copy folder
> `~/backups/` ke laptop/HP untuk proteksi kalau SD card rusak.

---

## 11. Checklist Verifikasi Akhir

- [ ] `sudo systemctl status cashflow-bot.service` → `active (running)`
- [ ] `/start` dibalas bot
- [ ] `/saldo` menampilkan 12 wallet
- [ ] Pesan teks tercatat (`beli bensin 10k cash`)
- [ ] Foto struk → review item → koreksi → dompet → tercatat
- [ ] `/audit` menampilkan jejak transaksi
- [ ] `/batal` membatalkan pencatatan terakhir
- [ ] `crontab -l` menampilkan jadwal backup
- [ ] File backup `.gz` terbentuk di `~/backups/finance_assistant/`
- [ ] Reboot Pi → bot hidup sendiri (`sudo reboot`, tunggu 2–3 menit, `/saldo`)

---

## 12. Troubleshooting

| Gejala | Penyebab umum | Solusi |
|---|---|---|
| `409 Conflict` di log | 2 instance jalan bersamaan | `pkill -f "src.bot"` (manual) / `sudo systemctl stop` yang ganda |
| `UndefinedColumn ... does not exist` | Kolom tabel live ≠ skema | Bandingkan `\d nama_tabel` di psql dengan `sql/schema.sql`, perbaiki via `ALTER TABLE` |
| `AI parser sedang bermasalah` | Gemini API error / model pensiun | Cek `GEMINI_API_KEY` & `GEMINI_MODEL` di `.env`; lihat `logs/bot.log` |
| Bot tidak merespons, log timeout Telegram | IPv6 tethering rusak | Ulangi Langkah 7 (cek/pin IPv4) |
| `Database tidak bisa dihubungi` | PostgreSQL mati / kredensial salah | `sudo systemctl status postgresql`; cek `DB_*` di `.env` |
| `ModuleNotFoundError: No module named 'src'` | Jalan dari folder yang salah | Harus dari `~/project/finance_assistant`: `python -m src.bot` |
| `Config kurang: ...` saat start | `.env` belum diisi lengkap | Lengkapi `config/.env` (lihat `.env.example`) |
| Service `failed` berulang | Salah path venv / `.env` kosong | `journalctl -u cashflow-bot.service` untuk lihat error persisnya |
| Backup cron tidak jalan | `crontab -e` belum disimpan | `crontab -l` harus menampilkan barisnya; cek `logs/backup.log` |

---

## 13. Update Rutin

Setiap ada update kode baru:

```bash
cd ~/project/finance_assistant
git pull --rebase
# kalau requirements.txt berubah:
source venv/bin/activate && pip install -r requirements.txt
sudo systemctl restart cashflow-bot.service
sudo systemctl status cashflow-bot.service
```

## Terkait

**Dokumen utama:**

- [[finance_assistant_dokumentasi]] — dokumentasi lengkap project (arsitektur, referensi, jurnal)

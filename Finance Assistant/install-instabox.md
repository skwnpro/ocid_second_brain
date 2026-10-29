# Install Finance Assistant di Instabox

Sistem pencatat keuangan Telegram (2 bot + 1 database) yang jalan di VPS
gratis instabox.sh. Dokumen ini merangkum instalasi dari nol sampai kedua
bot auto-start setiap mesin nyala.

Lihat juga: [[postgresql]] (perintah database yang dipakai project ini),
[[flow-diagram]] (alur data dan flowchart kedua bot).

## Gambaran sistem

- **Recorder bot** — mencatat pengeluaran, pemasukan, transfer, hutang,
  dan OCR struk. Token: `TELEGRAM_BOT_TOKEN`.
- **Advisor bot** — tanya-jawab read-only di atas database yang sama
  (total, per kategori, ringkasan, saldo, hutang). Token:
  `ADVISOR_BOT_TOKEN`. Tidak bisa menulis apa pun ke database.
- **Database** — PostgreSQL, database `finance_assistant`, di mesin yang
  sama.
- **Repo** — `github.com/skwnpro/finance_assistant` (private), kode di
  `src/` (recorder) dan `advisor/` (advisor).

## Spesifikasi mesin

- Nama mesin: `finance-bot`, instabox.sh (program invite, gratis).
- Firecracker microVM, Debian 13 (trixie), 4 vCPU / 8 GB RAM.
- Disk: 16 GB root + 20 GB persistent di `/data`. Home user ada di
  `/data/home` (bukan `/home`).
- User: `user`. PID 1 = systemd. Python 3.13.5.

## Batasan free tier dan strategi

- Mesin **pause setelah 30 menit idle ATAU 8 jam jalan**. Pause = beku:
  memori dan proses dipertahankan, file di `/data` aman, resume < 1 detik.
- Mesin **tidak auto-resume**. Bangunnya manual: SSH, `instabox exec`,
  dashboard, atau akses port yang dipublish.
- `instabox logs` **tidak** membangunkan mesin.
- Strategi yang dipakai: **pause/resume manual**. Mesin dinyalakan saat
  mau mencatat, dimatikan (pause) setelah selesai. Tidak ada scheduler
  on/off otomatis dan tidak ada pinger eksternal.
- Konsekuensi: pesan Telegram yang dikirim saat mesin pause akan
  diantre Telegram dan dikirim berurutan saat mesin bangun (praktisnya
  ~24 jam, jangan diandalkan lebih). Pesan yang diproses lewat tengah
  malam dicatat dengan tanggal proses, bukan tanggal kirim — kalau
  penting, sebutkan tanggalnya eksplisit (backdate).

## Persiapan

- Buat mesin di dashboard instabox.sh, pastikan status Running.
- Akses awal: tombol SSH di dashboard (perintah SSH + token berumur
  pendek), atau CLI `instabox` (`instabox login`, `instabox ssh
  finance-bot`).
- Catatan Termux: CLI `instabox` bisa diinstall di Termux (arm64), tapi
  `~/.local/bin` tidak ada di PATH (tambahkan manual ke `~/.bashrc`),
  dan binary non-Termux membaca `/etc/resolv.conf` sistem (bukan
  `$PREFIX/etc/resolv.conf`) — kalau DNS gagal dengan `[::1]:53`,
  pakai SSH dashboard saja, itu jalur tercepat.

## Langkah instalasi

### 1. Update dan tools

```bash
sudo apt-get update
sudo apt-get install -y git python3-venv postgresql nano
```

Image instabox tidak membawa daftar apt dan tidak membawa `nano`,
jadi dua perintah di atas wajib. Hasilnya: PostgreSQL 17.

### 2. Database dan user

```bash
psql -h localhost -U finance_admin -d finance_assistant
```

Kalau user `finance_admin` belum ada, buat dulu sebagai superuser
postgres (karena `finance_admin` tidak punya hak CREATEROLE):

```bash
sudo -u postgres psql -d finance_assistant
```

Isi database (jalankan sebagai user yang sesuai, lihat catatan di
bawah):

```sql
CREATE USER finance_admin WITH PASSWORD 'PASSWORD-KUAT';
CREATE DATABASE finance_assistant OWNER finance_admin;
ALTER DATABASE finance_assistant SET timezone TO 'Asia/Jakarta';
```

Tes koneksi:

```bash
psql -h localhost -U finance_admin -d finance_assistant -c "SELECT 1;"
```

Harus mengembalikan `1`. Catatan:

- Password database hanya diketahui pemilik, jangan pernah dikirim
  lewat chat.
- Kalau tidak sengaja membuat user/database dengan password
  placeholder, perbaiki dengan urutan: `DROP DATABASE` dulu, baru
  `DROP USER` (database yang dimiliki user menghalangi penghapusan
  user).
- Error `Only roles with the CREATEROLE attribute may create roles`
  artinya sesi psql saat itu bukan superuser — ulangi lewat
  `sudo -u postgres psql`.

### 3. Clone repo

```bash
mkdir -p ~/project
git clone https://github.com/skwnpro/finance_assistant.git ~/project/finance_assistant
cd ~/project/finance_assistant
```

Repo private, jadi clone butuh auth: PAT (Contents read-only cukup
untuk clone; read+write kalau mau push) atau SSH key. Simpan kredensial
di password manager atau SSH key — PAT yang hilang berarti pull
berikutnya minta token lagi. Rute SSH yang awet:

```bash
ssh-keygen -t ed25519 -C "vps-finance-bot"
cat ~/.ssh/id_ed25519.pub
```

Tempel public key ke GitHub → Settings → SSH and GPG keys → New SSH
key, lalu:

```bash
git remote set-url origin git@github.com:skwnpro/finance_assistant.git
ssh -T git@github.com
git pull
```

### 4. Virtual environment dan dependensi

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Isi `requirements.txt`: `psycopg2-binary`, `python-telegram-bot`,
`python-dotenv`, `google-genai`. **Venv harus ada di
`~/project/finance_assistant/venv`** — service systemd mencari
`venv/bin/python` di jalur itu. Venv yang dibuat di tempat lain adalah
penyebab error `203/EXEC` (lihat troubleshooting).

### 5. Skema database

**PENTING — jangan pakai `sql/schema.sql` dari repo tanpa memeriksa.**
Repo pernah membawa `schema.sql` versi Oktober 2026 dengan nama kolom
yang salah (`ammount`, `current_ballance`, `wallet_id`,
`moneyflow` tanpa `destination_type`). Skema yang benar ada di
`schema.md` / file `schema.sql` terverifikasi. Periksa dulu:

```bash
grep -n "ammount\|wallet_id\|current_ballance" sql/schema.sql
```

Kalau grep menemukan apa pun, file itu versi lama — jangan dipakai.
Muat skema yang benar (8 tabel + 57 kategori):

```bash
psql -h localhost -U finance_admin -d finance_assistant -f schema.sql
```

Output yang diharapkan: 8× `CREATE TABLE` + `INSERT 0 57`.

### 6. Seed dompet

```bash
psql -h localhost -U finance_admin -d finance_assistant -f sql/seed_wallets.sql
```

12 dompet: Mandiri ocid, BNI ocid, Jago ocid, Jago cadangan,
Shopeepay ocid, BCA mila, BNI Mila 1, BNI Mila 2, Jago cash buffer,
Jago kantong utama, Shopeepay mila, cash. Catatan: seed lama sempat
menulis `ago cash buffer` (huruf J hilang) — kalau menemukan itu,
perbaiki dengan:

```bash
sed -i "s/ago cash buffer/Jago cash buffer/" sql/seed_wallets.sql
psql -h localhost -U finance_admin -d finance_assistant \
  -c "TRUNCATE wallets RESTART IDENTITY CASCADE;"
psql -h localhost -U finance_admin -d finance_assistant -f sql/seed_wallets.sql
```

Daftar tabungan (tabel `savings`) masih kosong sampai pemilik
memberikan daftarnya — bukan error, memang belum didefinisikan.

### 7. File .env

```bash
nano config/.env
```

Isi (token dan password diisi pemilik, jangan dikirim lewat chat):

```
TELEGRAM_BOT_TOKEN=<token bot recorder dari BotFather>
ADVISOR_BOT_TOKEN=<token bot advisor dari BotFather>
GEMINI_API_KEY=<kunci Gemini>
GEMINI_MODEL=gemini-3.5-flash
DB_HOST=localhost
DB_PORT=5432
DB_NAME=finance_assistant
DB_USER=finance_admin
DB_PASSWORD=<password finance_admin>
DB_READER_USER=finance_reader
DB_READER_PASS=<password finance_reader>
ALLOWED_USER_IDS=<id Telegram pemilik, dipisah koma>
```

`config/.env` tidak boleh masuk git (sudah dikeluarkan dari tracking
sejak insiden commit .env saat repo sempat public; token Telegram dan
kunci Gemini dari periode itu sudah dirotasi).

### 8. Tes manual recorder

```bash
source venv/bin/activate
python -m src.bot
```

Log harus menunjukkan `Bot cashflow aktif (model=...)`. Tes dari
Telegram: `/start`, `/saldo` (12 dompet muncul), kirim `beli bensin
10k cash`, lalu `/batal` (transaksi terhapus). Kalau bot menjawab
`Config kurang`, ada baris `.env` yang belum diisi.

### 9. Service systemd recorder (auto-start)

Matikan dulu bot manual (`Ctrl+C`) — dua instance dengan token yang
sama akan rebutan dan kalah satu (error 409 Conflict).

File service ada di repo: `systemd/cashflow-bot.service`. Sesuaikan
user dan path dari versi Raspi ke VPS:

```bash
sed -e 's/^User=exedy/User=user/' \
    -e 's|/home/exedy|/data/home|g' \
    systemd/cashflow-bot.service > /tmp/cashflow-bot.service
grep -E "User=|WorkingDirectory=|ExecStart=" /tmp/cashflow-bot.service
```

Tiga baris itu harus menunjukkan `User=user`,
`WorkingDirectory=/data/home/project/finance_assistant`, dan
`ExecStart=/data/home/project/finance_assistant/venv/bin/python -m
src.bot`. Kalau sudah benar:

```bash
sudo cp /tmp/cashflow-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable cashflow-bot.service
sudo systemctl start cashflow-bot.service
sudo systemctl status cashflow-bot.service
```

`enable` adalah yang membuat bot ikut nyala setiap mesin start.
`status` harus `active (running)`. Bukti final: `/saldo` dari
Telegram dibalas oleh instance service (bukan manual).

### 10. Backup otomatis

```bash
crontab -e
```

Tambahkan (menjalankan `scripts/backup_db.sh` tiap jam 3 pagi):

```
0 3 * * * /data/home/project/finance_assistant/scripts/backup_db.sh
```

Retensi: harian 7, mingguan 4, bulanan 6. Script membaca kredensial
dari `config/.env` tanpa mengeksekusinya. Verifikasi dengan
`crontab -l`.

### 11. Snapshot

Setelah semua hijau, simpan snapshot dari dashboard instabox
(tombol Save snapshot). Snapshot mencakup `/data/home` — termasuk
`.env` — jadi jangan bagikan snapshot ke siapa pun.

### 12. Bot kedua (advisor)

1. BotFather → `/newbot` → token → `ADVISOR_BOT_TOKEN` di `.env`.
2. User database read-only (sebagai superuser postgres, karena
   `finance_admin` tidak punya CREATEROLE):

```sql
CREATE USER finance_reader WITH PASSWORD 'PASSWORD-BARU';
GRANT CONNECT ON DATABASE finance_assistant TO finance_reader;
GRANT USAGE ON SCHEMA public TO finance_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO finance_reader;
```

Opsional (sebagai `finance_admin`, agar tabel baru di masa depan
otomatis ikut):

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO finance_reader;
```

3. Isi `DB_READER_USER` dan `DB_READER_PASS` di `.env`.
4. Tes manual: `python -m advisor.bot` → log `Advisor bot aktif
   (model=...)` → `/start` → tanya `pengeluaran bulan ini berapa`.
   Database kosong menjawab nol — itu normal, catat dulu transaksi
   lewat recorder baru tanya lagi.
5. `Ctrl+C` yang manual, lalu pasang service:

```bash
sudo cp systemd/advisor-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable advisor-bot.service
sudo systemctl start advisor-bot.service
sudo systemctl status advisor-bot.service
```

## Operasional harian

```bash
sudo systemctl status cashflow-bot.service
sudo systemctl status advisor-bot.service
sudo systemctl restart cashflow-bot.service
sudo journalctl -u cashflow-bot.service -n 30
sudo journalctl -u advisor-bot.service -n 30
```

Catatan: `journalctl` tanpa `sudo` menampilkan `No entries` karena
user biasa tidak bisa membaca log service sistem — selalu pakai
`sudo`. Log aplikasi juga ada di `logs/` di folder project
(rotasi 5 MB × 5).

Perilaku pause/resume:

- **Pause** (30 menit idle / 8 jam jalan): proses dibekukan, bukan
  dimatikan. Saat resume, bot lanjut sendiri tanpa start ulang.
- **Stop/start penuh atau reboot**: systemd menjalankan service yang
  sudah `enable` secara otomatis. Inilah yang dimaksud "auto hidup
  pas sistem jalan".
- Update kode: `git pull` lalu `sudo systemctl restart
  <nama-service>`.

## Troubleshooting

| Gejala | Penyebab & perbaikan |
|---|---|
| `409 Conflict` | Dua instance bot jalan (manual + service). Matikan yang manual (`pkill -f "src.bot"` atau `Ctrl+C`). |
| `203/EXEC`, service exit-code | `venv/bin/python` tidak ada di jalur `ExecStart`. Buat ulang venv di folder project (`python3 -m venv venv` + `pip install -r requirements.txt`), lalu `systemctl restart`. |
| `Failed to reload daemon: Access denied` | `systemctl daemon-reload` tanpa `sudo`. Semua perintah `systemctl` pakai `sudo`. |
| Bot jawab `Config kurang` | Ada kunci `.env` yang kosong. Lengkapi, lalu restart service. |
| `Only roles with the CREATEROLE attribute may create roles` | Sesi psql bukan superuser. Buat role lewat `sudo -u postgres psql`. |
| `journalctl` kosong (`No entries`) | Butuh `sudo`. |
| Bot tidak membalas setelah resume | Cek `systemctl status` — kalau service `active`, tunggu antrean Telegram terkirim. Kalau `failed`, lihat `journalctl`. |
| `git pull` minta token lagi | PAT tidak tersimpan. Pakai SSH key (lihat langkah 3) atau simpan PAT di credential helper + password manager. |

## Keamanan (jangan dilanggar)

- Jangan pernah commit `config/.env` atau file berisi token/password.
- Jangan kirim token, password, atau PAT lewat chat — tulis langsung
  ke `.env` / password manager / GitHub Settings.
- Token yang pernah ditempel di chat dianggap bocor dan harus
  direvoke + rotasi.
- Snapshot instabox mengandung `.env` — jangan dibagikan.
- Advisor hanya punya hak SELECT — secara teknis tidak bisa merusak
  data, lapis pengaman ini jangan dicabut.

# PostgreSQL — Knowledge Project Finance Assistant

Kumpulan perintah PostgreSQL yang dipakai project ini, fungsinya, dan
jebakan yang pernah ketemu. Ditulis untuk pemakai yang masih belajar
SQL — semua contoh bisa dicopy langsung.

Lihat juga: [[install-instabox]] (instalasi lengkap di VPS),
[[flow-diagram]] (alur data antar tabel).

## Gambaran

- **Server**: PostgreSQL 17 di mesin yang sama dengan bot.
- **Database**: `finance_assistant` (timezone `Asia/Jakarta`).
- **User aplikasi**: `finance_admin` — pemilik semua tabel, dipakai
  recorder untuk baca + tulis.
- **User advisor**: `finance_reader` — hanya `SELECT`, dipakai advisor.
  Sengaja dipisah supaya bot tanya-jawab secara teknis tidak bisa
  merusak data.
- **Tabel**: 8 tabel domain + 1 tabel audit yang dibuat bot otomatis.

## Masuk ke psql

```bash
psql -h localhost -U finance_admin -d finance_assistant
```

Keluar: `\q`. Di dalam psql, prompt-nya `finance_assistant=#`.
Perintah psql diawali backslash (bukan SQL) dan **tidak** perlu
titik koma. Perintah SQL perlu titik koma di akhir.

Perintah psql yang sering dipakai:

| Perintah | Fungsi |
|---|---|
| `\l` | Daftar database |
| `\dt` | Daftar tabel |
| `\d wallets` | Struktur tabel `wallets` (kolom + tipe) |
| `\d+ cashflow` | Struktur + detail tabel |
| `\c finance_assistant` | Pindah database |
| `\q` | Keluar |

Jebakan pemula (pernah kejadian): menempel `psql DROP TABLE ...` di
dalam prompt psql. `psql` adalah perintah shell, bukan perintah psql.
Di dalam psql, tulis SQL-nya saja: `DROP TABLE ...;`.

## Skema tabel

| Tabel | Isi | Kolom kunci |
|---|---|---|
| `wallets` | 12 dompet | `id`, `wallets_name`, `current_balance` |
| `savings` | Tabungan (masih kosong, daftar belum didefinisikan) | `id`, `saving_name`, `current_balance` |
| `categories` | 57 kategori + subkategori (seed) | `category_name`, `sub_category_name` |
| `income` | Pemasukan | `date`, `source`, `amount`, `destination_wallets` |
| `cashflow` | Pengeluaran | `date`, `category`, `item`, `amount`, `wallets_id` |
| `moneyflow` | Transfer antar dompet/tabungan sendiri | `source_type`, `source_id`, `destination_type`, `destination_id`, `amount` |
| `debt` | Hutang & piutang | `person_name`, `type` (`debt`/`receivable`), `original_amount`, `remaining_amount`, `due_date`, `status` |
| `adjustment` | Koreksi saldo absolut | `wallets_id`, `amount`, `adjustment_type` (`initial`/`correction`) |
| `audit_log` | Jejak audit semua aksi bot (dibuat otomatis saat bot start) | `user_id`, `action`, `kind`, `row_id`, `detail` (JSONB) |

Aturan data yang penting:

- Semua tanggal transaksi memakai WIB (`today_wib()` di kode, timezone
  database `Asia/Jakarta`).
- `amount` bertipe `NUMERIC(15, 2)` — desimal, bukan float, supaya
  tidak ada galat pembulatan uang.
- `cashflow.wallets_id` bisa NULL: itu artinya pengeluaran dari
  tabungan (skema tidak punya kolom sumber tabungan di `cashflow`).
- `debt.status`: `unpaid`, `partial`, `paid`. Pembayaran hutang
  memakai FIFO (hutang tertua dilunasi dulu).
- `adjustment` hanya untuk dompet, bukan tabungan, dan menyimpan
  saldo absolut (delta dihitung dari selisih).

## DDL — membuat dan mengubah struktur

Dipakai saat instalasi dan perbaikan awal:

```sql
-- Database + timezone
CREATE DATABASE finance_assistant OWNER finance_admin;
ALTER DATABASE finance_assistant SET timezone TO 'Asia/Jakarta';

-- User aplikasi dan user read-only
CREATE USER finance_admin WITH PASSWORD 'PASSWORD-KUAT';
CREATE USER finance_reader WITH PASSWORD 'PASSWORD-BARU';

-- Hak advisor: konek + baca semua tabel yang ada
GRANT CONNECT ON DATABASE finance_assistant TO finance_reader;
GRANT USAGE ON SCHEMA public TO finance_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO finance_reader;

-- Hak advisor untuk tabel yang dibuat di masa depan
-- (dijalankan sebagai finance_admin, pemilik tabel)
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT ON TABLES TO finance_reader;
```

Perbaikan kolom bersejarah (database live pernah diperbaiki dari
nama kolom yang salah — jangan diulang, ini arsip):

```sql
ALTER TABLE income RENAME COLUMN ammount TO amount;
ALTER TABLE savings RENAME COLUMN current_ballance TO current_balance;
ALTER TABLE adjustment RENAME COLUMN wallet_id TO wallets_id;
```

Perintah `CREATE USER` butuh hak CREATEROLE. `finance_admin` tidak
punya hak itu, jadi pembuatan role harus lewat superuser:

```bash
sudo -u postgres psql -d finance_assistant
```

Errornya kalau salah user: `Only roles with the CREATEROLE
attribute may create roles`.

## DML — perintah yang dijalankan bot

### Recorder: tulis transaksi

```sql
-- Pengeluaran baru (kategori + dompet + tanggal WIB)
INSERT INTO cashflow (date, category, item, amount, wallets_id)
VALUES ('2026-10-29', 'transport', 'bensin', 100000, 1);

-- Pemasukan baru
INSERT INTO income (date, source, amount, destination_wallets)
VALUES ('2026-10-29', 'gaji', 5000000, 1);

-- Saldo dompet bergerak (bump_balance): tambah/kurangi dari saldo lama
UPDATE wallets SET current_balance = current_balance + 100000
WHERE id = 1;

-- Transfer antar dompet sendiri (dicatat di moneyflow + 2x bump_balance)
INSERT INTO moneyflow (date, source_type, source_id,
  destination_type, destination_id, amount)
VALUES ('2026-10-29', 'wallet', 1, 'wallet', 2, 500000);

-- Hutang baru / pembayaran (FIFO di kode, bukan di SQL)
INSERT INTO debt (person_name, type, original_amount,
  remaining_amount, due_date, status)
VALUES ('Mila', 'debt', 500000, 500000, '2026-11-15', 'unpaid');

-- Undo (/batal): hapus baris transaksi secara permanen (hard-delete)
DELETE FROM cashflow WHERE id = 42;
```

Setiap aksi tulis selalu dibarengi satu baris audit dalam **transaksi
yang sama** (atomik — kalau satu gagal, keduanya batal):

```sql
INSERT INTO audit_log (user_id, action, kind, row_id, detail)
VALUES (123456, 'create', 'cashflow', 42,
  '{"moves": [{"wallet_id": 1, "delta": -100000}]}');
```

### Recorder: baca (perintah /saldo, /riwayat, /utang, /audit)

```sql
-- /saldo: semua dompet + tabungan
SELECT wallets_name, current_balance FROM wallets ORDER BY id;
SELECT saving_name, current_balance FROM savings ORDER BY id;

-- /riwayat: 3 query terpisah digabung di Python (UNION pernah gagal)
SELECT date, category, item, amount FROM cashflow
ORDER BY date DESC, id DESC LIMIT 10;
SELECT date, source, amount FROM income
ORDER BY date DESC, id DESC LIMIT 10;
SELECT date, person_name, type, amount FROM debt
ORDER BY date DESC, id DESC LIMIT 10;

-- /utang: yang belum lunas saja
SELECT person_name, type, remaining_amount, due_date FROM debt
WHERE status <> 'paid' ORDER BY due_date NULLS LAST, id;

-- /audit: jejak terakhir, created_at ditampilkan dalam WIB
SELECT created_at, action, kind, label FROM audit_log
ORDER BY id DESC LIMIT 10;
```

### Advisor: baca saja (semua query SELECT)

Pola yang sama dipakai semua intent advisor. `start`/`end` adalah
batas periode WIB (end eksklusif — tanggal akhir tidak ikut):

```sql
-- Total pengeluaran periode (boleh + filter dompet/kategori)
SELECT COALESCE(SUM(amount), 0) FROM cashflow
WHERE date >= '2026-10-01' AND date < '2026-11-01';
-- + AND wallets_id = 1
-- + AND category = 'groceries'

-- Breakdown per kategori, terbesar dulu
SELECT category, COALESCE(SUM(amount), 0) FROM cashflow
WHERE date >= '2026-10-01' AND date < '2026-11-01'
GROUP BY category ORDER BY 2 DESC;

-- Total pemasukan periode
SELECT COALESCE(SUM(amount), 0) FROM income
WHERE date >= '2026-10-01' AND date < '2026-11-01';

-- Pengeluaran terbesar (N = limit, default 5)
SELECT item, amount, category, date FROM cashflow
WHERE date >= '2026-10-01' AND date < '2026-11-01'
ORDER BY amount DESC LIMIT 5;

-- Hutang/piutang terbuka (sama seperti /utang recorder)
SELECT person_name, type, remaining_amount, due_date FROM debt
WHERE status <> 'paid' ORDER BY due_date NULLS LAST, id;
```

`COALESCE(SUM(...), 0)` penting: `SUM` atas nol baris menghasilkan
NULL, bukan 0. `NULLS LAST` penting: hutang tanpa jatuh tempo
(`due_date` NULL) tampil di bawah, bukan di atas.

### Maintenance

```sql
-- Seed ulang dompet (setelah perbaiki typo seed)
TRUNCATE wallets RESTART IDENTITY CASCADE;

-- Hapus total dan buat ulang (dipakai saat password placeholder
-- tidak sengaja terpakai — urutan penting: database dulu, baru user)
DROP DATABASE finance_assistant;
DROP USER finance_admin;

-- Cek isi tabel secara cepat
SELECT COUNT(*) FROM cashflow;
SELECT * FROM wallets ORDER BY id;
```

`TRUNCATE ... CASCADE` menghapus juga baris di tabel lain yang
mereferensi dompet (mis. `cashflow.wallets_id`). Jangan dipakai di
database berisi data asli kecuali memang mau reset total.

## Backup

Script `scripts/backup_db.sh` menjalankan `pg_dump` lewat cron tiap
jam 3 pagi, membaca kredensial dari `config/.env` tanpa
mengeksekusinya. Retensi: harian 7, mingguan 4, bulanan 6.

Backup manual:

```bash
pg_dump -h localhost -U finance_admin -d finance_assistant > backup.sql
```

Restore manual (database kosong):

```bash
psql -h localhost -U finance_admin -d finance_assistant -f backup.sql
```

## Aturan main (jangan dilanggar)

1. **SELECT dulu sebelum UPDATE/DELETE.** Tidak ada `UPDATE` atau
   `DELETE` tanpa `WHERE` — satu perintah ceroboh bisa mengubah
   semua baris.
2. **Jangan edit saldo langsung.** Saldo hanya bergerak lewat
   `bump_balance` (transaksi) atau `adjustment` (koreksi absolut).
   Edit `current_balance` manual membuat audit tidak cocok dengan
   data.
3. **Jangan hapus baris audit.** `audit_log` adalah jejak — undo
   bot menghapus transaksi, bukan auditnya.
4. **Tanggal selalu WIB.** Filter tanggal di query harus pakai
   rentang WIB; database memang disetel `Asia/Jakarta`, tapi
   pengirim query dari zona lain tetap harus sadar zona.
5. **Advisor tidak boleh INSERT/UPDATE/DELETE.** Kalau ada kebutuhan
   tulis dari advisor, itu desain yang salah — tulis hanya lewat
   recorder.
6. **Kredensial tidak pernah masuk file SQL yang di-commit.**
   Password hanya di `.env` dan di kepala pemilik.

## Error yang pernah ketemu

| Error | Artinya & perbaikan |
|---|---|
| `Only roles with the CREATEROLE attribute may create roles` | Bukan superuser. Buat role lewat `sudo -u postgres psql`. |
| `syntax error` setelah menempel `psql ...` di prompt psql | `psql` itu perintah shell. Di dalam psql tulis SQL-nya saja. |
| `relation "cashflow" does not exist` | Skema belum dimuat, atau salah database (`\l` dan `\c` untuk cek). |
| `permission denied for table ...` | User tidak punya GRANT. Cek `GRANT SELECT` untuk reader, atau login sebagai owner. |
| `could not connect ... Connection refused` | Service postgres mati. Cek `sudo systemctl status postgresql`. |
| `password authentication failed` | Password salah / password di `.env` tidak cocok dengan yang di database. |
| `database "finance_assistant" is being accessed by other users` | Ada koneksi aktif (bot jalan). Matikan bot dulu sebelum `DROP DATABASE`. |

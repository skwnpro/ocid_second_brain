---
title: 9Router + Hermes Setup finance-bot
date: 2026-11-09
---

# 9Router + Hermes Setup finance-bot

Rangkuman setup 9Router + pindahin Hermes Agent ke 9Router di VPS instabox `finance-bot`, dikerjakan 2026-11-09. Ditulis seolah dikerjakan sendiri tanpa bantuan AI — semua dependency, konfigurasi, dan kemungkinan error ada di sini.

Mesin yang dipakai: VPS instabox `finance-bot` yang sama dengan [[install-instabox]] — Debian 13, 4 vCPU / 8 GB RAM, home `/data/home`.

## Arsitektur

Hermes gateway (Telegram) → `http://localhost:20128/v1` (9Router, OpenAI-compatible) → Gemini API.

- Hermes ngomong ke 9Router pakai API key 9Router khusus (`hermes`), bukan kunci Gemini langsung.
- 9Router yang megang kunci Gemini (diambil dari `config/.env` finance bot) dan nerusin request ke Google.
- 9Router cuma dengerin `127.0.0.1:20128` — dari luar VPS nggak bisa diakses langsung. Dashboard-nya juga localhost-only.
- Bot finance (`cashflow-bot`, `advisor-bot`) tetap jalan sendiri sebagai service systemd, nggak lewat 9Router.

## Dependency

- VPS `finance-bot` jalan, user `user`, Node **22.23.3**, npm **10.9.9**
- Hermes Agent **0.23.0** udah kepasang (`~/.local/bin/hermes`), gateway jalan, Telegram connected
- Kunci Gemini yang udah ada di VPS (`/data/home/project/finance_assistant/config/.env`)
- Akun instabox + CLI `instabox` (buat SSH/exec dari laptop sendiri)
- RAM: 9Router makan ~100 MB, Hermes gateway ~250 MB — masih longgar di 8 GB

## 0. Cek yang udah ada

```bash
instabox ls
instabox exec finance-bot -- 'hermes --version; node --version; npm --version'
instabox exec finance-bot -- 'grep -n "^  default:\|^  provider:\|^  base_url:" ~/.hermes/config.yaml'
instabox exec finance-bot -- 'sudo systemctl status cashflow-bot.service advisor-bot.service --no-pager | grep -E "●|Active:"'
```

Harusnya keluar: Hermes 0.23.0, Node 22, config masih `default: "upstage/solar-mini4:free"`, `provider: "nous"`, `base_url: "https://inference-api.nousresearch.com/v1"`, dan dua bot finance `active (running)`.

## 1. Install CLI instabox di laptop sendiri (kalau belum ada)

```bash
curl -fsSL https://instabox.sh/install.sh | sh
instabox login
instabox whoami
instabox ls
```

`login` buka halaman device code di browser, masukin kodenya. `whoami` harus nunjukin email sendiri. Installer naruh binary di `/usr/local/bin` kalau bisa ditulis, kalau nggak di `~/.local/bin` — dua-duanya awet di laptop sendiri. (Catatan: di VM yang sering ke-replace, binary di `/usr/local/bin` bisa hilang — install ulang pakai perintah yang sama, login-nya tetap kesimpen di `~/.config/instabox/`.)

## 2. Install 9Router di VPS

Install global pakai npm. Install sebagai user biasa gagal `EACCES`, harus pakai `sudo`:

```bash
instabox exec finance-bot -- 'sudo npm install -g 9router'
instabox exec finance-bot -- '9router --version; which 9router'
```

Versi yang terpasang: **0.5.116**, binary di `/usr/local/bin/9router`.

## 3. Jalanin 9Router sebagai service systemd — jangan jalanin manual

Jalanin manual (`9router &`) itu mati pas VPS ke-pause/restart dan nggak ada yang nyalain lagi. Bikin unit systemd:

```bash
sudo tee /etc/systemd/system/9router.service > /dev/null <<'EOF'
[Unit]
Description=9Router LLM router (localhost only)
After=network-online.target
Wants=network-online.target

[Service]
User=user
WorkingDirectory=/data/home
ExecStart=/usr/local/bin/9router -H 127.0.0.1 -p 20128 --no-browser --skip-update
Restart=always

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now 9router.service
sudo systemctl status 9router.service --no-pager | head -n 5
```

Kenapa `-H 127.0.0.1`: default 9Router itu bind `0.0.0.0` (terbuka ke luar) — jangan. Localhost-only, dashboard cuma bisa dibuka lewat tunnel (langkah 10).

Verifikasi:

```bash
curl -s -m 5 http://127.0.0.1:20128/v1/models | python3 -c "import json,sys; d=json.load(sys.stdin); print(len(d['data']), 'models')"
```

Harusnya keluar `864 models`. Itu katalog doang — belum bisa dipakai sebelum ada provider yang nyambung (langkah 4).

## 4. Sambungin Gemini sebagai upstream

Dashboard 9Router login pakai password default — harus diganti dulu sebelum bisa dipakai dari luar (langkah 9). Provider ditambah lewat API dashboard, dari dalam VPS (localhost):

```bash
# login dulu, simpan cookie
curl -s -c /tmp/nr-cookies.txt -H 'Content-Type: application/json' \
  -d '{"password": "<password default>"}' http://127.0.0.1:20128/api/auth/login

# tambah provider Gemini (kunci dibaca dari config/.env, jangan di-print)
curl -s -b /tmp/nr-cookies.txt -H 'Content-Type: application/json' \
  -d '{"provider": "gemini", "apiKey": "<kunci dari config/.env>", "name": "Gemini", "defaultModel": "gemini-2.5-flash"}' \
  http://127.0.0.1:20128/api/providers
```

Kunci diambil dari `/data/home/project/finance_assistant/config/.env` di VPS — **jangan pernah di-print di chat, log, atau file**. Paling aman: pakai script di VPS yang baca file `.env` itu pas jalan, terus hapus script-nya sesudah selesai.

Hasil: connection id `7138c563-23d7-4b5f-abab-2ddff718f32b`, provider `gemini`, defaultModel `gemini-2.5-flash`.

Verifikasi kunci + koneksi:

```bash
# langsung ke Google, harus 200 dan nampilin daftar model
curl -s "https://generativelanguage.googleapis.com/v1beta/models?key=<kunci>" | head -c 300

# lewat 9Router, harus 200 dan nampilin model Google beneran
curl -s -b /tmp/nr-cookies.txt http://127.0.0.1:20128/api/providers/7138c563-23d7-4b5f-abab-2ddff718f32b/models | head -c 300
```

Kalau dua-duanya 200, kunci dan koneksinya valid. Kunci yang dipakai panjangnya 53 karakter, bukan diawali `AIza`.

## 5. Bikin API key 9Router buat Hermes

Chat ke `/v1/chat/completions` itu butuh API key 9Router yang asli:

```bash
curl -s -b /tmp/nr-cookies.txt -H 'Content-Type: application/json' \
  -d '{"name": "hermes"}' http://127.0.0.1:20128/api/keys
```

Hasil: key id `8450ed84-88a7-453c-bda5-169535688e44`, prefix `sk-544cd`. Key lengkapnya kesimpen di `~/.9router/db/data.sqlite` tabel `apiKeys` — baca dari situ kalau butuh, jangan di-print.

Tanpa Bearer → `Missing API key`. Bearer asal-asalan → `Invalid API key`.

## 6. Tes chat completion — jebakan model Gemini

```bash
curl -s http://127.0.0.1:20128/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer <key hermes>" \
  -d '{"model": "gemini/gemini-3.6-flash", "messages": [{"role": "user", "content": "jawab dengan satu angka saja: 2+2"}], "max_tokens": 20}'
```

Yang kejadian pas nyoba model satu-satu:

- `gemini/gemini-2.5-flash` → **503**, Google 404: model ini udah nggak dikasih ke user baru, disuruh pakai `gemini-3.8-flash`.
- `gemini/gemini-3.8-flash` → timeout (90 detik, nggak balik).
- `gemini/gemini-3.6-flash` → **200, jawab "4"**. Ini yang dipakai.

Catatan: prompt kecil kayak `2+2` kehitung 2011 prompt token di 9Router — dia nambahin system prompt sendiri. Nggak masalah buat pemakaian biasa, tapi jangan kaget lihat angkanya.

## 7. Pindahin Hermes ke 9Router

Backup dulu:

```bash
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.bak-before-9router
```

Edit blok `model:` di `~/.hermes/config.yaml` (baris 81, 113, 117):

```yaml
model:
  default: "gemini/gemini-3.6-flash"
  provider: "custom"
  base_url: "http://localhost:20128/v1"
  key_env: "NINE_ROUTER_API_KEY"
```

Settingan lama (buat rollback): `default: "upstage/solar-mini4:free"`, `provider: "nous"`, `base_url: "https://inference-api.nousresearch.com/v1"`.

Terus isi key-nya di `~/.hermes/.env` (nilai = key `hermes` dari DB 9Router, baca pakai script, jangan di-print):

```bash
NINE_ROUTER_API_KEY=<key dari DB 9Router>
OPENAI_API_KEY=<key yang sama>
```

Tes dari CLI (tanpa Telegram):

```bash
hermes -z "jawab dengan satu angka saja: 2+2, tanpa penjelasan"
```

Harusnya keluar `4`.

Terus restart gateway biar bot Telegram-nya pakai provider baru:

```bash
hermes gateway restart
```

Verifikasi di log:

```bash
grep "Model context warmed" ~/.hermes/logs/gateway.log | tail -n 2
cat ~/.hermes/gateway_state.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('gateway_state'), d.get('platforms',{}).get('telegram'))"
```

Harusnya keluar `Model context warmed: gemini/gemini-3.6-flash -> 1048576 tokens` dan Telegram `connected`. Terakhir, tes beneran dari Telegram — kirim apa aja ke bot Hermes.

## 8. Gateway dijadiin systemd service (insiden 13:20)

Hari yang sama, jam ~13:20, chat ke bot nggak dibalas. Penyebabnya: VPS ke-pause sendiri (free plan: pause abis 30 menit idle — polling Telegram ternyata nggak dihitung aktivitas), dan pas bangun lagi gateway Hermes udah mati karena waktu itu dia jalan sebagai proses biasa, bukan service. 9Router + bot finance selamat karena mereka service systemd.

Perintah restart-nya (`hermes gateway restart`) waktu itu juga gantung — tapi restart-nya sendiri udah jalan, cuma CLI-nya yang nggak balik. Jangan panik, cek `gateway_state.json` + `ps` dulu sebelum ngulang perintah.

Fix-nya:

```bash
sudo HERMES_HOME=/data/home/.hermes /data/home/.local/bin/hermes gateway install --system
sudo systemctl status hermes-gateway.service --no-pager | head -n 6
```

Harusnya `hermes-gateway.service` jadi `enabled` + `active (running)`. Sekarang gateway nyala sendiri tiap mesin bangun, kayak bot finance.

Catatan penting: gateway nge-drop update Telegram yang ngantri selama dia offline (`drop_pending_on_cold_boot: true`) — chat yang dikirim pas gateway mati itu hilang, harus dikirim ulang.

## 9. Ganti password dashboard 9Router

Password default ditolak buat login dari luar: 403 `mustChangePassword` ("Default password must be changed before remote access"). Gantinya harus dari localhost — login dari dalam VPS dulu, baru PATCH:

```bash
# dari VPS: login dulu, simpan cookie
curl -s -c /tmp/nr-cookies.txt -H 'Content-Type: application/json' \
  -d '{"password": "<password default>"}' http://127.0.0.1:20128/api/auth/login

# ganti password
curl -s -b /tmp/nr-cookies.txt -X PATCH -H 'Content-Type: application/json' \
  -d '{"currentPassword": "<password default>", "newPassword": "<password baru>"}' \
  http://127.0.0.1:20128/api/settings

# tes: login pakai password baru → 200, mustChangePassword false
curl -s -H 'Content-Type: application/json' \
  -d '{"password": "<password baru>"}' http://127.0.0.1:20128/api/auth/login
```

PATCH tanpa cookie login bakal 401 `Unauthorized` — jadi urutannya jangan kebalik. Password baru jangan ditulis di file ini, catat di tempat aman.

Status 2026-11-09: password udah diganti, login dari luar jalan.

## 10. Buka dashboard dari luar (opsional)

Dashboard cuma dengerin localhost, jadi dari laptop sendiri ada dua cara:

**Cara 1 — port-forward (paling bersih):**

```bash
instabox port-forward finance-bot 20128:20128
```

Biarin terminal kebuka, terus buka `http://localhost:20128` di browser. Butuh terminal interaktif (TTY) — dari VM tanpa TTY, koneksi SSH-nya langsung ketutup (`Connection closed by ... port 3128`).

**Cara 2 — publish private (buat sekali lihat):**

```bash
instabox port publish finance-bot 20128 --private
instabox port ls finance-bot
instabox port unpublish finance-bot 20128
```

Dapat URL token-gated, buka di browser, login pakai password dashboard. Begitu selesai, langsung unpublish — jangan dibiarin ngegantung. URL yang dipakai 2026-11-09 udah di-unpublish (cek: `port ls` kosong, URL-nya 404).

## 11. Pakai Hermes-nya

Chat biasa dari Telegram, gateway sekarang jawab pakai Gemini 3.6-flash lewat 9Router. Kalau Gemini/9Router down, fallback ke Nous solar-mini4:free belum dipasang — pasangnya dari terminal sendiri (interaktif, nggak bisa di-script):

```bash
hermes fallback add
```

Terus pilih Nous / `upstage/solar-mini4:free`.

## Kemungkinan error dan cara benerinnya

- **`npm install -g 9router` EACCES:** install sebagai user biasa gagal nulis ke folder global. Pakai `sudo npm install -g 9router`.
- **`/v1/chat/completions` → `Missing API key`:** request nggak bawa Bearer. Bikin key di langkah 5.
- **`/v1/chat/completions` → `Invalid API key`:** Bearer-nya asal-asalan (misal `test`). Harus key `hermes` yang asli dari DB 9Router.
- **`gemini-2.5-flash` → 503 "no longer available to new users":** Google udah nggak kasih model ini ke user baru. Pindah ke `gemini-3.6-flash` (langkah 6). Jangan pakai 2.5-flash di config mana pun.
- **`gemini-3.8-flash` timeout:** sempat timeout di percobaan pertama. Pakai 3.6-flash yang udah terbukti jalan.
- **Chat balik kosong padahal status 200:** cek SSE mentahnya — kemungkinan `max_tokens` kekecilan kemakan reasoning, atau parser stream-nya salah baca delta. Tes ulang pakai prompt `2+2` dan print baris SSE-nya.
- **Bot Telegram nggak respon abis VPS ke-pause:** gateway mati (bukan service). Cek `hermes gateway status` — kalau bilang "recorded process is gone", install service di langkah 8, terus kirim ulang chat-nya (pesan lama ke-drop).
- **Dashboard login 403 `mustChangePassword`:** password masih default. Ganti dari localhost di langkah 9.
- **PATCH `/api/settings` 401 `Unauthorized`:** belum login / nggak bawa cookie. Login dulu dari localhost, baru PATCH.
- **Login dashboard gagal terus, "attempt(s) left before lockout":** tiap gagal login ngurangin jatah, jangan brute force. Tunggu / login dari localhost.
- **`instabox ssh` / `port-forward` langsung ketutup dari VM tanpa TTY:** itu keterbatasan terminal non-interaktif, bukan salah perintah. Pakai terminal sendiri, atau `port publish` (langkah 10).
- **CLI `instabox` hilang (`command not found`):** binary di `/usr/local/bin` ikut kehapus pas VM ke-replace. Install ulang pakai `install.sh`, login-nya tetap kesimpen di `~/.config/instabox/`. Buat awet, install ke folder yang persistent (`INSTABOX_INSTALL_DIR=...`).
- **Bot finance ikut mati?** Nggak — `cashflow-bot` dan `advisor-bot` itu service systemd sendiri, nggak lewat 9Router dan nggak keganggu sama perubahan Hermes.

## Catatan lain

- Kunci Gemini yang sama dipakai finance bot dengan `GEMINI_MODEL=gemini-3.5-flash` — model itu nggak muncul di daftar model Google, jadi panggilan Gemini bot finance kemungkinan gagal. Worth dicek terpisah.
- Akses agent (Muse) ke instabox pakai agent login yang key-nya expired 2026-11-16 — kalau mau dibantu lagi abis tanggal itu, approve ulang login-nya.
- Free plan instabox: pause abis 30 menit idle; mesin ini juga nunjukin batas jalan 120 menit per run (`maxRunMinutes`). `instabox exec` dan polling Telegram ternyata nggak dihitung aktivitas, jadi jangan ngandelin itu buat nahan mesin tetap nyala. Bangunin pakai `instabox resume finance-bot` (proses ikut ke-resume), atau tombol Resume di dashboard instabox.

## Checklist akhir, sebelum mulai kerja

- [ ] `sudo systemctl status 9router.service` → `active (running)`, `curl http://127.0.0.1:20128/v1/models` → 200
- [ ] Provider `gemini` ada di dashboard, `GET /api/providers/{id}/models` → 200
- [ ] `hermes -z "jawab dengan satu angka saja: 2+2"` → `4`
- [ ] `grep "Model context warmed" ~/.hermes/logs/gateway.log` → `gemini/gemini-3.6-flash -> 1048576 tokens`
- [ ] `sudo systemctl status hermes-gateway.service` → `enabled` + `active (running)`
- [ ] Telegram connected, tes chat dari Telegram dibalas
- [ ] Password dashboard udah bukan password default
- [ ] `sudo systemctl status cashflow-bot.service advisor-bot.service` → dua-duanya `active (running)`

## Terkait

- [[hermes-agent-install-setup]] — instalasi awal Hermes Agent (Nous Portal Free, solar-mini4:free, gateway manual) sebelum pindah ke 9Router; catatan: gateway sekarang udah jadi systemd service (langkah 8), bukan manual lagi
- [[install-instabox]] — VPS instabox `finance-bot` tempat semua ini jalan
- [[finance_assistant_installation_guide]] — bot finance yang satu mesin dengan Hermes

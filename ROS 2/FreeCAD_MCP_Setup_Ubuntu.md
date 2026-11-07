---
title: FreeCAD + MCP Setup Ubuntu
date: 2026-11-07
---

# FreeCAD + MCP Setup Ubuntu

Rangkuman setup FreeCAD + MCP di mini PC (MSI Cubi 5, ground station, Ubuntu Desktop 24.04), dikerjakan 2026-11-07. Ditulis seolah dikerjakan sendiri tanpa bantuan AI — semua dependency, konfigurasi, dan kemungkinan error ada di sini.

Mesin yang dipakai: mini PC ground station yang sama dengan [[ROS2_Bruskart_Robot_Project_Overview]].

## Arsitektur

Antigravity (MCP client) → `uvx freecad-mcp` (MCP server, stdio) → XML-RPC `127.0.0.1:9875` → addon FreeCADMCP di dalam FreeCAD (GUI).

Dua sisi harus hidup barengan: FreeCAD kebuka + RPC Server jalan, dan Antigravity kebuka + MCP `freecad` Connected. Satu mati, satu lagi error.

## Dependency

- Ubuntu 24.04, user `exedy`
- FreeCAD **1.1.4** (snap, bukan apt)
- uv **0.13.7** (buat jalanin `freecad-mcp` dari PyPI)
- git, curl, tar, desktop-file-utils, python3
- Antigravity App **44.5.1**
- Akun Google Gemini Plus (buat login Antigravity)

## 0. Cek yang udah ada

```bash
snap info freecad
freecad.cmd --version
which git curl uv uvx
ls ~/.local/bin/uvx
```

Harusnya keluar FreeCAD 1.1.4. Kalau `uv` / `uvx` belum ada, lanjut ke langkah 2.

## 1. Install FreeCAD 1.1 — jangan pakai apt

Apt Ubuntu 24.04 itu FreeCAD 0.21, addon MCP-nya nggak cocok. Pakai snap:

```bash
sudo snap install freecad
snap info freecad
freecad.cmd --version
```

Cek `tracking: latest/stable` dan `installed: 1.1-...`. Jalankan FreeCAD sekali dari app menu, biar folder profilnya kebentuk.

## 2. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env
~/.local/bin/uvx --version
```

Catat path absolutnya: `/home/exedy/.local/bin/uvx`. GUI kayak Antigravity nggak baca PATH shell, jadi nanti di config MCP harus pakai path absolut ini, bukan cuma `uvx`.

## 3. Clone repo MCP-nya

Repo: neka-nat/freecad-mcp.

```bash
git clone https://github.com/neka-nat/freecad-mcp.git ~/freecad-mcp
ls ~/freecad-mcp/addon/FreeCADMCP
```

Harusnya ada `InitGui.py`, `Init.py`, `package.xml`, dan folder `rpc_server/`.

## 4. Copy addon ke folder Mod — ini langkah yang paling gampang salah

Untuk FreeCAD snap, folder Mod-nya **bukan** `~/.local/share/FreeCAD/Mod/`. Itu folder FreeCAD non-snap.

Folder yang bener buat snap:

```bash
mkdir -p ~/snap/freecad/common/Mod
cp -r ~/freecad-mcp/addon/FreeCADMCP ~/snap/freecad/common/Mod/
ls ~/snap/freecad/common/Mod/
```

Harusnya keluar `FreeCADMCP` (plus `manifest.json` yang udah ada).

Buat jaga-jaga, copy juga ke folder non-snap:

```bash
mkdir -p ~/.local/share/FreeCAD/Mod
cp -r ~/freecad-mcp/addon/FreeCADMCP ~/.local/share/FreeCAD/Mod/
```

Verifikasi path yang dipakai FreeCAD beneran:

```bash
freecad.cmd -c "import FreeCAD; print(FreeCAD.ConfigGet('UserAppData'))"
```

Harusnya keluar `/home/exedy/snap/freecad/common/`. Mod dir = UserAppData + `Mod/`.

## 5. Tes MCP server-nya

```bash
~/.local/bin/uvx freecad-mcp --help
```

Harusnya keluar usage dengan opsi `--host` (default `localhost`), `--auth-token`, `--freecadcmd`. Pertama kali jalan dia bakal download 39 package dari PyPI, itu normal.

## 6. Install Antigravity App (tar.gz)

```bash
ls -lh ~/Downloads/Antigravity.tar.gz
sudo apt update && sudo apt install -y curl tar desktop-file-utils python3
sudo mv ~/Downloads/Antigravity/Antigravity-x64 /opt/antigravity
sudo chown root:root /opt/antigravity/chrome-sandbox
sudo chmod 4755 /opt/antigravity/chrome-sandbox
sudo ln -sf /opt/antigravity/antigravity /usr/local/bin/antigravity
cat /opt/antigravity/version
which antigravity
```

Versi yang terpasang: 44.5.1. Bikin launcher biar muncul di app menu:

```bash
cat > /tmp/antigravity.desktop <<'EOF'
[Desktop Entry]
Name=Antigravity
Comment=Google Antigravity agent-first development platform
Exec=/usr/local/bin/antigravity %U
Terminal=false
Type=Application
Categories=Development;IDE;
StartupWMClass=Antigravity
EOF
sudo cp /tmp/antigravity.desktop /usr/share/applications/antigravity.desktop
sudo update-desktop-database /usr/share/applications/
```

Buka Antigravity dari app menu, login pakai akun Google Gemini Plus.

## 7. Daftarin MCP ke Antigravity

Cara GUI: Customizations (sidebar) > Installed > MCP Servers > Add MCP Server (+).

Cara file (yang dipakai, lebih pasti): edit `~/.gemini/config/mcp_config.json`, isinya harus persis gini:

```json
{
  "mcpServers": {
    "freecad": {
      "command": "/home/exedy/.local/bin/uvx",
      "args": ["freecad-mcp"]
    }
  }
}
```

Cek isinya:

```bash
cat ~/.gemini/config/mcp_config.json
```

Terus **restart Antigravity** (tutup, buka lagi). Config MCP baru dibaca pas start. Cek di Customizations > Installed > MCP Servers, `freecad` harus Connected, 12 tools.

## 8. Yang harus dilakuin di app FreeCAD-nya

Harus dibuka — nggak bisa dari terminal.

1. Buka FreeCAD dari app menu.
2. Di dropdown workbench (tengah atas), pilih **MCP Addon**.
3. Klik **Start RPC Server**.
4. Centang **Auto-Start Server** biar besok otomatis.
5. **Jangan tutup FreeCAD** selama Antigravity mau dipakai.

Verifikasi dari terminal (di tab lain):

```bash
ss -tlnp | grep 9875
```

Harusnya keluar:

```
LISTEN  127.0.0.1:9875  users:(("FreeCAD",pid=...,fd=...))
```

`127.0.0.1:9875` itu sama dengan `localhost:9875` — `localhost` itu nama untuk `127.0.0.1`. Jadi kalau di panel tulisannya `127.0.0.1:9875`, itu udah bener, nggak perlu diubah.

Status 2026-11-07: RPC server terkonfirmasi jalan di `127.0.0.1:9875` (LISTEN, FreeCAD pid 24814), Antigravity juga jalan. Test pertama yang berhasil: bikin 3D dari drawing 2D bentuk PNG.

## 9. Pakai MCP-nya di Antigravity

Bikin **new conversation** aja, jangan pakai chat lama. MCP tools itu otomatis masuk ke agent, nggak perlu `/` atau `@`:

- `@` itu buat mention file/folder, bukan MCP.
- `/` itu buat slash command / skill, bukan MCP.

Langsung perintahin pakai bahasa biasa, sebut FreeCAD biar jelas:

> Buatin box 100 x 60 x 20 mm di FreeCAD pakai MCP freecad, terus save sebagai box_test.FCStd

Kalau mau lebih pasti dia pakai MCP, bukan nulis kode sendiri:

> Cek dulu dokumen FreeCAD yang lagi kebuka pakai MCP freecad, terus buatin box 100 x 60 x 20 mm di dokumen itu.

## Kemungkinan error dan cara benerinnya

- **Workbench "MCP Addon" nggak ada di dropdown:** addon ditaruh di folder Mod yang salah, atau FreeCAD belum di-restart. FreeCAD cuma baca Mod pas start. Cek `ls ~/snap/freecad/common/Mod/`, harus ada `FreeCADMCP`. Kalau nggak ada, ulang langkah 4, terus tutup FreeCAD sampai benar-benar keluar, buka lagi.
- **Antigravity bilang MCP freecad disconnected / connection error:** FreeCAD ketutup, atau RPC Server belum di-Start. Balik ke langkah 8. Cek juga `ss -tlnp | grep 9875` — kalau kosong, RPC-nya mati.
- **MCP `freecad` nggak muncul di Antigravity sama sekali:** `mcp_config.json` kosong (file 0 byte) atau isinya salah, atau Antigravity belum di-restart setelah edit config. Cek `cat ~/.gemini/config/mcp_config.json`, pastikan JSON-nya valid dan `command`-nya path absolut `/home/exedy/.local/bin/uvx`.
- **`uvx: command not found` di Antigravity:** itu karena config pakai `uvx` doang tanpa path. Ganti ke path absolut di langkah 7.
- **FreeCAD versi 0.21 (dari apt):** addon nggak akan jalan. Uninstall apt-nya, install snap 1.1 di langkah 1.
- **Antigravity nggak kebuka / layar hitam:** biasanya AppArmor di Ubuntu 24.04+ ngeblok sandbox Chromium. Jalankan dari terminal biar kelihatan error-nya, jangan langsung pakai `--no-sandbox` permanen. Pastikan `chrome-sandbox` udah `root:root` dan `4755` (langkah 6).
- **`antigravity --version` error "Missing X server or $DISPLAY":** itu normal kalau jalanin dari SSH tanpa layar. Antigravity itu app GUI, harus dibuka dari desktop, bukan dari terminal SSH.
- **SSH langsung ke mini PC ditolak:** di Ubuntu Desktop, `openssh-server` nggak kepasang bawaan. Install dulu: `sudo apt install -y openssh-server && sudo systemctl enable --now ssh`. Ini cuma relevan kalau akses dari mesin lain — kalau duduk langsung di mini PC-nya, langkah ini nggak perlu.

## Checklist akhir, sebelum mulai kerja

- [ ] `freecad.cmd --version` → 1.1.4
- [ ] `~/.local/bin/uvx freecad-mcp --help` → jalan
- [ ] `ls ~/snap/freecad/common/Mod/` → ada `FreeCADMCP`
- [ ] `cat ~/.gemini/config/mcp_config.json` → isinya kayak langkah 7
- [ ] Antigravity udah restart, `freecad` Connected, 12 tools
- [ ] FreeCAD kebuka, workbench MCP Addon, RPC Server jalan, `ss -tlnp | grep 9875` → `127.0.0.1:9875`

## Terkait

- [[ROS2_Bruskart_Robot_Project_Overview]] — mini PC ground station + Ubuntu Desktop 24.04 yang sama
- [[ROS2_Installation]] — instalasi Ubuntu / ROS 2 di mesin yang sama

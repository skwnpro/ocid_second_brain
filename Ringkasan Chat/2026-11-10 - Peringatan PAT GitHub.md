# Peringatan PAT GitHub

Ringkasan — peringatan keamanan soal token GitHub (PAT) yang diminta Hermes/Cue.

**Catatan tanggal:** peringatan ini sebenarnya disampaikan 9 November 2026, bukan 10 November — ditulis di sini karena masuk daftar topik simulasi hari ini. Kejadiannya: 9 Nov 10:37 WIB, Ocid Hermes Bot (Hermes/Cue) meminta user membuat PAT GitHub ber-scope repo dan menyerahkannya; 9 Nov 10:41 WIB, Bre mengirim peringatan di main chat.

## Isi peringatan

- **Jangan paste PAT GitHub ke chat Hermes/Cue.** Token yang dipaste di chat dianggap bocor — sama seperti insiden 20 Oktober, saat user mem-paste fine-grained PAT di chat dan Bre menolak memakainya (token chat-paste tidak pernah disimpan/dipakai, dianggap exposed dan harus di-revoke).
- PAT ber-scope repo bisa membuka repo privat `finance_assistant` dan `ocid_second_brain` (vault).
- **Pakai jalan amannya, yang ditawarkan Hermes sendiri:** `gh auth login` di mesinnya — nilainya sama dengan menyerahkan PAT, tanpa token yang beredar di chat.
- Kalau sudah sempat ter-paste, revoke dulu di GitHub Settings → Developer settings → Personal access tokens.

## Terkait

- [[2026-11-10 - Hermes dan Gemini]] — Hermes/Cue yang meminta PAT; konteks kuota dan setup Hermes hari itu.

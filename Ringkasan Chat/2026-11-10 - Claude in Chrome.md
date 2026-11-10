# Claude in Chrome

Ringkasan 10 November 2026 — jawaban jujur soal extension Claude di Chrome (main chat, 10:38 WIB).

## Bisa klik-klik beneran

- Claude in Chrome itu extension resmi Anthropic, bukan sekadar sidebar chat. Claude membaca halaman, lalu **klik, ketik, isi form, navigasi antar tab** sendiri, memakai login yang sudah aktif di Chrome user. General availability sejak 26 Agustus 2026 untuk semua paket Claude berbayar.

## Batasannya

- **Berbayar:** butuh Claude Pro ($20/bulan) ke atas, tidak ada free tier. User hanya punya Gemini Plus ($4,99/bln), jadi ini langganan baru kalau mau coba.
- **Hanya Chrome desktop:** tidak jalan di Edge/Brave/Arc, tidak jalan di HP.
- **Risiko prompt injection itu nyata:** halaman web bisa menyisipkan instruksi tersembunyi yang dibaca Claude sebagai perintah user. Anthropic sendiri mengakui proteksinya "aren't foolproof" dan menyarankan jangan dipakai untuk banking atau data kesehatan. Ada kontrol izin per situs dan approve manual, tapi tes red-team mode otonomnya pernah tembus 23,6%.
- **Bisa mentok juga:** CAPTCHA dan bot detection tetap jadi tembok, dan kadang lambat atau sering minta konfirmasi untuk hal sensitif.

## Catatan

- Untuk kerjaan yang dilempar ke Bre, extension ini tidak dibutuhkan — browser cloud Bre juga bisa klik-klik dan isi form. Kelebihan Claude in Chrome: jalan di Chrome user sendiri pakai akun user, jadi lebih enak untuk kerjaan personal yang dilakukan langsung di browser.

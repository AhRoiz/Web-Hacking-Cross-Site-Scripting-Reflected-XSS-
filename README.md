---

## 💥 Web Hacking: Cross-Site Scripting (Reflected XSS)
**Target:** DVWA (Security Level: Low)
**Objective:** Membuktikan kerentanan XSS dengan menampilkan Session Cookie pengguna (Proof of Concept untuk Session Hijacking).

### 1. Attack Phase (Red Team)

**Skenario:**
Aplikasi web menerima input nama pengguna dan menampilkannya kembali di layar tanpa melakukan filter karakter spesial HTML/JavaScript.

**Payload:**
Menyuntikkan kode JavaScript untuk memanggil objek sensitif `document.cookie`.
```html
<script>alert(document.cookie)</script>
```

<img width="955" height="716" alt="Screenshot 2025-11-28 093444" src="https://github.com/user-attachments/assets/b96562ed-6bf8-484d-a6a4-4d74dfcbda9d" />

Hasil: Browser mengeksekusi script tersebut dan memunculkan Pop-up berisi PHPSESSID (Session Token). Jika token ini dikirim ke server penyerang, akun pengguna dapat diambil alih tanpa password (Session Hijacking).

2. Defense & Remediation (Blue Team)
Root Cause: Aplikasi tidak melakukan Output Encoding. Karakter berbahaya seperti < dan > dianggap sebagai teks biasa.

Rekomendasi Perbaikan:

Output Encoding: Mengonversi karakter spesial menjadi entitas HTML aman sebelum ditampilkan ke browser.

PHP: Gunakan fungsi htmlspecialchars($input, ENT_QUOTES, 'UTF-8');

HttpOnly Cookies: Mengatur flag HttpOnly pada session cookie agar tidak dapat dibaca melalui JavaScript (document.cookie).

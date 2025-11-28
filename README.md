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

<img width="956" height="761" alt="Screenshot 2025-11-28 093528" src="https://github.com/user-attachments/assets/cf3f53bd-2bfa-4381-8934-33b7783b1876" />

Hasil: Browser mengeksekusi script tersebut dan memunculkan Pop-up berisi PHPSESSID (Session Token). Jika token ini dikirim ke server penyerang, akun pengguna dapat diambil alih tanpa password (Session Hijacking).



### 2. Defense & Remediation (Blue Team) 
Root Cause: Aplikasi tidak melakukan Output Encoding. Karakter berbahaya seperti < dan > dianggap sebagai teks biasa.

Rekomendasi Perbaikan:

Output Encoding: Mengonversi karakter spesial menjadi entitas HTML aman sebelum ditampilkan ke browser.

PHP: Gunakan fungsi htmlspecialchars($input, ENT_QUOTES, 'UTF-8');

HttpOnly Cookies: Mengatur flag HttpOnly pada session cookie agar tidak dapat dibaca melalui JavaScript (document.cookie).

---

## 🐍 Python Scripting: Building a Custom Port Scanner
**Objective:** Transisi dari *Tool User* menjadi *Tool Builder*. Memahami cara kerja pemindaian jaringan pada level protokol (TCP Handshake) dengan menulis script Python dari nol.

### 1. The Code (Logika di Balik Layar)
Alih-alih menggunakan Nmap, saya membuat alat pemindai sederhana menggunakan library `socket` pada Python. Script ini bekerja dengan cara mencoba melakukan "Three-Way Handshake" ke setiap port target.

**Core Logic:**
```python
# Membuat socket (Telepon Virtual)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
socket.setdefaulttimeout(1) # Batas waktu tunggu 1 detik

# Mencoba koneksi (0 = Terbuka, Error = Tertutup)
result = s.connect_ex((target, port))
if result == 0:
    print(f"Port {port} is OPEN")
s.close()
```

### 2. Execution & Results
Script dijalankan untuk memindai 6.500 port pertama pada target Metasploitable 2.

Command:

```Bash

python3 scanner.py 192.168.56.101
```
Hasil Scan (Output): Script berhasil mengidentifikasi layanan kritis dan juga backdoor tersembunyi.

```Plaintext

Scanning Target: 192.168.56.101
--------------------------------------------------
Port 21 is OPEN   (FTP)
Port 22 is OPEN   (SSH)
Port 23 is OPEN   (Telnet)
Port 80 is OPEN   (HTTP)
Port 3306 is OPEN (MySQL Database)
Port 5432 is OPEN (PostgreSQL)
Port 5900 is OPEN (VNC Remote Desktop)
Port 1524 is OPEN (Ingreslock Backdoor)
Port 6000 is OPEN (X11)
...
```
<img width="654" height="513" alt="Screenshot 2025-11-28 224733" src="https://github.com/user-attachments/assets/c6ba8c86-a385-4037-a85c-9c008016f4b3" />


### 3. Technical Analysis (Why Nmap is Better?)
Selama pengujian, script ini memakan waktu hampir 2 jam untuk memindai 6.500 port.

Performance Bottleneck: Script ini bekerja secara Serial (Single-Threaded). Artinya, program menunggu port 1 selesai (timeout/connect), baru lanjut ke port 2.

Rumus: 6500 port x 1 detik (timeout) ≈ 108 menit.

Comparison with Nmap: Nmap jauh lebih cepat karena menggunakan Multi-threading / Asynchronous Scanning, di mana ia bisa mengirim ribuan paket sekaligus tanpa menunggu satu per satu.

### 4. Blue Team Perspective (Detection)
Script ini sangat "berisik" (Noisy) dan mudah dideteksi oleh Firewall/IDS.

Metode: Full TCP Connect Scan.

Log Signature: Firewall akan melihat satu IP (Penyerang) mencoba koneksi berurutan (Sequential) ke port 1, 2, 3, dst dalam waktu singkat.

Mitigasi: Mengatur aturan firewall untuk memblokir IP yang melakukan port knocking berurutan (Scan Detection).



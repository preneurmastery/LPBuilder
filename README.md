# LCC Intelligence - Landing Page & Ecosystem Documentation

Dokumentasi ini menjelaskan alur kerja (*flow*), arsitektur teknis, dan integrasi sistem untuk proyek **LCC Intelligence**.

## 1. Ikhtisar Sistem
LCC Intelligence adalah ekosistem *landing page* berbasis AI yang dirancang untuk pengiklan (advertiser). Sistem ini mengintegrasikan *frontend* yang persuasif dengan *backend* otomatisasi pembayaran dan aktivasi akun secara *real-time*.

---

## 2. Arsitektur Teknis

### Frontend (Landing Page)
- **Lokasi:** `~/LPBuilder/index.html`
- **Teknologi:** HTML5, Tailwind CSS, FontAwesome, JavaScript (Vanilla).
- **Fitur Utama:**
  - **Dynamic Countdown:** Menciptakan urgensi (scarcity).
  - **Niche Carousel:** Menampilkan contoh LP berdasarkan industri.
  - **Unified Tracking:** Integrasi Meta Pixel dan Custom Tracker.
  - **Responsive Checkout:** Form pengambilan data (Nama, Email, WA) sebelum pembayaran.

### Backend (Payment & Activation)
- **Lokasi:** `~/midtrans-integration-test/index.js`
- **Service Name (PM2):** `midtrans-test`
- **Teknologi:** Node.js, Express, MongoDB, Midtrans SDK.
- **Tanggung Jawab:**
  - Menghasilkan Snap Token Midtrans.
  - Menangani Webhook/Callback dari Midtrans.
  - Aktivasi user di Database (MongoDB).
  - Sinkronisasi data ke Meta Conversions API (CAPI).

---

## 3. Alur Pengguna (User Flow)

### Langkah 1: Kunjungan & Interaksi
Pengunjung masuk ke *Landing Page*. Sistem mulai melacak perilaku pengguna (scroll depth, klik FAQ, waktu di halaman) melalui **Meta Pixel** dan **Custom Tracker**.

### Langkah 2: Checkout (Data Entry)
Pengunjung mengisi Nama, Email, dan WhatsApp di bagian *Pricing*. Klik tombol "GABUNG SEKARANG" akan memicu fungsi `handleJoin()`:
1. Mengirim POST request ke `/api/get-snap-token`.
2. Backend menghasilkan `order_id` unik (Contoh: `ORDER-171316...`).
3. Frontend mengarahkan pengguna ke halaman pembayaran **Midtrans Snap**.

### Langkah 3: Pembayaran (Midtrans)
Pengguna melakukan pembayaran melalui berbagai metode (E-wallet, VA, Qris, dll).

### Langkah 4: Webhook & Aktivasi (Server-Side)
Setelah pembayaran sukses (`settlement`), Midtrans mengirimkan notifikasi ke `/api/midtrans-callback`:
1. **Verifikasi:** Backend memvalidasi keaslian notifikasi ke server Midtrans.
2. **Double-Layer Activation:**
   - **File-Based:** Email dicatat di `/home/apfitriyani/paid_emails.json` (sebagai backup tercepat).
   - **Database:** Status di koleksi `payments` diubah menjadi `paid`.
   - **User Update:** Mencari user di `tracker_users`, lalu mengubah status di koleksi `accounts` menjadi `active`.
3. **Meta CAPI:** Mengirim event `Purchase` ke Meta Server untuk optimasi iklan yang lebih akurat.

### Langkah 5: Redirect Pasca Bayar
Pengguna diarahkan kembali ke `https://preneurmastery.com/lcc/register` dengan parameter query (nama, email, phone) yang sudah terisi otomatis untuk menyelesaikan registrasi akun.

---

## 4. Integrasi Pihak Ketiga
- **Midtrans:** Gateway pembayaran.
- **Meta (Facebook) Pixel & CAPI:** Pelacakan konversi sisi klien dan server.
- **MongoDB Atlas:** Database utama untuk manajemen akun dan status pembayaran.
- **Gemini AI / Kimi:** (Logic eksternal) Digunakan untuk pembuatan copywriting di dalam ekosistem LCC.

---

## 5. Pemeliharaan (Maintenance)
- **Cek Status Service:** `pm2 list`
- **Log Real-time:** `pm2 logs midtrans-test`
- **Daftar Email Terbayar:** `cat ~/paid_emails.json`

---
*Dokumentasi ini dibuat secara otomatis oleh Gemini CLI Agent berdasarkan audit sistem pada April 2026.*

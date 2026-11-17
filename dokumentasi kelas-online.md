# Kelas Online (`kelas-online`)

Dokumentasi arsitektur, alur kerja pengembangan, dan panduan konteks proyek **Kelas Online**. File ini dirancang sebagai referensi utama bagi pengembang maupun asisten AI di masa mendatang agar langsung memahami konteks teknis, arsitektur kode, penyimpanan audio, serta alur *deployment* proyek ini.

## 1. Gambaran Umum Proyek

* **Nama Proyek:** Kelas Online (`kelas-online`)

* **Tujuan:** Aplikasi web pembelajaran daring tertutup (khusus member/siswa terdaftar) yang menyediakan materi pembelajaran serta pemutar audio terproteksi.

* **Konsep Akses & Autentikasi:**

  * **Login:** Menggunakan **Username atau Email** beserta **Password**.

  * **Registrasi Tertutup (Manual Admin):** Tidak ada pendaftaran otomatis oleh publik saat ini. Akun siswa dibuat secara manual oleh admin.

  * **Halaman Register (`register.astro`):** Berfungsi sebagai halaman panduan bagi calon siswa untuk menghubungi Admin via WhatsApp agar didaftarkan (sekaligus disiapkan jika di kemudian hari ingin dibuka form pendaftaran mandiri).

  * **Proteksi Rute & Audio:** Halaman materi (`materi.astro`) dan tautan audio dilindungi oleh *middleware* dan endpoint server (`/api/audio.js`) sehingga hanya bisa diakses oleh pengguna yang memiliki sesi login valid.

## 2. Tech Stack & Infrastruktur

* **Framework:** [Astro](https://astro.build/) **v4** (`^4.16.0`)

* **Bahasa Utama:** HTML, Vanilla CSS, Vanilla JavaScript (ES Modules) — *tanpa framework CSS/JS berat agar mudah diedit secara modular*.

* **Mode Rendering:** Server-Side Rendering (`output: 'server'`) menggunakan adapter `@astrojs/cloudflare` (diperlukan untuk validasi sesi login dan proteksi *streaming* audio).

* **Database & Penyimpanan:**

  * **Cloudflare D1:** Database SQLite untuk menyimpan data akun siswa dan metadata materi (termasuk pemetaan ID materi ke URL audio).

  * **Sesi Login:** *HttpOnly Cookie* bertanda tangan (HMAC).

  * **Storage Audio (Internet Archive):** File audio diunggah ke **Internet Archive** (`archive.org`) menggunakan nama item/file berupa kode acak yang disamarkan (bukan judul materi asli). Aplikasi hanya menyimpan tautan langsungnya di sisi server/database dan meneruskannya melalui `/api/audio.js` setelah sesi login tervalidasi.

* **Version Control & Alur Kerja:** Git & GitHub (`https://github.com/drivezimba/kelas-online`)

* **Hosting & Deployment:** Cloudflare Pages (terhubung langsung ke repositori GitHub untuk *auto-build* dan *auto-deploy*).

## 3. Alur Pengembangan & Konfigurasi Proyek

* **Fokus Alur Kerja:** Pengembangan kode dilakukan di editor lokal (Kali Linux) lalu **langsung di-push ke GitHub** (`git push`). Pengujian dan proses *build* berjalan langsung di lingkungan server **Cloudflare Pages** tanpa menjalankan server lokal (`npm run dev` / emulator lokal tidak digunakan).

* **Konfigurasi `astro.config.mjs`:**
  Tetap menggunakan Astro v4 dengan layanan gambar `noop` dan `platformProxy: { enabled: false }` agar bersih dan kompatibel penuh saat di-*build* di Cloudflare Pages:

```
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare({
    platformProxy: {
      enabled: false
    }
  }),
  image: {
    service: {
      entrypoint: 'astro/assets/services/noop'
    }
  }
});

```

## 4. Struktur Direktori Proyek

Kode dipisahkan secara modular antara komponen tampilan, tata letak (*layout*), logika backend, halaman, JavaScript klien, dan CSS agar mudah dirawat tanpa file yang menumpuk:

```
kelas-online/
├── public/                  # Aset statis publik (favicon, logo, gambar umum)
│   ├── favicon.ico
│   └── favicon.svg
├── src/
│   ├── components/          # Komponen UI modular yang dapat dipakai ulang
│   │   ├── Header.astro     # Bagian kepala website
│   │   └── Navbar.astro     # Menu navigasi utama
│   ├── layouts/             # Kerangka dasar halaman
│   │   └── BaseLayout.astro # Pembungkus utama (menggabungkan Header, Navbar, CSS, & JS)
│   ├── lib/                 # Modul logika backend (server-side)
│   │   ├── auth.js          # Utilitas pembuatan/verifikasi sesi login & hash password
│   │   └── db.js            # Koneksi & query database (Cloudflare D1)
│   ├── pages/               # Rute halaman web & endpoint API
│   │   ├── api/             # Endpoint backend (tidak merender HTML)
│   │   │   ├── login.js     # POST: Validasi username/email & password, set cookie sesi
│   │   │   ├── logout.js    # POST/GET: Hapus cookie sesi & redirect ke login
│   │   │   └── audio.js     # GET: Proxy/streaming audio dari Internet Archive setelah cek sesi
│   │   ├── index.astro      # Halaman utama / beranda publik
│   │   ├── login.astro      # Halaman formulir masuk siswa
│   │   ├── register.astro   # Halaman panduan pendaftaran via WhatsApp Admin
│   │   └── materi.astro     # Halaman daftar materi & pemutar audio (terproteksi)
│   ├── scripts/             # JavaScript khusus sisi klien (browser)
│   │   └── main.js          # Interaksi UI (toggle navbar, kontrol pemutar audio, dll.)
│   ├── styles/              # Stylesheet CSS terpisah
│   │   ├── global.css       # Gaya tampilan global (tipografi, warna, layout dasar)
│   │   └── login.css        # Gaya tampilan khusus halaman login & register
│   └── middleware.js        # Penjaga rute otomatis (redirect ke /login jika belum terautentikasi)
├── astro.config.mjs         # Konfigurasi Astro, adapter Cloudflare, & image service noop
├── package.json             # Daftar dependensi & skrip npm
└── README.md                # Dokumentasi proyek ini

```

## 5. Status Pengembangan Saat Ini

* \[x\] Inisialisasi proyek Astro v4 (template *minimal/empty*)

* \[x\] Penyesuaian `astro.config.mjs` (`image.service` -> `noop`)

* \[x\] Inisialisasi Git & koneksi ke repositori GitHub (`drivezimba/kelas-online`)

* \[x\] Pembuatan kerangka folder dan file kosong di dalam `src/`

* \[ \] Pengisian kode tampilan dasar (`BaseLayout.astro`, `Header.astro`, `Navbar.astro`, `global.css`, `main.js`)

* \[ \] Pengisian kode halaman publik (`index.astro`, `login.astro`, `register.astro`, `login.css`)

* \[ \] Implementasi autentikasi & database (`lib/db.js`, `lib/auth.js`, `middleware.js`, `api/login.js`, `api/logout.js`)

* \[ \] Implementasi halaman materi & streaming audio terproteksi dari Internet Archive (`materi.astro`, `api/audio.js`)

* \[ \] Instalasi adapter `@astrojs/cloudflare` & koneksi auto-deploy ke Cloudflare Pages

## 6. Alur Kerja Git & Deployment

* **Repositori GitHub:** `https://github.com/drivezimba/kelas-online`

* **Branch Utama:** `main`

* **Identitas Git Lokal:**

  * `user.name`: `drivezimba`

  * `user.email`: `dl.drive.zimba@gmail.com`

### Perintah Rutin Update Kode ke GitHub:

Setiap kali selesai mengedit kode, langsung kirim perubahan ke GitHub dengan menjalankan perintah berikut di folder `kelas-online`:

```
git add . && git commit -m "deskripsi singkat perubahan" && git push

```

Ketika repositori sudah dihubungkan ke **Cloudflare Pages**, setiap `git push` ke branch `main` akan otomatis memicu proses *build* (`npm run build`) dan memperbarui aplikasi secara langsung.

## 7. Panduan Instruksi untuk Asisten AI Selanjutnya

Jika Anda membantu mengembangkan proyek ini pada sesi berikutnya:

1. **Patuhi Struktur Modular:** Jangan menggabungkan semua CSS dan JS ke dalam satu file `.astro`. Tempatkan CSS di `src/styles/` dan JS interaktif browser di `src/scripts/`.

2. **Alur Kerja Tanpa Server Lokal:** Pengguna tidak menjalankan server lokal (`npm run dev`). Fokuskan kode agar langsung siap di-push ke GitHub dan di-build otomatis di Cloudflare Pages.

3. **Proteksi Tautan Internet Archive:** Jangan menaruh URL asli `archive.org` secara terbuka di kode HTML frontend. Gunakan endpoint server `/api/audio.js` untuk memvalidasi sesi login terlebih dahulu sebelum meneruskan aliran (*stream*) audio ke pemutar di halaman `materi.astro`.

4. **Gaya Bahasa:** Gunakan bahasa Indonesia yang jelas, terstruktur, dan langsung pada langkah teknis.
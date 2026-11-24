# README: Daftar Pembaruan Proyek "Kelas Online"

Dokumen ini adalah daftar kerja untuk pembaruan berikutnya. Dibuat untuk dikerjakan oleh pemilik atau AI yang membaca paket zip terbaru.

**Aturan umum (wajib diikuti):**
1. **Jangan ubah isi materi** (`src/content/pelajaran/**/*.mdx`) kecuali diminta secara eksplisit.
2. **Jangan tulis rahasia baru ke kode.** Kunci, password, dan token hanya boleh di Cloudflare Pages → Settings → Variables and Secrets (tipe Secret).
3. Perubahan **kecil dan terfokus**. Untuk setiap file yang berubah, berikan isi lengkap file dan sebutkan path tujuannya.
4. Sebelum menyarankan perubahan yang bergantung pada versi library, cek `package-lock.json`. Versi di sana yang berlaku.
5. Setelah perubahan, `npm run build` harus berhasil. Cloudflare Pages menjalankan build yang sama.
6. Catat setiap perubahan di bagian **Riwayat Perubahan** di bawah.

---

## A. Keamanan (prioritas tertinggi)

| No | Status | Masalah | Lokasi | Perbaikan yang disarankan |
|---|---|---|---|---|
| S1 | [ ] | **Kunci penandatanganan sesi tertulis di kode.** Nilainya `kunci-rahasia-kelas-online-ganti-nanti-2026`. Siapa pun yang bisa membaca kode bisa membuat token dengan `role: "super_admin"`. | `src/lib/auth.js` baris 2 | Pindahkan ke secret `SESSION_SECRET` di Cloudflare. Baca lewat `locals.runtime.env` atau `import.meta.env` sesuai adapter. Ganti nilainya. Setelah itu semua sesi lama tidak valid, dan itu diharapkan. |
| S2 | [ ] | **Password tertulis dalam teks biasa** untuk `mbm121204` (super admin) dan `siswa01`. | `src/lib/db.js` baris 1–18 dan fallback di `findUser` baris 57–66 | Hapus daftar `USERS` dan cabang fallback dari kode. Ganti password super admin di database. Anggap password lama sudah bocor jika repo pernah publik. |
| S3 | [ ] | **Jalur fail-open saat `db` tidak ada.** `hasCourseAccess` mengembalikan `true`. | `src/lib/db.js` baris 122. Dipakai di `src/pages/api/audio.js`, `src/pages/materi/[mapel]/[pertemuan].astro`. Di `src/pages/materi/index.astro`, `isAdmin \|\| !db` juga menampilkan semua kursus. | Ubah menjadi fail-closed: jika `!db`, kembalikan `false` untuk peserta biasa. Admin tetap boleh. Sesuaikan juga `getAllCourses` dan `getUserAllowedCourses` agar tidak memberi akses penuh tanpa database. |
| S4 | [ ] | **Hash password tanpa salt** (SHA-256 biasa). | `src/lib/db.js` baris 45–51 | Ganti dengan PBKDF2 (Web Crypto, dengan salt acak per pengguna, minimal 100.000 iterasi) dan simpan format yang bisa dikenali. Lakukan migrasi: saat login berhasil dengan hash lama, hitung ulang dengan format baru. |
| S5 | [ ] | **Tidak ada batas percobaan login.** | `src/pages/api/login.js` | Tambahkan batas sederhana per identifier dan per IP, misalnya tabel D1 atau counter di KV. Tambahkan jeda setelah beberapa kali gagal. |
| S6 | [ ] | **Sesi tidak diperiksa ulang ke database.** Role dan status akun dibaca dari token yang berlaku 7 hari. | `src/lib/auth.js`, `src/middleware.js` | Pendekkan masa berlaku token (misalnya 12 jam) atau periksa ulang ke D1 pada aksi sensitif. Pastikan penghapusan dan penurunan role langsung berlaku. |
| S7 | [ ] | **Open redirect.** Nilai `return_to` dari form dipakai langsung. | `src/pages/api/change-password.js` baris 17 | Terima hanya path relatif yang diawali `/` dan tidak diawali `//`. Jika tidak valid, pakai `/akun`. |
| S8 | [ ] | **Logout bisa lewat GET.** | `src/pages/api/logout.js` | Hapus handler `GET`, dan gunakan `POST` dari form di Header/Sidebar. |
| S9 | [ ] | Aksi `update_courses` tidak memeriksa apakah target adalah siswa. | `src/pages/api/admin/students.js` baris 33–40 | Tambahkan pemeriksaan role target = `siswa` di `updateStudentCourses`, seperti yang sudah dilakukan `deleteStudent`. |

## B. Bug fungsional

| No | Status | Masalah | Lokasi | Perbaikan yang disarankan |
|---|---|---|---|---|
| B1 | [ ] | **Filter dan pencarian beranda tidak menyembunyikan kartu.** `.course-card` memakai `display: flex`, sehingga atribut `hidden` diabaikan. (Sudah diverifikasi.) | `src/pages/index.astro` baris 973 (CSS) dan sekitar baris 596 (script `card.hidden = !show`) | Tambahkan `.course-card[hidden] { display: none; }` di blok CSS yang sama. |
| B2 | [ ] | Pesan salah di `deleteAdmin`. Jika `changes === 0` (ID tidak ada atau bukan admin), pesannya selalu "Akun Super Admin dilindungi". | `src/lib/db.js` baris 264–280 | Cek dulu apakah target ada dan role-nya `admin`. Pesan hanya disesuaikan dengan kondisi itu. |
| B3 | [ ] | Fallback `DEFAULT_COURSES` terjadi diam-diam jika tabel kosong atau query gagal. Kolom `price` dari DB tidak dipakai. | `src/lib/db.js` baris 91–101 | Catat error ke `console.error`. Tampilkan peringatan di dasbor admin. Gunakan `price` dari DB. |
| B4 | [ ] | Harga di beranda hardcoded di `COURSE_META`. | `src/pages/index.astro` | Pindahkan ke kolom `price` di DB, atau jelaskan di README bahwa `COURSE_META` harus diperbarui manual. |
| B5 | [ ] | Kursus baru di DB tanpa entri `COURSE_META` otomatis masuk "Metodologi & Riset" dengan deskripsi generik. | `src/pages/index.astro` | Tampilkan kategori "Lainnya" atau tandai kursus yang belum punya metadata. |
| B6 | [ ] | Mode dev tidak punya database, sehingga halaman terkunci tidak bisa diuji lokal. | `astro.config.mjs` (`platformProxy: { enabled: false }`) | Aktifkan `platformProxy` untuk dev, atau sediakan database lokal dengan `wrangler d1 execute --local`. |

## C. Teks dan antarmuka

| No | Status | Masalah | Lokasi | Perbaikan yang disarankan |
|---|---|---|---|---|
| C1 | [ ] | "Satu kali aktivasi" tetapi harganya "/ bulan". Dari HANDOFF, belum diverifikasi ulang baris pastinya. | `src/pages/index.astro` sekitar baris 152 | Samakan teksnya dengan model harga yang berlaku. |
| C2 | [ ] | Singkatan "/ bln" berbeda dengan "/ bulan". Dari HANDOFF. | `src/pages/index.astro` sekitar baris 267 | Gunakan satu bentuk saja. |
| C3 | [ ] | "8+ Pertemuan Tersedia" menghitung semua file MDX, bukan pertemuan per kursus. Dari HANDOFF. | `src/pages/index.astro` sekitar baris 237 | Hitung dari koleksi `pelajaran`, atau ubah teksnya. |
| C4 | [ ] | Tombol filter di `role="tablist"` tidak memakai `role="tab"` atau `aria-selected`. | `src/pages/index.astro` | Tambahkan atribut ARIA yang sesuai, dan kelola `aria-selected` dari script. |
| C5 | [ ] | Kotak pencarian di beranda tidak punya label yang bisa dibaca pembaca layar. | `src/pages/index.astro` | Tambahkan `<label>` tersembunyi visual (`sr-only`) atau `aria-label`. |
| C6 | [ ] | Kode mati: blok `.etalase-*` di CSS tidak dipakai. | `src/styles/global.css` | Cari dulu class yang dibentuk secara dinamis, lalu hapus blok yang benar-benar tidak terpakai. |

## D. Kerapian dan pemeliharaan

| No | Status | Masalah | Perbaikan |
|---|---|---|---|
| D1 | [ ] | `wrangler.toml` berisi `database_id`. Ini bukan rahasia, tetapi tidak perlu diubah. | Biarkan. Jangan tambahkan token atau secret ke file ini. |
| D2 | [ ] | Tidak ada tes otomatis. | Tambahkan minimal tes untuk `verifySessionToken`, `hasCourseAccess` (termasuk jalur `!db`), dan `changeOwnPassword`. |
| D3 | [ ] | `CHANGES.diff` dan HANDOFF menyebut perubahan CSS sidebar dan pemisahan `lesson.css`. Belum diuji di browser. | Uji sesuai bagian "Cara menguji" di bawah sebelum dianggap selesai. |

---

## E. Cara menguji setelah perubahan

1. `npm install` (jika perlu), lalu `npm run build`. Harus berhasil tanpa error.
2. `npm run dev` (atau `astro dev --background` sesuai `AGENTS.md`). Cek:
   - Halaman pertemuan: judul, audio, dan navigasi bawah tampil. Padding `.lesson-detail` di layar kecil 1.25rem.
   - Sidebar: rail desktop diperkecil (border-radius 16px), drawer mobile di bawah 768px tingginya mengikuti isi.
   - Beranda: filter dan pencarian benar-benar menyembunyikan kartu (setelah B1).
3. Uji akses: login sebagai peserta, buka kursus yang tidak dimiliki (harus terkunci), lalu buka `/api/audio` untuk kursus yang tidak dimiliki (harus 403).
4. Uji login dengan password salah berulang (setelah S5).
5. Deploy ke Cloudflare Pages dengan **Create deployment**, lalu tes di mode penyamaran.

## F. Setelah deploy

- Pastikan secret `SESSION_SECRET` (setelah S1) dan secret lain sudah terisi. Setelah mengganti secret, lakukan deploy ulang.
- Ganti password super admin dan siswa uji. Jangan gunakan nilai yang tertulis di kode lama.

---

## Riwayat Perubahan

| Tanggal | Yang diubah | Oleh |
|---|---|---|
| (isi) | | |

---

## Prompt untuk AI (salin-tempel di chat baru)

> Saya punya proyek Astro 4.16 dengan adapter Cloudflare (output server) dan database D1 bernama "kelas-online". Binding database: `DB`. Saya mengunggah zip proyek terbaru.
>
> Baca dulu `README.md` (daftar pembaruan ini), `HANDOFF.md`, dan `project/AGENTS.md`. Ikuti aturan di bagian "Aturan umum".
>
> Kerjakan **hanya** item yang saya tandai, dan ikuti urutan prioritas: S1 sampai S9, lalu B1 sampai B6, lalu C dan D. Jika saya tidak menyebutkan item tertentu, tanyakan dulu sebelum mengubah.
>
> Aturan kerja:
> - Jangan ubah isi `src/content/pelajaran/**/*.mdx`.
> - Jangan menulis rahasia, password, atau token baru ke kode. Gunakan nama secret di Cloudflare dan jelaskan langkahnya kepada saya.
> - Untuk setiap file yang berubah, berikan isi lengkap file dan path tujuannya.
> - Sebelum menyarankan perubahan yang bergantung pada versi library, cek `package-lock.json`.
> - Jika ada perubahan yang berisiko memutus login atau akses peserta (S1, S3, S4, S6), jelaskan dampaknya dan langkah migrasi sebelum mengubah kode.
> - Setelah selesai, tulis ringkasan: item apa yang dikerjakan, file mana yang berubah, dan langkah uji mana yang harus saya jalankan.
> - Perbarui bagian "Riwayat Perubahan" di README.md.

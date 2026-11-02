# Template Tema Website Modul Interaktif Astro v5

Dokumen ini berisi panduan lengkap dan seluruh file kode yang dibutuhkan untuk membangun **Tema Website Modul Pembelajaran Interaktif Astro v5**. Tampilan dan tata letak tema ini disesuaikan sepenuhnya dengan spesifikasi pada **Contoh Tampilan Web** serta struktur folder pada **Struktur Folder Astro** dan **Catatan Struktur Folder** [1, 2, 3].

---

## 1. Arsitektur Tema & Fitur Utama

- **Otomatisasi Content Collections (Astro v5)**: Menggunakan fitur `glob` loader terbaru dari Astro v5 pada `src/content.config.ts`. Setiap kali Anda menambahkan file `.md` baru di dalam subfolder materi, Astro secara otomatis mendeteksi dan memperbarui navigasi tanpa perlu mengubah kode HTML/JS secara manual.
- **Navigasi Sidebar Dinamis Akordeon**: Menyusun hirarki berjenjang (`Kelas` -> `Semester` -> `Mata Pelajaran` -> `Daftar Pelajaran`) secara otomatis dari folder materi yang ada.
- **Sistem 3 Tema Warna**: Mendukung mode **Terang (Light)**, **Mode Baca (Sepia)**, dan **Gelap (Dark)** menggunakan variabel CSS kustom dengan preferensi yang tersimpan di `localStorage`.
- **Pengatur Ukuran Font**: Tombol interaktif `A-` dan `A+` untuk menyesuaikan ukuran teks bacaan materi sesuai kenyamanan pengguna.
- **Dukungan Teks Arab & Typography**: Integrasi Google Fonts (`Inter` untuk teks umum dan `Amiri` untuk teks Arab) serta styling khusus RTL (`.teks-arab`).
- **Navigasi Artikel Sebelumnya & Selanjutnya**: Menghitung urutan materi secara otomatis berdasarkan kelas, semester, mata pelajaran, dan nomor pelajaran sehingga tombol `Materi Sebelumnya` dan `Materi Selanjutnya` di bawah artikel selalu tepat sasaran.

---

## 2. Daftar File yang Perlu Dibuat / Diubah

Berikut adalah struktur file tema Astro v5 yang perlu Anda sesuaikan di dalam direktori proyek Anda:

```text
src/
├── content/                         <-- Folder materi Markdown Anda
│   └── kelas-1/semester-1/akhlak/pelajaran-1.md
├── layouts/
│   └── Layout.astro                 <-- Layout Utama & UI (Header, Sidebar, Script Tema/Font)
├── pages/
│   ├── [...slug].astro              <-- Dynamic Route untuk Pembaca Materi & Generator Sidebar
│   └── index.astro                  <-- Beranda Utama / Auto-redirect ke Materi Pertama
└── content.config.ts                <-- Konfigurasi Content Collections Astro v5
astro.config.mjs                     <-- Konfigurasi Integrasi Tailwind CSS
```

---

## 3. Kode Lengkap File Proyek

### A. Konfigurasi Content Collections (`src/content.config.ts`)

File ini mengonfigurasi fitur pindaian materi di Astro v5 menggunakan `glob` loader dari `astro/loaders`.

```typescript
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

// Mengonfigurasi koleksi materi agar memindai seluruh folder di src/content
const materi = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content' }),
  schema: z.object({
    title: z.string(),
    author: z.string().optional().default('Ustadz Pembimbing'),
    bab: z.string().optional(),
    readTime: z.string().optional().default('5 menit'),
  }),
});

export const collections = { materi };
```

---

### B. Konfigurasi Astro (`astro.config.mjs`)

```javascript
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()],
});
```

---

### C. Layout Utama & Theme Wrapper (`src/layouts/Layout.astro`)

Layout ini mencakup Header, Navbar, Pengatur Font, Dropdown Tema, Font Google (`Inter` & `Amiri`), serta Ikon `@phosphor-icons/web`.

```astro
---
interface Props {
  title: string;
}

const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{title}</title>
    
    <!-- Google Fonts: Inter (Latin) & Amiri (Arabic) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Amiri:ital,wght@0,400;0,700;1,400&family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <!-- Phosphor Icons -->
    <script is:inline src="https://unpkg.com/@phosphor-icons/web"></script>

    <!-- Tailwind CSS (Variabel Kustom 3 Tema) -->
    <script is:inline src="https://cdn.tailwindcss.com"></script>
    <script is:inline>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        arabic: ['Amiri', 'serif'],
                    },
                    colors: {
                        theme: {
                            bg: 'var(--bg-main)',
                            panel: 'var(--bg-panel)',
                            text: 'var(--text-main)',
                            muted: 'var(--text-muted)',
                            border: 'var(--border)',
                            accent: 'var(--accent)',
                            hover: 'var(--accent-hover)',
                        }
                    }
                }
            }
        }
    </script>

    <style is:global>
        /* CSS Variables untuk Sistem 3 Tema */
        :root {
            --bg-main: #f8fafc;
            --bg-panel: #ffffff;
            --text-main: #1e293b;
            --text-muted: #64748b;
            --border: #e2e8f0;
            --accent: #0ea5e9;
            --accent-hover: #0284c7;
        }

        :root[data-theme="dark"] {
            --bg-main: #0f172a;
            --bg-panel: #1e293b;
            --text-main: #f1f5f9;
            --text-muted: #94a3b8;
            --border: #334155;
            --accent: #38bdf8;
            --accent-hover: #7dd3fc;
        }

        :root[data-theme="baca"] {
            --bg-main: #f3ece1;
            --bg-panel: #fcf9f2;
            --text-main: #433422;
            --text-muted: #857560;
            --border: #e6dccb;
            --accent: #d97706;
            --accent-hover: #b45309;
        }

        body, .transition-theme {
            transition: background-color 0.4s ease, color 0.4s ease, border-color 0.4s ease;
        }

        .teks-arab {
            direction: rtl;
            text-align: right;
            line-height: 2.2;
        }

        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: var(--bg-main); 
        }
        ::-webkit-scrollbar-thumb {
            background: var(--border); 
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: var(--text-muted); 
        }

        .sidebar-transition {
            transition: transform 0.3s ease-in-out;
        }
    </style>
</head>
<body class="bg-theme-bg text-theme-text font-sans h-screen flex overflow-hidden antialiased">

    <!-- Overlay untuk Mobile Sidebar -->
    <div id="mobile-overlay" class="fixed inset-0 bg-black/50 z-40 hidden lg:hidden transition-opacity" onclick="toggleSidebar()"></div>

    <!-- SIDEBAR CONTAINER -->
    <slot name="sidebar" />

    <!-- AREA KONTEN UTAMA -->
    <main class="flex-1 flex flex-col min-w-0 h-full relative">
        
        <!-- TOP NAVIGATION BAR -->
        <header class="h-16 bg-theme-panel border-b border-theme-border flex items-center justify-between px-4 lg:px-8 shrink-0 transition-theme shadow-sm relative z-40">
            <div class="flex items-center gap-3 md:gap-6">
                <button onclick="toggleSidebar()" class="lg:hidden p-2 rounded-lg bg-theme-bg border border-theme-border text-theme-text hover:text-theme-accent">
                    <i class="ph ph-list text-xl"></i>
                </button>
                <a href="/" class="flex items-center gap-2 font-bold text-lg text-theme-accent">
                    <i class="ph ph-books text-2xl"></i>
                    <span>Modul Sekolah</span>
                </a>
            </div>

            <div class="flex items-center gap-3 lg:gap-5">
                <!-- Search Box -->
                <div class="relative hidden sm:block">
                    <i class="ph ph-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-theme-muted"></i>
                    <input type="text" placeholder="Cari materi..." class="bg-theme-bg border border-theme-border rounded-full pl-9 pr-4 py-1.5 text-sm text-theme-text focus:outline-none focus:border-theme-accent focus:ring-1 focus:ring-theme-accent w-48 lg:w-64 transition-all">
                </div>
                
                <!-- Font Size Adjuster (A- / A+) -->
                <div class="flex items-center bg-theme-bg rounded-lg border border-theme-border p-1">
                    <button onclick="ubahFont(-1)" class="px-3 py-1 font-serif text-sm font-bold text-theme-muted hover:text-theme-text rounded hover:bg-theme-panel transition" title="Perkecil Teks">
                        A-
                    </button>
                    <div class="w-[1px] h-4 bg-theme-border mx-1"></div>
                    <button onclick="ubahFont(1)" class="px-3 py-1 font-serif text-lg font-bold text-theme-muted hover:text-theme-text rounded hover:bg-theme-panel transition" title="Perbesar Teks">
                        A+
                    </button>
                </div>

                <div class="w-[1px] h-6 bg-theme-border hidden sm:block"></div>

                <!-- Theme Selector Dropdown -->
                <div class="relative">
                    <button onclick="toggleThemeMenu()" id="btn-theme-main" class="flex items-center justify-center p-2 text-theme-text bg-theme-bg border border-theme-border rounded-lg shadow-sm hover:bg-theme-panel transition" title="Ganti Tema">
                        <i class="ph ph-sun text-xl" id="main-theme-icon"></i>
                        <i class="ph ph-caret-down text-xs ml-1 text-theme-muted"></i>
                    </button>
                    
                    <div id="theme-dropdown" class="hidden absolute right-0 mt-2 w-40 bg-theme-panel border border-theme-border rounded-xl shadow-lg overflow-hidden z-50">
                        <button onclick="setTema('light')" class="w-full flex items-center gap-3 px-4 py-3 text-left text-sm text-theme-text hover:bg-theme-bg transition">
                            <i class="ph ph-sun text-lg"></i> Terang
                        </button>
                        <button onclick="setTema('baca')" class="w-full flex items-center gap-3 px-4 py-3 text-left text-sm text-theme-text hover:bg-theme-bg transition">
                            <i class="ph ph-book-open text-lg text-amber-600"></i> Mode Baca
                        </button>
                        <button onclick="setTema('dark')" class="w-full flex items-center gap-3 px-4 py-3 text-left text-sm text-theme-text hover:bg-theme-bg transition">
                            <i class="ph ph-moon text-lg text-sky-400"></i> Gelap
                        </button>
                    </div>
                </div>
            </div>
        </header>

        <!-- AREA KONTEN SCROLLABLE -->
        <div class="flex-1 overflow-y-auto p-4 lg:p-10 relative scroll-smooth" id="scroll-area">
            <slot />
        </div>
    </main>

    <!-- INTERAKTIVITAS CLIENT-SIDE -->
    <script is:inline>
        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            const overlay = document.getElementById('mobile-overlay');
            if (sidebar && overlay) {
                if (sidebar.classList.contains('-translate-x-full')) {
                    sidebar.classList.remove('-translate-x-full');
                    overlay.classList.remove('hidden');
                } else {
                    sidebar.classList.add('-translate-x-full');
                    overlay.classList.add('hidden');
                }
            }
        }

        function toggleAccordion(id) {
            const content = document.getElementById(id);
            const icon = document.getElementById(`icon-${id}`);
            if (content && icon) {
                if (content.classList.contains('hidden')) {
                    content.classList.remove('hidden');
                    icon.style.transform = 'rotate(180deg)';
                } else {
                    content.classList.add('hidden');
                    icon.style.transform = 'rotate(0deg)';
                }
            }
        }

        function toggleThemeMenu() {
            const dropdown = document.getElementById('theme-dropdown');
            if (dropdown) dropdown.classList.toggle('hidden');
        }

        document.addEventListener('click', function(event) {
            const menu = document.getElementById('theme-dropdown');
            const btn = document.getElementById('btn-theme-main');
            if (menu && btn && !btn.contains(event.target) && !menu.contains(event.target)) {
                menu.classList.add('hidden');
            }
        });

        function setTema(tema) {
            document.documentElement.removeAttribute('data-theme');
            if (tema !== 'light') {
                document.documentElement.setAttribute('data-theme', tema);
            }
            const iconEl = document.getElementById('main-theme-icon');
            if (iconEl) {
                if (tema === 'light') iconEl.className = 'ph ph-sun text-xl';
                else if (tema === 'dark') iconEl.className = 'ph ph-moon text-xl text-sky-400';
                else if (tema === 'baca') iconEl.className = 'ph ph-book-open text-xl text-amber-600';
            }
            const dropdown = document.getElementById('theme-dropdown');
            if (dropdown) dropdown.classList.add('hidden');
            localStorage.setItem('preferensiTema', tema);
        }

        let ukuranFont = 16;
        function ubahFont(perubahan) {
            const artikel = document.getElementById('artikel-konten');
            if (artikel) {
                let ukuranBaru = ukuranFont + (perubahan * 2);
                if (ukuranBaru >= 14 && ukuranBaru <= 24) {
                    ukuranFont = ukuranBaru;
                    artikel.style.fontSize = `${ukuranFont}px`;
                }
            }
        }

        window.addEventListener('DOMContentLoaded', () => {
            const temaTersimpan = localStorage.getItem('preferensiTema') || 'light';
            setTema(temaTersimpan);
        });
    </script>
</body>
</html>
```

---

### D. Halaman Pembaca Materi Dinamis (`src/pages/[...slug].astro`)

File halaman ini bertugas:
1. Mengambil semua file materi Markdown melalui `getCollection('materi')`.
2. Mengurutkan materi secara matematis (`Kelas 1..6` -> `Semester 1..2` -> `Mapel` -> `Pelajaran 1..N`).
3. Membangun struktur **Menu Sidebar Akordeon secara otomatis**, sehingga jika file baru ditambahkan, menu langsung terupdate tanpa link rusak.
4. Menghitung materi **Sebelumnya** dan **Selanjutnya** secara dinamis untuk tombol navigasi artikel bawah.

```astro
---
import Layout from '../layouts/Layout.astro';
import { getCollection, render } from 'astro:content';

export async function getStaticPaths() {
  const allEntries = await getCollection('materi');

  // Helper pengurutan hirarkis
  const parseSortKey = (id: string) => {
    const parts = id.split('/'); // ex: ['kelas-1', 'semester-1', 'akhlak', 'pelajaran-1']
    const kelasNum = parseInt(parts[0]?.replace('kelas-', '') || '0', 10);
    const semNum = parseInt(parts[1]?.replace('semester-', '') || '0', 10);
    const mapel = parts[2] || '';
    const pelNum = parseInt(parts[3]?.replace('pelajaran-', '') || '0', 10);
    return { kelasNum, semNum, mapel, pelNum };
  };

  const sortedEntries = allEntries.sort((a, b) => {
    const keyA = parseSortKey(a.id);
    const keyB = parseSortKey(b.id);

    if (keyA.kelasNum !== keyB.kelasNum) return keyA.kelasNum - keyB.kelasNum;
    if (keyA.semNum !== keyB.semNum) return keyA.semNum - keyB.semNum;
    if (keyA.mapel !== keyB.mapel) return keyA.mapel.localeCompare(keyB.mapel);
    return keyA.pelNum - keyB.pelNum;
  });

  return sortedEntries.map((entry, index) => {
    const prev = index > 0 ? sortedEntries[index - 1] : null;
    const next = index < sortedEntries.length - 1 ? sortedEntries[index + 1] : null;
    return {
      params: { slug: entry.id },
      props: { entry, prev, next, allMateri: sortedEntries },
    };
  });
}

const { entry, prev, next, allMateri } = Astro.props;
const { Content } = await render(entry);

// Memecah ID untuk Breadcrumb & Label Navigasi
const pathParts = entry.id.split('/');
const kelasNum = pathParts[0]?.replace('kelas-', '') || '1';
const semNum = pathParts[1]?.replace('semester-', '') || '1';
const mapelRaw = pathParts[2] || 'akhlak';
const mapelLabel = mapelRaw.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ');

// Pemetaan Ikon Spesifik Mata Pelajaran
const mapelIcons: Record<string, string> = {
  'akhlak': 'ph-heart',
  'aqidah': 'ph-mosque',
  'bahasa-arab': 'ph-book-open-text',
  'bahasa-indonesia': 'ph-chat-circle-text',
  'fiqih': 'ph-scales',
  'imla': 'ph-pen-nib',
  'ipa': 'ph-flask',
  'ips': 'ph-globe-hemisphere-west',
  'matematika': 'ph-calculator',
  'siroh': 'ph-scroll',
  'tajwid': 'ph-book-bookmark'
};

// Mengorganisir struktur hirarki menu sidebar secara otomatis
type LessonLink = { id: string; title: string; active: boolean };
type MapelGroup = { name: string; icon: string; lessons: LessonLink[] };
type SemesterGroup = { name: string; mapels: Record<string, MapelGroup> };
type KelasGroup = { name: string; semesters: Record<string, SemesterGroup> };

const menuTree: Record<string, KelasGroup> = {};

allMateri.forEach((item) => {
  const parts = item.id.split('/');
  const kKey = parts[0] || 'kelas-1';
  const sKey = parts[1] || 'semester-1';
  const mKey = parts[2] || 'akhlak';

  if (!menuTree[kKey]) {
    const kN = kKey.replace('kelas-', '');
    menuTree[kKey] = { name: `Kelas ${kN}`, semesters: {} };
  }
  if (!menuTree[kKey].semesters[sKey]) {
    const sN = sKey.replace('semester-', '');
    menuTree[kKey].semesters[sKey] = { name: `Semester ${sN}`, mapels: {} };
  }
  if (!menuTree[kKey].semesters[sKey].mapels[mKey]) {
    const mName = mKey.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ');
    const mIcon = mapelIcons[mKey] || 'ph-book';
    menuTree[kKey].semesters[sKey].mapels[mKey] = { name: mName, icon: mIcon, lessons: [] };
  }

  menuTree[kKey].semesters[sKey].mapels[mKey].lessons.push({
    id: item.id,
    title: item.data.title || parts[3] || item.id,
    active: item.id === entry.id
  });
});
---

<Layout title={`${entry.data.title} - Modul Sekolah`}>
    
    <!-- SIDEBAR NAVIGATION DINAMIS -->
    <aside id="sidebar" slot="sidebar" class="bg-theme-panel border-r border-theme-border w-72 h-full flex flex-col absolute lg:relative z-50 transform -translate-x-full lg:translate-x-0 sidebar-transition shadow-2xl lg:shadow-none">
        <div class="h-16 flex items-center px-6 border-b border-theme-border justify-between">
            <div class="flex items-center gap-2 font-bold text-lg text-theme-accent">
                <i class="ph ph-books text-2xl"></i>
                <span>Modul Sekolah</span>
            </div>
            <button onclick="toggleSidebar()" class="lg:hidden text-theme-muted hover:text-theme-text">
                <i class="ph ph-x text-xl"></i>
            </button>
        </div>

        <div class="flex-1 overflow-y-auto p-4 space-y-1" id="menu-container">
            {Object.keys(menuTree).map((kKey) => {
                const kelas = menuTree[kKey];
                const isCurrentKelas = kKey === `kelas-${kelasNum}`;
                return (
                    <div class="mb-2">
                        <button onclick={`toggleAccordion('${kKey}')`} class="w-full flex items-center justify-between p-3 rounded-lg text-theme-text hover:bg-theme-bg transition font-medium">
                            <div class="flex items-center gap-3">
                                <i class="ph ph-graduation-cap text-lg text-theme-muted"></i>
                                {kelas.name}
                            </div>
                            <i class="ph ph-caret-down text-sm transition-transform duration-300" id={`icon-${kKey}`} style={isCurrentKelas ? 'transform: rotate(180deg)' : ''}></i>
                        </button>

                        <div id={kKey} class={isCurrentKelas ? "pl-4 mt-1 space-y-1" : "hidden pl-4 mt-1 space-y-1"}>
                            {Object.keys(kelas.semesters).map((sKey) => {
                                const semester = kelas.semesters[sKey];
                                const semId = `${kKey}-${sKey}`;
                                const isCurrentSem = isCurrentKelas && sKey === `semester-${semNum}`;
                                return (
                                    <div class="mb-1">
                                        <button onclick={`toggleAccordion('${semId}')`} class="w-full flex items-center justify-between p-2 rounded-lg text-sm text-theme-muted hover:text-theme-text hover:bg-theme-bg transition">
                                            <span>{semester.name}</span>
                                            <i class="ph ph-caret-down text-xs transition-transform duration-300" id={`icon-${semId}`} style={isCurrentSem ? 'transform: rotate(180deg)' : ''}></i>
                                        </button>

                                        <div id={semId} class={isCurrentSem ? "pl-3 mt-1 space-y-1" : "hidden pl-3 mt-1 space-y-1"}>
                                            {Object.keys(semester.mapels).map((mKey) => {
                                                const mapel = semester.mapels[mKey];
                                                const isCurrentMapel = isCurrentSem && mKey === mapelRaw;
                                                return (
                                                    <div class="mb-1">
                                                        <div class="px-2 py-1 text-xs font-semibold text-theme-muted uppercase tracking-wider flex items-center gap-2">
                                                            <i class={`ph ${mapel.icon} text-sm text-theme-accent`}></i>
                                                            {mapel.name}
                                                        </div>
                                                        <div class="pl-2 space-y-0.5 border-l-2 border-theme-border ml-2">
                                                            {mapel.lessons.map((les) => (
                                                                <a 
                                                                    href={`/${les.id}`} 
                                                                    class={`block p-1.5 text-xs rounded transition ${les.active ? 'bg-theme-accent/10 text-theme-accent font-bold border-l-2 border-theme-accent -ml-[2px]' : 'text-theme-muted hover:text-theme-text hover:bg-theme-bg'}`}
                                                                >
                                                                    {les.title}
                                                                </a>
                                                            ))}
                                                        </div>
                                                    </div>
                                                );
                                            })}
                                        </div>
                                    </div>
                                );
                            })}
                        </div>
                    </div>
                );
            })}
        </div>

        <div class="p-4 border-t border-theme-border text-xs text-theme-muted text-center">
            &copy; 2026 Modul Pembelajaran Interaktif
        </div>
    </aside>

    <!-- AREA ARTIKEL KONTEN -->
    <div class="max-w-4xl mx-auto pb-20">
        
        <!-- Breadcrumb Navigasi -->
        <nav class="flex items-center text-sm text-theme-muted font-medium mb-6 bg-theme-panel px-4 py-2 rounded-lg border border-theme-border w-max shadow-sm">
            <i class="ph ph-house mr-2 text-theme-accent"></i>
            <span class="hover:text-theme-accent cursor-pointer transition">Kelas {kelasNum}</span>
            <i class="ph ph-caret-right mx-2 text-xs"></i>
            <span class="hover:text-theme-accent cursor-pointer transition">Semester {semNum}</span>
            <i class="ph ph-caret-right mx-2 text-xs"></i>
            <span class="text-theme-text font-semibold">{mapelLabel}</span>
        </nav>

        <!-- Header Artikel -->
        <div class="mb-8 border-b border-theme-border pb-6">
            <div class="inline-block px-3 py-1 bg-theme-accent text-white rounded-full text-xs font-semibold mb-3 tracking-wide">
                {entry.data.bab || `MODUL - ${mapelLabel.toUpperCase()}`}
            </div>
            <h1 class="text-3xl md:text-4xl font-bold mb-4 text-theme-text transition-theme leading-tight">
                {entry.data.title}
            </h1>
            <div class="flex items-center gap-4 text-sm text-theme-muted">
                <div class="flex items-center gap-1"><i class="ph ph-user"></i> {entry.data.author}</div>
                <div class="flex items-center gap-1"><i class="ph ph-clock"></i> Estimasi baca: {entry.data.readTime}</div>
            </div>
        </div>

        <!-- Render Isi Artikel Markdown -->
        <article id="artikel-konten" class="text-base text-theme-text leading-relaxed space-y-6 transition-theme">
            <Content />
        </article>

        <!-- NAVIGASI SEBELUMNYA / SELANJUTNYA DINAMIS -->
        <div class="mt-16 flex flex-col sm:flex-row justify-between items-stretch gap-6 pt-8 border-t border-theme-border">
            
            <!-- Tombol Sebelum -->
            {prev ? (
                <a href={`/${prev.id}`} class="flex-1 flex items-center justify-start gap-4 p-4 rounded-xl border-2 border-theme-border bg-theme-bg hover:bg-theme-panel hover:border-theme-muted text-left transition group">
                    <div class="p-3 bg-theme-panel rounded-lg border border-theme-border group-hover:border-theme-muted transition">
                        <i class="ph ph-arrow-left text-xl text-theme-muted group-hover:text-theme-accent"></i>
                    </div>
                    <div>
                        <div class="text-xs font-bold text-theme-muted uppercase tracking-wider mb-1">Materi Sebelumnya</div>
                        <div class="font-semibold text-theme-text line-clamp-1 text-sm md:text-base">{prev.data.title}</div>
                    </div>
                </a>
            ) : (
                <div class="flex-1 p-4 rounded-xl border border-dashed border-theme-border text-xs text-theme-muted flex items-center justify-center">
                    Materi Awal
                </div>
            )}
            
            <!-- Tombol Selanjutnya -->
            {next ? (
                <a href={`/${next.id}`} class="flex-1 flex items-center justify-end gap-4 p-4 rounded-xl border-2 border-theme-accent bg-theme-accent text-white hover:bg-theme-hover hover:border-theme-hover text-right transition shadow-lg group">
                    <div>
                        <div class="text-xs font-bold text-white/80 uppercase tracking-wider mb-1">Materi Selanjutnya</div>
                        <div class="font-semibold line-clamp-1 text-sm md:text-base">{next.data.title}</div>
                    </div>
                    <div class="p-3 bg-white/20 rounded-lg group-hover:bg-white/30 transition border border-white/30">
                        <i class="ph ph-arrow-right text-xl"></i>
                    </div>
                </a>
            ) : (
                <div class="flex-1 p-4 rounded-xl border border-dashed border-theme-border text-xs text-theme-muted flex items-center justify-center">
                    Materi Akhir
                </div>
            )}

        </div>

    </div>
</Layout>
```

---

### E. Halaman Beranda Utama (`src/pages/index.astro`)

Halaman beranda ini secara otomatis mengarahkan pengunjung ke materi pelajaran pertama yang tersedia dalam sistem.

```astro
---
import { getCollection } from 'astro:content';

const allMateri = await getCollection('materi');

if (allMateri.length > 0) {
  allMateri.sort((a, b) => a.id.localeCompare(b.id));
  return Astro.redirect(`/${allMateri[0].id}`);
}
---

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Modul Sekolah - Belum Ada Materi</title>
</head>
<body style="font-family: sans-serif; text-align: center; padding: 50px;">
    <h1>Modul Pembelajaran Interaktif</h1>
    <p>Belum ada materi <code>.md</code> di dalam folder <code>src/content/</code>.</p>
    <p>Silakan tambahkan file materi untuk mulai membaca.</p>
</body>
</html>
```

---

### F. Contoh Format File Materi Markdown (`src/content/kelas-1/semester-1/akhlak/pelajaran-1.md`)

Berikut adalah contoh format penulisan file Markdown yang mendukung teks Arab dan styling box seperti di **Contoh Tampilan Web**:

```markdown
---
title: "Adab dan Keutamaan Menuntut Ilmu"
author: "Ustadz Pembimbing"
bab: "BAB 1 - AKHLAK"
readTime: "5 menit"
---

Segala puji bagi Allah, Rabb semesta alam. Menuntut ilmu merupakan salah satu ibadah yang paling agung.

<div class="bg-theme-panel border-l-4 border-theme-accent p-6 rounded-r-xl shadow-sm my-8 transition-theme">
    <h3 class="text-lg font-semibold mb-4 flex items-center gap-2">
        <i class="ph ph-bookmark-simple text-theme-accent"></i> Dalil Menuntut Ilmu
    </h3>
    
    <div class="teks-arab font-arabic text-3xl md:text-4xl text-theme-text mb-6">
        طَلَبُ الْعِلْمِ فَرِيضَةٌ عَلَى كُلِّ مُسْلِمٍ
    </div>
    
    <p class="italic text-theme-muted text-center text-sm md:text-base border-t border-theme-border pt-4">
        "Menuntut ilmu itu wajib atas setiap muslim."
    </p>
</div>

## Adab-Adab Penuntut Ilmu

- **1. Ikhlas Karena Allah ﷻ**: Niatkan belajar murni untuk mengangkat kebodohan dari diri sendiri dan orang lain.
- **2. Bersungguh-sungguh**: Jangan bermalas-malasan.
- **3. Menghormati Guru**: Guru adalah jembatan ilmu.

<div class="mt-8">
    <h2 class="text-2xl font-bold mb-4">Doa Memohon Ilmu</h2>
    
    <div class="teks-arab font-arabic text-3xl md:text-4xl text-theme-accent mt-6 mb-4">
        اللَّهُمَّ إِنِّي أَسْأَلُكَ عِلْمًا نَافِعًا
    </div>
    <p class="text-theme-muted italic text-center">
        "Ya Allah, sungguh aku memohon kepada-Mu ilmu yang bermanfaat."
    </p>
</div>
```

---

## 4. Cara Penambahan Materi Baru di Masa Mendatang

Untuk menambah materi baru di kemudian hari, Anda cukup membuat file Markdown baru di dalam struktur folder yang sudah ada tanpa perlu mengubah kode Astro:

1. **Format Nama & Lokasi File**:
   Simpan file `.md` baru mengikuti struktur: `src/content/[kelas]/[semester]/[mapel]/[nama-file].md`  
   *Contoh*: `src/content/kelas-1/semester-1/akhlak/pelajaran-2.md`
2. **Isi Frontmatter Sesuai Standar**:
   ```yaml
   ---
   title: "Adab Kepada Orang Tua"
   author: "Ustadz Pembimbing"
   bab: "BAB 2 - AKHLAK"
   readTime: "6 menit"
   ---
   ```
3. **Deteksi Otomatis**:
   Begitu file disimpan, Astro v5 secara otomatis akan:
   - Menambahkan menu pelajaran baru di sidebar bawah kelas, semester, dan mata pelajaran yang sesuai.
   - Menghubungkan link navigasi sidebar secara presisi ke URL `/[slug]`.
   - Mengurutkan tombol **Materi Selanjutnya** pada pelajaran sebelumnya agar mengarah ke materi baru tersebut.

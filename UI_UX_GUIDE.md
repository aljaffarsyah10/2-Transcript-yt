# UI/UX GUIDE & DESIGN SYSTEM — transscipt-yt

**Document Version:** 1.0.0  
**Phase:** Phase 3 — UI/UX Planning & Design System  
**Project:** `transscipt-yt` (YouTube Transcript Converter)  
**Status:** Approved for Implementation

---

## 1. Design Philosophy & Direction

`transscipt-yt` dirancang dengan pendekatan **Modern Minimalist Productivity Tool**:
* **Readability-First:** Konten utama adalah teks transcript panjang. Tipografi, tinggi baris (*line-height*), dan kontras warna dioptimalkan untuk kenyamanan membaca tanpa distraksi.
* **Instant Clarity:** Pengguna langsung memahami cara kerja aplikasi dalam 3 detik pertama: paste URL YouTube → klik tombol / tekan Enter → salin transcript.
* **Low Cognitive Load:** Tidak ada registrasi wajib, tidak ada konfigurasi rumit, tidak ada elemen dekoratif yang mengganggu fokus kerja.
* **Responsive & Fluid:** Berfungsi mulus di desktop layar lebar, laptop, tablet, hingga smartphone layar kecil tanpa *horizontal overflow*.

---

## 2. Information Architecture & Navigation

Aplikasi berupa **Single Page Application (SPA)** yang terstruktur secara vertikal:

```
+-------------------------------------------------------------+
| Navbar (Sticky/Fixed top)                                   |
| - Logo & Nama Aplikasi ("transscipt-yt")                    |
| - Tagline / Quick Status / GitHub Link                      |
+-------------------------------------------------------------+
| Main Content Area (Max width: 896px / max-w-4xl, centered)  |
|                                                             |
| 1. Hero Section                                             |
|    - Main Headline: "Extract Subtitles & Transcripts..."    |
|    - Subheadline: Panduan singkat & format URL didukung     |
|                                                             |
| 2. Input Form Card                                          |
|    - YouTube URL Input (auto-focus, clear button, icon)     |
|    - Primary Action Button ("Get Transcript")               |
|    - URL validation message / helper chips                  |
|                                                             |
| 3. Dynamic State Area                                       |
|    - State A: Empty/Default Guide State                     |
|    - State B: Skeleton Loading State                        |
|    - State C: Error State (Alert with Retry Button)         |
|    - State D: Success Transcript Result Card                |
|        * Card Header: Video info, Language badge, Word count|
|        * Card Toolbar: Copy Full, Download TXT, Clear       |
|        * Card Body: Scrollable transcript reading container |
+-------------------------------------------------------------+
| Footer                                                      |
| - Disclaimer (YouTube copyright & privacy note)             |
| - Open source & attribution credits                         |
+-------------------------------------------------------------+
```

---

## 3. Wireframes

### 3.1 Desktop Wireframe (Viewport >= 1024px)

```
+-----------------------------------------------------------------------------------+
|  [▶ transscipt-yt]                                           [GitHub] [Theme]     |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|                              Convert YouTube to Text                              |
|             Extract full transcripts and subtitles instantly with one click.       |
|                                                                                   |
|        +------------------------------------------------------------------+       |
|        | [icon] https://www.youtube.com/watch?v=...     [x] | [Get Transcript] | |
|        +------------------------------------------------------------------+       |
|        Supported formats: youtube.com/watch, youtu.be, youtube.com/shorts         |
|                                                                                   |
|        +------------------------------------------------------------------+       |
|        | [Badge: English (en)] [Auto-generated]             [642 words]   |       |
|        |                                                                  |       |
|        | [📋 Copy Transcript]   [📥 Download .txt]        [🔄 Clear]      |       |
|        +------------------------------------------------------------------+       |
|        | [00:00] Hello everyone, welcome back to another video tutorial.  |       |
|        | [00:04] Today we are going to explore how modern web APIs work.  |       |
|        | [00:09] First, let's look at the basic requirements...           |       |
|        |                                                                  |       |
|        | (Scrollable area, max-height: 480px)                             |       |
|        |                                                                  |       |
|        +------------------------------------------------------------------+       |
|                                                                                   |
+-----------------------------------------------------------------------------------+
|  (C) 2026 transscipt-yt • Captions extracted via YouTube accessible timedtext     |
+-----------------------------------------------------------------------------------+
```

### 3.2 Mobile Wireframe (Viewport < 640px)

```
+-----------------------------------------------+
| [▶ transscipt-yt]                    [GitHub] |
+-----------------------------------------------+
|                                               |
|           Convert YouTube to Text             |
|      Extract transcripts in seconds.          |
|                                               |
| +-------------------------------------------+ |
| | [icon] https://youtu.be/...           [x] | |
| +-------------------------------------------+ |
| | [        Get Transcript Button          ] | |
| +-------------------------------------------+ |
|                                               |
| +-------------------------------------------+ |
| | [Badge: en] [Auto]            [642 words] | |
| |                                           | |
| | [📋 Copy]      [📥 TXT]          [🔄 New] | |
| +-------------------------------------------+ |
| | [00:00] Hello everyone, welcome back...   | |
| | [00:04] Today we are going to explore...  | |
| |                                           | |
| | (Scrollable area, max-height: 380px)      | |
| +-------------------------------------------+ |
|                                               |
| (C) 2026 transscipt-yt                        |
+-----------------------------------------------+
```

---

## 4. Design Tokens & Styling System

Menggunakan Tailwind CSS v4 design tokens yang terintegrasi dengan CSS Variables shadcn/ui.

### 4.1 Color Palette

| Token | CSS Variable / Tailwind | Hex Value (Approx) | Penggunaan |
| :--- | :--- | :--- | :--- |
| **Primary** | `var(--primary)` / `indigo-600` | `#4F46E5` | Tombol CTA utama, link aktif, accent border |
| **Primary Hover** | `indigo-700` | `#4338CA` | Hover state tombol utama |
| **Background** | `var(--background)` | `#F8FAFC` (Slate 50) | Latar belakang seluruh halaman |
| **Surface / Card** | `var(--card)` | `#FFFFFF` | Latar belakang container form & result |
| **Text Primary** | `var(--foreground)` | `#0F172A` (Slate 900) | Judul utama, teks transcript, heading |
| **Text Muted** | `var(--muted-foreground)`| `#64748B` (Slate 500) | Subtitle, label timestamp, disclaimer |
| **Border** | `var(--border)` | `#E2E8F0` (Slate 200) | Garis pemisah, border input, border card |
| **Success** | `emerald-600` | `#059669` | Copy success alert, badge validasi |
| **Destructive / Error**| `var(--destructive)` / `red-600`| `#DC2626` | Pesan error validasi, alert kegagalan |
| **Warning** | `amber-600` | `#D97706` | Status caption auto-generated |

### 4.2 Typography System

* **Primary Font:** Inter / Geist Sans (variabel font sans-serif bawaan Next.js).
* **Monospace Font:** Geist Mono / JetBrains Mono (digunakan untuk timestamp dan video ID display).

| Skala Tipografi | Class Tailwind | Ukuran / Line-Height | Weight | Penggunaan |
| :--- | :--- | :--- | :--- | :--- |
| **Display Title** | `text-3xl md:text-4xl` | 32px / 40px (1.2) | Bold (700) | Judul Hero Section |
| **Section Subtitle**| `text-base md:text-lg` | 16px / 28px (1.6) | Regular (400) | Penjelasan di bawah judul |
| **Transcript Body** | `text-base leading-relaxed`| 16px / 28px (1.75) | Regular (400) | Paragraf teks transcript |
| **Timestamps** | `font-mono text-xs` | 12px / 16px | Medium (500) | Indikator waktu `[00:15]` |
| **Button Text** | `text-sm font-medium` | 14px / 20px | Medium (500) | Tombol CTA dan toolbar |
| **Captions / Meta** | `text-xs text-muted-foreground` | 12px / 16px | Regular (400) | Word count, language tag |

### 4.3 Spacing & Layout Rhythm

* **Page Container:** `max-w-4xl mx-auto px-4 sm:px-6 lg:px-8`
* **Section Gap:** `space-y-8` (32px vertical rhythm antar komponen)
* **Card Padding:** `p-5 sm:p-7` (padding nyaman di dalam card)
* **Input-Button Gap:** `gap-3` pada desktop (baris), `space-y-3` pada mobile (vertikal)

### 4.4 Elevation & Border Radius

* **Radius:** `rounded-xl` (12px) untuk Card dan container besar; `rounded-lg` (8px) untuk Input dan Button.
* **Shadow:** `shadow-xs` untuk input; `shadow-sm` untuk Card default; `shadow-md` untuk state melayang/hover.

---

## 5. Detailed Component Specifications

### 5.1 Navbar (`src/components/layout/navbar.tsx`)
* **Struktur:** Logo icon YouTube warna merah (`#FF0000`), teks brand `transscipt-yt` semibold, dan badge `v1.0 (MVP)`.
* **Right item:** Tombol shortcut GitHub repository dengan ikon Lucide.

### 5.2 Hero Section (`src/features/transcript/components/hero-section.tsx`)
* **Heading:** *"Convert YouTube to Clean Text"*
* **Subheading:** *"Extract full transcripts and subtitles instantly with one click. Read, search, and copy with ease."*
* **Tags pendukung:** Chips kecil menunjukkan format yang didukung: `Standard URL`, `Shorts`, `youtu.be`.

### 5.3 YouTube Input Form (`src/features/transcript/components/youtube-input-form.tsx`)
* **Input Field:**
  * Placeholder: *"Paste YouTube link here (e.g. https://www.youtube.com/watch?v=...)"*
  * Ikon prefix: Link / Search icon.
  * Clear button: Tombol silang `(X)` muncul otomatis jika input terisi.
  * Auto paste button (opsional bila diizinkan browser).
* **Submit Button:**
  * Teks default: *"Get Transcript"*
  * Teks saat loading: *"Fetching..."* disertai spinning loader icon.
  * State disabled ketika input kosong atau proses fetch sedang berlangsung.

### 5.4 Transcript Result View (`src/features/transcript/components/transcript-result.tsx`)
* **Result Header:**
  * Language badge: misal `"English (en)"` atau `"Indonesian (id)"`.
  * Caption type badge: `"Manual Caption"` atau `"Auto-generated"`.
  * Statistik ringkas: Jumlah kata (*words count*) & jumlah segmen.
* **Toolbar Actions:**
  * **Copy Button:** Salin seluruh teks ke clipboard, berubah menjadi centang hijau *"Copied!"* selama 2 detik.
  * **Download TXT Button:** Mengunduh isi teks sebagai file `.txt` (`transcript-{videoId}.txt`).
  * **Toggle Timestamp Button:** Pilihan menampilkan teks bergaya narasi paragraf vs berformat waktu baris per baris `[MM:SS]`.
* **Content Container:**
  * Box dengan latar `bg-muted/30`, border `border-border`, `rounded-lg`.
  * Tinggi terkontrol: `max-h-[480px] overflow-y-auto` dengan scrollbar halus.
  * Teks selectable dan ramah pembaca.

---

## 6. UI State Matrix

| State | Kondisi Pemicu | Tampilan Visual & Perilaku |
| :--- | :--- | :--- |
| **1. Empty State** | Halaman baru dimuat, belum ada input URL | Kartu tips sederhana dengan 3 langkah mudah: *"1. Copy YouTube URL → 2. Paste di atas → 3. Dapatkan teks lengkap"*. Tidak menampilkan kotak abu-abu kosong yang membingungkan. |
| **2. Loading State** | Tombol submit diklik, request API sedang berlangsung | - Tombol submit berubah menjadi disabled + spinner.<br>- Area hasil menampilkan `Skeleton` beranimasi shimmer (kartu header skeleton + 5 baris paragraf skeleton).<br>- Teks indikator: *"Mengambil subtitle dari YouTube..."* |
| **3. Success State** | API mengembalikan status 200 dengan payload transcript | - Kartu hasil muncul dengan animasi fade-in halus.<br>- Metadata video/bahasa tampil lengkap.<br>- Toolbar aktif (Copy, Download, Clear).<br>- Fokus visual diarahkan ke area hasil. |
| **4. Error State** | Validasi gagal, video tanpa caption, atau provider error | - Alert box dengan varian `destructive` (border merah lembut, background merah muda).<br>- Ikon AlertTriangle merah.<br>- Judul error singkat + pesan penjelasan yang bersahabat.<br>- Tombol aksi *"Try Again"* jika terjadi timeout/kegagalan sementara. |

---

## 7. Responsive Behavior Guidelines

### Breakpoints:
* **Mobile (< 640px):**
  * Input dan Tombol berada pada baris terpisah (vertikal, `w-full`).
  * Toolbar result card menggunakan grid 2x2 atau baris tombol compact.
  * Padding container `px-4`.
  * Maksimal tinggi scroll transcript `360px` agar muat di layar handphone.
* **Tablet (640px - 1023px):**
  * Input dan tombol menyatu dalam satu baris fleksibel (`flex-row`).
  * Layout tombol toolbar berada dalam satu baris rapi.
* **Desktop (>= 1024px):**
  * Container terpusat dengan `max-w-4xl`.
  * Padding dan font optimal untuk monitor kerja.
  * Maksimal tinggi scroll transcript `500px`.

---

## 8. Accessibility & Usability (WCAG 2.1 AA)

1. **Color Contrast:** Rasio kontras teks utama (`Slate 900`) terhadap background (`White / Slate 50`) adalah minimal **12:1** (jauh melampaui standar AA 4.5:1).
2. **Keyboard Navigation:** Seluruh kontrol (Input field, tombol submit, tombol copy, tombol download) memiliki status `:focus-visible` yang jelas dengan ring outline kontras tinggi.
3. **Screen Reader Feedback:**
   * Notifikasi status loading dan pesan sukses copy menggunakan `aria-live="polite"`.
   * Tombol ikon menyertakan `aria-label` eksplisit (misal `aria-label="Copy full transcript"`).
4. **Semantics:** Menggunakan elemen HTML5 semantik (`<header>`, `<main>`, `<section>`, `<article>`, `<footer>`).

---

## 9. Alignment with PRD & Technical Architecture

* Desain ini 100% konsisten dengan kebutuhan P0 dan P1 pada **PRD Section 10** (`Minimalist, Readability-first, Responsive`).
* Desain mengintegrasikan seluruh komponen UI shadcn/ui yang telah terpasang di Phase 1: `Button`, `Input`, `Card`, `Textarea`, `Alert`, `Skeleton`.
* Mempersiapkan arsitektur komponen frontend modular yang akan diimplementasikan pada **Phase 4 & Phase 5**.

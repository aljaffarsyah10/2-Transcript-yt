# RELEASE NOTES — transscipt-yt

**Version:** 1.0.0 (MVP)  
**Release Date:** 2026-11-03  
**Status:** General Availability (GA) — Production Ready

---

## 1. Product Summary

**transscipt-yt** adalah aplikasi web cepat, gratis, dan ramah privasi untuk mengambil dan menyalin teks transcript atau subtitle dari video YouTube hanya dengan memasukkan URL video. 

Aplikasi ini beroperasi tanpa registrasi akun, tanpa database permanen, dan tanpa proses speech-to-text yang lambat—melainkan mengambil caption resmi YouTube yang tersedia secara instan dalam hitungan milidetik.

---

## 2. Key Features Included in v1.0.0

### Core MVP Capabilities (P0):
* **Multi-Format YouTube URL Parser:**
  * Mendukung `youtube.com/watch?v=...`, `youtu.be/...`, `youtube.com/shorts/...`, `youtube.com/embed/...`, `m.youtube.com`, dan input raw 11-char video ID.
  * Menangani parameter query tambahan (`&t=...`, `&list=...`, `?si=...`) secara cerdas.
* **Instant Transcript Extraction:**
  * Pengambilan subtitle berbasis serverless Route Handler dalam waktu < 1 detik.
  * Normalisasi data ke format teks naratif dan segmen berwaktu (*seconds offset*).
* **Interactive Results & Clipboard Tool:**
  * Tombol **Copy All** dengan feedback visual seketika ("Copied!" dengan centang hijau).
  * Pengunduhan instan berkas teks **.txt** langsung dari browser pengguna.
* **Comprehensive Error Handling:**
  * Penanganan kondisi kegagalan yang ramah pengguna (video tanpa caption, video private, ID tidak valid, timeout, dan rate limiting).
* **Modern Readability-First UI:**
  * Desain minimalis terfokus pada kenyamanan membaca teks panjang (*line-height 1.75*).
  * 100% responsif di desktop, tablet, dan smartphone layar kecil.

### P1 Supporting Capabilities:
* **Adaptive Timestamp Formatting:**
  * Menampilkan format `[MM:SS]` atau `[HH:MM:SS]` untuk video berdurasi panjang / podcast.
* **Per-Segment Copying:**
  * Tombol copy individual pada setiap baris kalimat saat mode timestamp diaktifkan.
* **Language Selection Filter:**
  * Pilihan bahasa caption (`Default`, `English`, `Indonesian`, `Spanish`, `French`, `German`) dengan re-fetch reaktif.
* **Video Preview Card:**
  * Pratinjau thumbnail resmi YouTube dan tautan cepat untuk membuka video asli.
* **Search Engine Optimization (SEO):**
  * Metadata OpenGraph, Twitter Card, dinamis `robots.txt`, dan `sitemap.xml`.

---

## 3. Architecture & Technical Highlights

* **Framework:** Next.js 16+ App Router dengan arsitektur **Modular Monolith** dan **Hexagonal/Adapter Pattern**.
* **Zero Database & Privacy First:** Tidak menyimpan URL, histori pencarian, atau data pengguna.
* **In-Memory Rate Limiter:** Sliding window limiter (15 request/menit/IP) untuk mencegah abuse.
* **SSRF Hardened:** Backend hanya menerima parameter 11 karakter video ID (`^[a-zA-Z0-9_-]{11}$`).
* **Zero Dependency Bloat:** Tidak membutuhkan Puppeteer/Chromium, FFmpeg, atau runtime Python terpisah.

---

## 4. Test & Quality Metrics

* **Automated Unit & Integration Test Suite:** 21/21 tests passed (100% passing rate).
* **TypeScript Type Checking:** 0 errors.
* **ESLint Code Quality:** 0 errors, 0 warnings.
* **Live Scraper Verification:** Berhasil diuji pada video nyata YouTube (`jNQXAC9IVRw` & `dQw4w9WgXcQ`).

---

## 5. Known Limitations & Future Roadmap (P2)

### Known Limitations:
1. Video yang **tidak memiliki caption/subtitle** (baik manual maupun auto-caption dari YouTube) tidak dapat menghasilkan transcript. Aplikasi akan menampilkan alert informatif bahwa subtitle tidak tersedia.
2. Pengambilan transcript bergantung pada akses publik timedtext YouTube.

### Future Roadmap (P2):
* AI Transcript Summarization (ringkasan poin-poin penting video).
* AI Translation ke bahasa yang tidak didukung oleh subtitle asli YouTube.
* Ekspor berkas PDF dan DOCX dengan format transkripsi profesional.
* Riwayat pencarian lokal di peramban (Local Storage).

---

## 6. How to Run Locally

```bash
# Clone repository
git clone https://github.com/<username>/transscipt-yt.git
cd transscipt-yt/transscipt-yt

# Install dependencies
npm install

# Run development server
npm run dev
# Buka http://localhost:3000

# Run automated test suite
npm test

# Run production build
npm run build
npm start
```

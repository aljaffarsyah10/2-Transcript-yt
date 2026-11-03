Berikut dokumen PRD & Sequential Development Playbook untuk project `transscipt-yt` (YouTube Transcript Converter).

Dokumen ini dirancang agar bisa langsung disimpan sebagai `PROJECT_PRD_AND_PLAYBOOK.md` di root repository dan digunakan secara bertahap menggunakan VS Code, Cursor, Claude Code, GitHub Copilot, atau AI Coding Assistant lainnya.

Ada beberapa keputusan arsitektur penting untuk project ini:

1. Next.js menjadi frontend sekaligus backend utama menggunakan App Router dan Route Handlers.
2. Tidak membutuhkan database untuk MVP karena aplikasi hanya mengambil transcript dan menampilkannya kepada pengguna.
3. Transcript diambil dari caption/subtitle YouTube yang tersedia, bukan melakukan speech-to-text dari audio.
4. Pengambilan transcript dibuat melalui adapter/service terpisah agar library atau provider dapat diganti jika terjadi perubahan pada mekanisme YouTube.
5. Axios dan Zustand tidak diwajibkan. Gunakan native `fetch` dan React state terlebih dahulu.
6. Tidak menggunakan Rust, Tauri, microservices, atau Docker untuk tahap MVP.
7. Aplikasi harus memiliki error handling untuk video yang tidak memiliki caption, video private, URL tidak valid, dan kegagalan mengambil transcript.

Catatan teknis: YouTube tidak menyediakan endpoint publik sederhana yang memungkinkan aplikasi mengambil semua caption video publik tanpa batasan. API resmi YouTube untuk caption memiliki persyaratan autentikasi dan izin tertentu. Karena itu, integrasi pengambilan transcript harus divalidasi pada Phase 2 sebelum menentukan library/provider yang akan digunakan untuk production.

# PROJECT_PRD_AND_PLAYBOOK.md

# PART A — PRODUCT REQUIREMENTS DOCUMENT (PRD)

## 1. Product Information

### 1.1 Product Identity

| Item                 | Description                           |
| -------------------- | ------------------------------------- |
| Project Name         | transscipt-yt                         |
| Product Type         | YouTube Transcript Converter          |
| Platform             | Web Application                       |
| Target Users         | Public                                |
| Development Approach | Solo Developer + AI Coding Assistant  |
| Development Strategy | MVP First                             |
| Architecture         | Modular Monolith                      |
| Frontend             | Next.js App Router                    |
| Language             | TypeScript                            |
| Styling              | Tailwind CSS                          |
| UI Components        | shadcn/ui                             |
| State Management     | React State (Zustand jika diperlukan) |
| Backend              | Next.js Route Handlers                |
| HTTP Client          | Native Fetch                          |
| Database             | Not required for MVP                  |
| Deployment           | Vercel atau VPS                       |
| Version Control      | Git                                   |

### 1.2 Product Description

transscipt-yt adalah aplikasi berbasis web yang memungkinkan pengguna mendapatkan transcript atau subtitle dari sebuah video YouTube hanya dengan memasukkan URL video.

Transcript yang berhasil diperoleh akan ditampilkan dalam bentuk teks yang dapat:

- Dibaca langsung di website.
- Disalin seluruhnya ke clipboard.
- Disalin per bagian jika diperlukan.
- Dipilih bahasa transcript jika tersedia.
- Digunakan kembali untuk kebutuhan catatan, dokumentasi, pembelajaran, dan penelitian.

### 1.3 Project Constraints

- Pengembangan dilakukan oleh satu developer.
- AI Coding Assistant digunakan untuk membantu implementasi.
- Fokus pada MVP yang dapat digunakan publik.
- Tidak membutuhkan registrasi pengguna.
- Tidak membutuhkan database pada MVP.
- Tidak menyimpan transcript secara permanen.
- Hanya mengambil caption/subtitle yang tersedia.
- Tidak melakukan download video atau audio.
- Tidak melakukan speech-to-text mandiri.
- Tidak mengakses video private atau melewati proteksi akses YouTube.
- Penggunaan library pihak ketiga harus divalidasi terlebih dahulu.

## 2. Product Overview

### 2.1 Problem Statement

Pengguna YouTube sering membutuhkan isi pembicaraan dari sebuah video dalam bentuk teks.

Beberapa permasalahan yang umum terjadi:

1. Pengguna harus membuka YouTube dan mencari fitur transcript secara manual.
2. Tidak semua pengguna mengetahui cara menampilkan transcript di YouTube.
3. Menyalin transcript dari YouTube terkadang kurang praktis.
4. Pengguna membutuhkan teks untuk membuat catatan atau dokumentasi.
5. Pengguna ingin membaca isi video tanpa harus menonton keseluruhan video.
6. Pengguna membutuhkan cara cepat untuk mendapatkan teks dari video yang memiliki subtitle.

### 2.2 Proposed Solution

Membangun website sederhana yang dapat mengambil transcript dari video YouTube berdasarkan URL.

Pengguna cukup:

1. Membuka website.
2. Memasukkan link YouTube.
3. Menekan tombol Get Transcript.
4. Menunggu proses pengambilan transcript.
5. Melihat hasil transcript.
6. Menyalin teks menggunakan tombol Copy.

### 2.3 Product Workflow

[How to Translate a YouTube Video with AI (2026 Reality Check) | ConvertAudioToText](https://images.openai.com/static-rsc-4/yxvaXVYhfkezDZSNnkrcfux70BwVNotHVfQawLq7up2yAyCm4Bd08LP8mbT3VHnF3N3FXEPHRLND8nqQQCHoyQ1UMXdwQRtsq4emn1FnUVmkqonnFI8VVtUV7GP-XA9hVBCknKpuJ6S05NWpF6rvo2wwFLQbwp-E5Sxw8wPznKU?purpose=inline)

[convertaudiototext.com](https://convertaudiototext.com/blog/translate-youtube-video-with-ai?utm_source=chatgpt.com)

Alur sistem:

```
User
  |
  v
Open Website
  |
  v
Input YouTube URL
  |
  v
Validate URL
  |
  v
Extract YouTube Video ID
  |
  v
Submit Request
  |
  v
Next.js API Route
  |
  v
Transcript Service
  |
  v
Retrieve Available Captions
  |
  v
Normalize Transcript
  |
  v
Return Transcript Data
  |
  v
Display Transcript
  |
  v
Copy to Clipboard
```

### 2.4 Core Value Proposition

- Simple: tidak memerlukan akun.
- Fast: cukup menggunakan link YouTube.
- Accessible: dapat digunakan melalui browser.
- Practical: transcript bisa langsung disalin.
- Language-aware: mendukung pemilihan bahasa yang tersedia.
- Responsive: dapat digunakan melalui desktop maupun mobile.

## 3. Product Goals

### 3.1 Primary Goals

| ID    | Goal                                 | Success Indicator                            |
| ----- | ------------------------------------ | -------------------------------------------- |
| PG-01 | Memudahkan pengambilan transcript    | User dapat memperoleh transcript melalui URL |
| PG-02 | Menyediakan interface sederhana      | Alur utama dapat diselesaikan tanpa panduan  |
| PG-03 | Menyediakan hasil yang dapat disalin | Copy transcript berfungsi                    |
| PG-04 | Mendukung berbagai format URL        | URL YouTube yang valid dapat diproses        |
| PG-05 | Menyediakan pengalaman responsive    | UI berfungsi di desktop dan mobile           |
| PG-06 | Menangani kegagalan dengan baik      | Error ditampilkan dengan pesan yang jelas    |

### 3.2 MVP Success Metrics

Target awal:

- 100% URL format yang didukung berhasil divalidasi dengan benar.
- Transcript berhasil ditampilkan ketika caption tersedia dan provider berfungsi.
- Pengguna dapat menyalin seluruh transcript.
- Tidak terdapat error kritis pada core user flow.
- Halaman utama memiliki waktu interaksi awal yang baik.
- Tidak terdapat horizontal overflow pada mobile.
- Production build berhasil tanpa error.

Target keberhasilan pengambilan transcript harus dievaluasi menggunakan kumpulan video pengujian dengan berbagai kondisi caption, bahasa, dan jenis video.

## 4. Target Users

### 4.1 General Users

Pengguna umum yang ingin mendapatkan teks dari video YouTube.

Kebutuhan:

- Memasukkan URL.
- Mendapatkan transcript.
- Menyalin hasil.

### 4.2 Students

Pelajar dan mahasiswa yang ingin membuat catatan dari materi pembelajaran.

Kebutuhan:

- Mendapatkan isi pembelajaran dalam bentuk teks.
- Menyalin materi.
- Membaca transcript tanpa menonton video berulang kali.

### 4.3 Content Creators

Content creator yang ingin mempelajari isi video atau menggunakan transcript sebagai bahan referensi.

Kebutuhan:

- Mengambil teks dengan cepat.
- Membaca keseluruhan pembicaraan.
- Menyalin bagian tertentu.

### 4.4 Researchers and Professionals

Pengguna yang membutuhkan dokumentasi isi video untuk penelitian atau pekerjaan.

Kebutuhan:

- Mendapatkan teks.
- Mempertahankan urutan pembicaraan.
- Menggunakan transcript untuk dokumentasi.

## 5. Scope & Feature Priorities

### 5.1 P0 — Core MVP

Wajib untuk MVP

| Feature             | Description                     |
| ------------------- | ------------------------------- |
| YouTube URL Input   | Input URL video YouTube         |
| URL Validation      | Validasi format URL             |
| Video ID Extraction | Mengambil ID video dari URL     |
| Get Transcript      | Mengambil transcript            |
| Transcript Display  | Menampilkan hasil transcript    |
| Copy Transcript     | Menyalin keseluruhan transcript |
| Loading State       | Menampilkan status proses       |
| Error Handling      | Menampilkan error yang relevan  |
| Language Detection  | Menampilkan bahasa transcript   |
| Responsive UI       | Mendukung desktop dan mobile    |

### 5.2 P1 — Supporting Features

Setelah core MVP

| Feature                 | Description                                         |
| ----------------------- | --------------------------------------------------- |
| Language Selection      | Memilih bahasa caption jika tersedia                |
| Timestamp Display       | Menampilkan timestamp pada transcript               |
| Copy Individual Segment | Menyalin bagian tertentu                            |
| Download TXT            | Mengunduh transcript sebagai file .txt              |
| Clear Input             | Menghapus input dan hasil                           |
| Video Metadata          | Menampilkan judul dan informasi video jika tersedia |
| SEO                     | Metadata dan SEO dasar                              |
| Theme Support           | Light dan dark mode                                 |

### 5.3 P2 — Future Development

| Feature                   | Description                         |
| ------------------------- | ----------------------------------- |
| AI Summary                | Merangkum transcript menggunakan AI |
| AI Translation            | Menerjemahkan transcript            |
| AI Chat                   | Bertanya berdasarkan isi transcript |
| Export PDF                | Mengunduh transcript sebagai PDF    |
| Export DOCX               | Mengunduh transcript sebagai DOCX   |
| Transcript History        | Menyimpan riwayat transcript        |
| User Authentication       | Login dan akun pengguna             |
| Cloud Storage             | Menyimpan transcript                |
| Multiple Video Processing | Memproses beberapa video            |
| Playlist Transcript       | Mengambil transcript dari playlist  |

### 5.4 MVP Boundary

MVP hanya berfokus pada:

```
YouTube URL
    |
    v
Transcript Retrieval
    |
    v
Transcript Display
    |
    v
Copy Transcript
```

Fitur AI, akun pengguna, database, dan penyimpanan riwayat tidak termasuk dalam implementasi awal.

## 6. Functional Requirements

### FR-01 — YouTube URL Input

Description

Sistem menyediakan input untuk memasukkan URL video YouTube.

Behavior

- Pengguna dapat memasukkan URL.
- Input menerima format YouTube yang didukung.
- Sistem menolak input kosong.
- Input dapat dihapus.

Acceptance Criteria

- Input dapat menerima URL.
- Input kosong tidak dapat diproses.
- URL tidak valid menghasilkan pesan kesalahan.
- Input dapat digunakan melalui desktop dan mobile.

### FR-02 — YouTube URL Validation

Description

Sistem memvalidasi URL sebelum mengirim permintaan ke backend.

Format yang didukung:

```
https://www.youtube.com/watch?v=VIDEO_ID

https://youtu.be/VIDEO_ID

https://www.youtube.com/shorts/VIDEO_ID

https://www.youtube.com/embed/VIDEO_ID

https://m.youtube.com/watch?v=VIDEO_ID
```

Acceptance Criteria

- URL valid dapat diproses.
- URL dari domain tidak dikenal ditolak.
- URL yang tidak memiliki video ID ditolak.
- Parameter tambahan seperti playlist dan timestamp tidak merusak proses ekstraksi.
- Sistem tidak menerima URL arbitrer untuk diteruskan sebagai request server-side.

### FR-03 — Video ID Extraction

Description

Sistem mengambil YouTube Video ID dari URL.

Acceptance Criteria

- Video ID berhasil diekstrak dari format URL yang didukung.
- Video ID divalidasi sesuai format yang diharapkan.
- Ekstraksi menggunakan utility terpisah.
- Utility memiliki unit test.

### FR-04 — Get Transcript

Description

Sistem mengambil transcript dari video YouTube.

Behavior

1. Frontend mengirim Video ID ke backend.
2. Backend memvalidasi request.
3. Backend memanggil Transcript Service.
4. Transcript Service mengambil caption yang tersedia.
5. Backend menormalisasi hasil.
6. Backend mengirimkan hasil ke frontend.

Acceptance Criteria

- Video dengan caption yang tersedia menghasilkan transcript.
- Video tanpa caption menghasilkan error yang sesuai.
- Video yang tidak dapat diakses menghasilkan error.
- Backend tidak mengembalikan stack trace kepada pengguna.
- Request memiliki batas waktu.
- Sistem tidak mengunduh audio atau video.

### FR-05 — Transcript Display

Description

Menampilkan transcript dalam area khusus.

Acceptance Criteria

- Transcript ditampilkan secara berurutan.
- Karakter dan tanda baca dipertahankan.
- Transcript panjang dapat di-scroll.
- Text dapat dipilih.
- Layout tidak rusak ketika transcript sangat panjang.

### FR-06 — Copy Transcript

Description

Menyediakan tombol untuk menyalin keseluruhan transcript.

Acceptance Criteria

- Tombol Copy tersedia setelah transcript berhasil diperoleh.
- Isi clipboard sesuai dengan transcript yang ditampilkan.
- Menggunakan Clipboard API.
- Terdapat feedback ketika proses copy berhasil.
- Kegagalan clipboard ditangani dengan baik.

### FR-07 — Loading State

Description

Menampilkan status selama proses pengambilan transcript.

Acceptance Criteria

- Loading muncul ketika request dimulai.
- Tombol submit tidak dapat diklik berulang kali ketika request berjalan.
- Loading hilang ketika request selesai atau gagal.

### FR-08 — Error Handling

Description

Sistem menangani kesalahan pengambilan transcript.

Acceptance Criteria

- Error ditampilkan dengan bahasa yang mudah dipahami.
- Input pengguna tidak langsung dihapus ketika request gagal.
- Pengguna dapat mencoba kembali.

### FR-09 — Language Detection

Description

Sistem mengidentifikasi bahasa caption yang diperoleh.

Acceptance Criteria

- Bahasa ditampilkan apabila tersedia.
- Informasi automatic atau manually created caption dipertahankan jika provider menyediakannya.
- Sistem tidak mengarang informasi bahasa.

### FR-10 — Responsive Interface

Description

Website dapat digunakan di berbagai ukuran layar.

Acceptance Criteria

- Desktop tidak mengalami layout overflow.
- Mobile memiliki input yang mudah digunakan.
- Tombol Copy dapat diakses dengan mudah.
- Area transcript dapat dibaca dengan nyaman.

## 7. Non-Functional Requirements

### 7.1 Performance

- Gunakan server-side processing untuk pengambilan transcript.
- Hindari request berulang yang tidak diperlukan.
- Terapkan timeout pada external request.
- Transcript panjang harus ditampilkan tanpa membebani rendering secara berlebihan.
- Hindari memuat library besar yang tidak diperlukan.

### 7.2 Security

- Validasi semua input di frontend dan backend.
- Validasi domain YouTube.
- Hindari SSRF.
- Terapkan rate limiting.
- Batasi ukuran request dan response.
- Jangan mengekspos API key atau secret.
- Jangan mengembalikan error internal kepada pengguna.

### 7.3 Reliability

- External provider harus dipanggil melalui adapter.
- Kegagalan provider harus menghasilkan error yang terstruktur.
- Request harus memiliki timeout.
- Sistem tidak boleh mengasumsikan semua video memiliki caption.

### 7.4 Maintainability

- Gunakan TypeScript.
- Pisahkan business logic dari UI.
- Gunakan reusable components.
- Gunakan struktur folder modular.
- Hindari duplikasi logic validasi.

### 7.5 Accessibility

- Input memiliki label.
- Tombol dapat diakses dengan keyboard.
- Status loading dapat dibaca screen reader.
- Kontras warna memadai.
- Gunakan semantic HTML.

### 7.6 Compatibility

Target browser:

- Chrome.
- Edge.
- Firefox.
- Safari.
- Browser mobile modern.

### 7.7 Privacy

- Tidak meminta akun YouTube pengguna.
- Tidak menyimpan transcript secara permanen.
- Tidak menyimpan URL ke database.
- Tidak mengumpulkan informasi pribadi yang tidak diperlukan.
- Berikan penjelasan bahwa transcript hanya tersedia jika caption dapat diakses.

## 8. Application Structure & Navigation

### 8.1 Pages

MVP cukup memiliki satu halaman utama.

| Route      | Page             | Description                    |
| ---------- | ---------------- | ------------------------------ |
| `/`        | Home             | Input URL dan hasil transcript |
| `/privacy` | Privacy Policy   | Kebijakan privasi              |
| `/terms`   | Terms of Service | Ketentuan penggunaan           |

Privacy dan Terms dapat menjadi halaman sederhana pada tahap P1 atau menjelang public release.

### 8.2 Navigation

```
Application
|
|-- Navbar
|   |-- Logo
|   |-- Home
|   |-- About
|
|-- Main Content
|   |-- Hero Section
|   |-- URL Input
|   |-- Get Transcript Button
|   |-- Result Section
|
|-- Footer
    |-- Privacy Policy
    |-- Terms
```

### 8.3 Reusable Components

| Component         | Responsibility                |
| ----------------- | ----------------------------- |
| Navbar            | Navigasi                      |
| Footer            | Informasi footer              |
| HeroSection       | Judul dan deskripsi           |
| YouTubeInput      | Input URL                     |
| TranscriptForm    | Form submission               |
| TranscriptResult  | Hasil transcript              |
| TranscriptToolbar | Tombol copy dan tindakan lain |
| LoadingState      | Status loading                |
| ErrorMessage      | Menampilkan error             |
| LanguageBadge     | Menampilkan bahasa            |

## 9. User Flow

### 9.1 Primary User Flow

[A Simple Guide to Getting Subtitles from YouTube | YoutubeToText](https://images.openai.com/static-rsc-4/ujiC_7K6KL_BNWKlCEmoVa6T-db9lZaUmfK34tg1Ir1QoPni3t7XTwE-UD5xYuhTBXQhSqTUB4av2jF5ZuYrpILRh8i-IfgzIHE9TP0WwyG4ltCwXW7B_AVlcFwA6C7pG7uicj8g08JjfTzv9lB6doAcmkw9H644TH4ryBwrIeI?purpose=inline)

[youtubetotext.ai](https://youtubetotext.ai/blog/subtitles-from-youtube/?utm_source=chatgpt.com)

```
START
  |
  v
Open Website
  |
  v
Enter YouTube URL
  |
  v
Click Get Transcript
  |
  v
Validate Input
  |
  +---- Invalid ----> Show Validation Error
  |
  v
Send API Request
  |
  v
Retrieve Transcript
  |
  +---- Failed -----> Show Error
  |
  v
Display Transcript
  |
  v
Click Copy
  |
  v
Show Copy Success
  |
  v
END
```

### 9.2 Alternative Flow

Video tidak memiliki caption

- Backend mengembalikan error `TRANSCRIPT_NOT_AVAILABLE`.
- Frontend menampilkan pesan.
- Pengguna dapat memasukkan URL lain.

Provider gagal

- Backend mengembalikan error yang sesuai.
- Pengguna mendapatkan opsi mencoba kembali.

URL tidak valid

- Request tidak dikirim.
- Pesan validasi muncul pada input.

## 10. UI/UX Requirements

### 10.1 Design Direction

Gunakan desain:

- Modern.
- Minimalist.
- Clean.
- Productivity-focused.
- Readability-first.

### 10.2 Layout

[TranscriptX — built with AI on Blink](https://images.openai.com/static-rsc-4/pr16YZscr2IVQTE9pmQQBr6mTg5TZCtCLxuv1TzEAwDo3eeQt23yBV_8_TkRO4hGrHPNCm7Ol5qJSvGGofIbnQTEDbjAHL1uMWMx1dvgRqjwSqB2Z-fxDXOA4TQ0wRzV8L2Q69qD58bhUBFByPhf5ajEyaltTbCMu2pHsYUdCyM?purpose=inline)

[How to Download YouTube Subtitles Easily](https://images.openai.com/static-rsc-4/TNmh7qpFYMIJeGSKQUwPCQo5WpUdDz85Csr9vpzYEAj49mKVVz2kxqFTOkXm35uF7LSFkrSKwk2RdS0gPvGRNZgOS8Vfo_zDWy3FIE934HqrFuGaevJKxarak4OXIVQpXfXOKcQKy_rp4yXWlAOPcLXCJRKqk0kMr0liq84bY3Y?purpose=inline)

7

Struktur halaman:

1. Navbar.
2. Hero section.
3. Input URL.
4. Get Transcript button.
5. Transcript result.
6. Footer.

### 10.3 Design Tokens

| Token          | Value         |
| -------------- | ------------- |
| Primary        | Blue / Indigo |
| Background     | Neutral 50    |
| Surface        | White         |
| Text Primary   | Neutral 900   |
| Text Secondary | Neutral 500   |
| Border         | Neutral 200   |
| Success        | Green         |
| Error          | Red           |
| Radius         | 8–12px        |

Nilai dapat disesuaikan pada Phase 3.

### 10.4 Typography

- Font: Inter atau Geist.
- Heading: semibold/bold.
- Body: regular.
- Transcript: nyaman dibaca dengan line-height 1.6–1.8.

### 10.5 Loading State

Tampilkan:

- Loading indicator.
- Text: `Getting transcript...`
- Disable submit button.

### 10.6 Empty State

Sebelum URL dimasukkan:

- Tampilkan instruksi singkat.
- Berikan contoh URL.
- Tidak menampilkan panel transcript kosong yang membingungkan.

### 10.7 Error State

Tampilkan:

- Pesan error.
- Penyebab yang dapat dipahami.
- Tombol Try Again jika relevan.

### 10.8 Success State

Tampilkan:

- Transcript.
- Bahasa.
- Informasi caption jika tersedia.
- Copy button.
- Timestamp jika fitur sudah diimplementasikan.

### 10.9 Responsive Behavior

Desktop

- Content memiliki max-width.
- Input dan tombol dapat berada dalam satu baris.
- Transcript memiliki area yang cukup luas.

Tablet

- Layout mengikuti ukuran layar.
- Komponen tidak saling bertumpuk.

Mobile

- Input dan tombol full width.
- Transcript mudah dibaca.
- Tombol toolbar dapat diakses dengan mudah.

## 11. Error Handling

Gunakan format error response yang konsisten.

```
export interface ApiError {
  success: false;
  error: {
    code: string;
    message: string;
  };
}
```

### 11.1 Error Mapping

| Error Code               | Cause                     | User Message                               |
| ------------------------ | ------------------------- | ------------------------------------------ |
| INVALID_URL              | URL tidak valid           | Please enter a valid YouTube URL           |
| INVALID_VIDEO_ID         | Video ID tidak valid      | Invalid YouTube video                      |
| TRANSCRIPT_NOT_AVAILABLE | Caption tidak tersedia    | Transcript is not available for this video |
| VIDEO_UNAVAILABLE        | Video tidak dapat diakses | This video is unavailable                  |
| LANGUAGE_NOT_AVAILABLE   | Bahasa tidak tersedia     | Selected language is unavailable           |
| PROVIDER_ERROR           | Provider gagal            | Unable to retrieve transcript              |
| REQUEST_TIMEOUT          | Request terlalu lama      | Request timed out. Please try again        |
| RATE_LIMITED             | Terlalu banyak request    | Too many requests. Please try again later  |
| INTERNAL_ERROR           | Kesalahan internal        | Something went wrong                       |

### 11.2 Error Handling Principles

- Jangan menampilkan stack trace.
- Jangan mengungkap detail internal provider.
- Catat error teknis melalui server logging.
- Berikan error code yang konsisten.
- Jangan menganggap semua error sebagai video tanpa transcript.

## 12. Technical Architecture

### 12.1 Architecture Decision

Selected Architecture

Next.js Modular Monolith + Transcript Provider Adapter

Alasan:

- Aplikasi relatif sederhana.
- Tidak membutuhkan komputasi berat.
- Tidak membutuhkan database untuk MVP.
- Pengambilan transcript dapat dilakukan melalui external provider/library.
- Frontend dan backend dapat dikelola dalam satu repository.

### 12.2 Architecture Overview

[Hexagonal Architecture in Next.js: Why Modern Applications Need It - Hexagonal Architecture in Next.js: Why Modern Applications Need It](https://images.openai.com/static-rsc-4/L4yzvVcxClCiZHj2YDARf7Y7oeoc35NqyRGH-uWk0j4MBDS8L5BjRmTdjk7P5w1NFPlSf9Ie8OHH5CtIyGjHFn6UccMFczc0cIiRQa0pgApPZF6F2qVlPKAqMelZHcDFI-VPaggaledaY8v54Ng0kZNrEpxzd7BL4kkTQ2P_eKY?purpose=inline)

[rottengrapes.tech](https://rottengrapes.tech/insights/hexagonal-architecture-in-nextjs-why-modern-applic/?utm_source=chatgpt.com)

```
                  USER
                    |
                    v
            NEXT.JS APPLICATION
                    |
          +---------+---------+
          |                   |
          v                   v
      FRONTEND             BACKEND
          |                   |
          v                   v
      React UI          Route Handler
          |                   |
          v                   v
      React State        Validation
                              |
                              v
                         Application
                           Service
                              |
                              v
                      Transcript Service
                              |
                              v
                      Provider Adapter
                              |
                              v
                     YouTube Transcript
                        Provider
```

### 12.3 Layer Responsibilities

#### Presentation Layer

Lokasi:

```
src/app/
src/components/
```

Tanggung jawab:

- Rendering UI.
- User interaction.
- Form handling.
- Loading dan error state.
- Menampilkan transcript.

#### Application Layer

Lokasi:

```
src/features/transcript/application/
```

Tanggung jawab:

- Mengatur alur pengambilan transcript.
- Memanggil transcript service.
- Mengelola hasil operasi.

#### Business Logic Layer

Lokasi:

```
src/features/transcript/domain/
```

Tanggung jawab:

- Transcript entity.
- Transcript types.
- Business rules.
- Error definitions.

#### Infrastructure Layer

Lokasi:

```
src/features/transcript/infrastructure/
```

Tanggung jawab:

- Implementasi provider.
- Integrasi library.
- External request.
- Mapping response provider.

#### API Layer

Lokasi:

```
src/app/api/transcript/route.ts
```

Tanggung jawab:

- Menerima request.
- Validasi input.
- Memanggil application service.
- Mengembalikan response JSON.

### 12.4 Backend Decision

Gunakan Next.js Route Handlers.

Endpoint:

```
POST /api/transcript
```

Request:

```
{
  "videoId": "VIDEO_ID",
  "language": "en"
}
```

Field `language` bersifat opsional. Jika tidak diberikan, provider dapat menggunakan bahasa default atau pilihan bahasa yang tersedia.

Success Response:

```
{
  "success": true,
  "data": {
    "videoId": "VIDEO_ID",
    "language": "en",
    "isGenerated": false,
    "segments": [
      {
        "text": "Hello everyone",
        "offset": 0,
        "duration": 2.5
      },
      {
        "text": "Welcome to this video",
        "offset": 2.5,
        "duration": 3
      }
    ],
    "transcript": "Hello everyone Welcome to this video"
  }
}
```

Error Response:

```
{
  "success": false,
  "error": {
    "code": "TRANSCRIPT_NOT_AVAILABLE",
    "message": "Transcript is not available"
  }
}
```

Catatan:

- `isGenerated` berarti automatic captions jika provider mampu mengidentifikasinya.
- Jika informasi tersebut tidak tersedia, gunakan `null`.
- Jangan mengarang status caption.
- `segments` mempertahankan timestamp dan urutan teks.
- `transcript` merupakan gabungan teks seluruh segmen.

### 12.5 Transcript Provider Strategy

Bagian ini merupakan keputusan teknis paling penting.

Pilihan implementasi:

| Approach                   | Advantage                            | Risk                                                      |
| -------------------------- | ------------------------------------ | --------------------------------------------------------- |
| YouTube Data API           | Official API                         | Caption download membutuhkan authorization dan permission |
| Community Library          | Integrasi relatif sederhana          | Rentan perubahan mekanisme YouTube                        |
| Third-party Transcript API | Integrasi mudah                      | Biaya, limit, dan ketergantungan provider                 |
| Dedicated Python Service   | Runtime dan library dapat dipisahkan | Menambah infrastruktur                                    |

Rekomendasi MVP:

Lakukan validasi community library atau third-party provider pada Phase 2.

Kriteria pemilihan:

1. Mendukung video publik.
2. Mendukung caption manual.
3. Mendukung automatic caption jika tersedia.
4. Mendukung pengambilan berdasarkan video ID.
5. Dapat mengambil daftar bahasa.
6. Memiliki mekanisme timeout.
7. Tidak mengharuskan download audio.
8. Dokumentasi dan maintenance masih layak.
9. Memiliki lisensi yang sesuai.
10. Penggunaannya sesuai dengan ketentuan layanan yang berlaku.

Jika library berjalan baik di Node.js, integrasikan langsung melalui adapter.

Jika membutuhkan Python atau runtime khusus, pisahkan menjadi dedicated service dan hubungkan melalui HTTP API.

Jangan memaksakan library yang belum terbukti berfungsi.

### 12.6 Frontend Data Flow

```
YouTubeInput
    |
    v
Validate URL
    |
    v
Extract Video ID
    |
    v
Submit Form
    |
    v
Transcript API Client
    |
    v
POST /api/transcript
    |
    v
Parse API Response
    |
    v
Update React State
    |
    v
TranscriptResult
```

### 12.7 State Management

Untuk MVP gunakan React state:

```
const [url, setUrl] = useState("");
const [loading, setLoading] = useState(false);
const [transcript, setTranscript] = useState(null);
const [error, setError] = useState(null);
```

Zustand belum diperlukan karena:

- State hanya digunakan pada halaman utama.
- Tidak ada global user session.
- Tidak ada data lintas halaman yang kompleks.

### 12.8 HTTP Client

Gunakan native `fetch`.

Alasan:

- Sudah tersedia pada Next.js.
- Tidak membutuhkan dependency tambahan.
- Cukup untuk satu endpoint utama.

Axios dapat ditambahkan jika kebutuhan API berkembang.

## 13. Data Model

### 13.1 Database Decision

No Database for MVP

Database belum diperlukan.

Alasan:

- Tidak ada login.
- Tidak ada riwayat transcript.
- Tidak ada penyimpanan permanen.
- Hasil hanya ditampilkan selama sesi pengguna.

### 13.2 Core Types

File:

```
src/features/transcript/domain/transcript.types.ts
```

```
export interface TranscriptSegment {
  text: string;
  offset: number;
  duration: number;
}

export interface Transcript {
  videoId: string;
  language: string | null;
  isGenerated: boolean | null;
  segments: TranscriptSegment[];
  transcript: string;
}

export interface TranscriptRequest {
  videoId: string;
  language?: string;
}

export interface TranscriptResponse {
  success: true;
  data: Transcript;
}
```

Error type:

```
export interface TranscriptError {
  success: false;
  error: {
    code: string;
    message: string;
  };
}
```

### 13.3 Entity Relationship

Tidak ada relational database pada MVP.

Data flow:

```
YouTube URL
    |
    v
Video ID
    |
    v
Transcript Request
    |
    v
Transcript Provider
    |
    v
Transcript Object
    |
    v
Frontend Display
```

### 13.4 Storage Strategy

| Data               | Storage        |
| ------------------ | -------------- |
| Input URL          | React state    |
| Video ID           | Request memory |
| Transcript         | React state    |
| Loading            | React state    |
| Error              | React state    |
| User history       | Not stored     |
| Transcript history | Not stored     |

## 14. Security & Privacy

### 14.1 Input Validation

- Validasi URL di frontend.
- Validasi Video ID di backend.
- Hanya menerima ID, bukan URL arbitrer, pada endpoint internal.
- Validasi bahasa jika diberikan.
- Batasi panjang input.

### 14.2 SSRF Protection

Backend tidak boleh menerima URL arbitrary lalu mengambil URL tersebut.

Gunakan Video ID yang sudah divalidasi.

Jika provider memerlukan URL, URL harus dibentuk menggunakan domain YouTube yang tetap dan telah ditentukan.

### 14.3 Rate Limiting

Terapkan rate limiting pada endpoint:

```
POST /api/transcript
```

Untuk deployment single instance, rate limiting sederhana dapat digunakan sementara.

Untuk deployment multi-instance atau serverless, gunakan shared rate limiter seperti Redis-based service.

Jangan mengandalkan in-memory rate limiter sebagai proteksi production multi-instance.

### 14.4 Request Limits

- Terapkan timeout.
- Batasi ukuran request.
- Batasi ukuran response.
- Hindari concurrent request berlebihan.
- Jangan membuat retry tanpa batas.

### 14.5 Environment Variables

Contoh:

```
NODE_ENV=development

TRANSCRIPT_PROVIDER=default

TRANSCRIPT_PROVIDER_API_KEY=

RATE_LIMIT_ENABLED=true
```

Tambahkan hanya environment variable yang benar-benar digunakan.

Jangan menyimpan secret pada source code.

### 14.6 Privacy Policy

Informasikan kepada pengguna bahwa:

- Aplikasi menerima URL YouTube.
- Transcript diambil dari caption yang tersedia.
- Aplikasi tidak menyimpan transcript secara permanen.
- Data dapat diproses melalui provider pihak ketiga apabila digunakan.
- Ketersediaan transcript bergantung pada akses caption.

## 15. Success Criteria

MVP dinyatakan selesai apabila:

### Product

- Pengguna dapat membuka website.
- Pengguna dapat memasukkan URL YouTube.
- Sistem dapat memvalidasi URL.
- Sistem dapat mengambil transcript ketika tersedia.
- Transcript dapat ditampilkan.
- Transcript dapat disalin.

### Technical

- TypeScript tidak memiliki error.
- ESLint berhasil.
- Production build berhasil.
- API response konsisten.
- External provider terintegrasi melalui adapter.
- Tidak terdapat secret yang terekspos.

### UX

- Loading state berfungsi.
- Error state berfungsi.
- Success state berfungsi.
- Responsive di desktop dan mobile.
- Transcript panjang dapat dibaca.

### Production

- Website dapat diakses melalui HTTPS.
- Environment variable production sudah dikonfigurasi.
- Rate limiting aktif.
- Error logging tersedia.
- Production smoke test berhasil.

## 16. Risks & Technical Constraints

| Risk                               | Impact                   | Mitigation                            |
| ---------------------------------- | ------------------------ | ------------------------------------- |
| YouTube mengubah mekanisme caption | Transcript gagal diambil | Gunakan provider adapter              |
| Video tidak memiliki caption       | Tidak ada transcript     | Tampilkan error informatif            |
| Library tidak kompatibel           | Backend gagal            | Validasi pada Phase 2                 |
| Request terlalu lama               | UX buruk                 | Timeout                               |
| Rate limit provider                | Request gagal            | Rate limiting dan backoff terbatas    |
| Traffic tinggi                     | Server overload          | Batasi concurrency                    |
| Transcript terlalu panjang         | Rendering berat          | Optimalkan rendering                  |
| Provider berbayar                  | Biaya meningkat          | Evaluasi pricing dan usage limit      |
| Ketentuan layanan berubah          | Risiko operasional       | Review kebijakan provider dan YouTube |

### 16.1 Important Technical Limitation

Aplikasi ini bukan YouTube downloader.

Aplikasi tidak menjanjikan transcript untuk semua video.

Transcript hanya dapat diambil apabila caption tersedia dan dapat diakses melalui mekanisme yang digunakan.

Jika video tidak memiliki caption, MVP akan menampilkan pesan bahwa transcript tidak tersedia.

# PART B — SEQUENTIAL DEVELOPMENT PLAYBOOK

## Development Workflow

Development harus mengikuti urutan berikut:

## 10 Development Phases

1. Phase 1 — Environment & Project Bootstrap

   Setup tools, Next.js, dependencies, dan Git.

2. Phase 2 — Technical Planning & Architecture

   Validasi transcript provider dan menghasilkan technical plan.

3. Phase 3 — UI/UX Planning & Design System

   Design direction, wireframe, dan design system.

4. Phase 4 — Frontend Foundation & Application Shell

   Layout, navbar, footer, routing, dan reusable components.

5. Phase 5 — Core Frontend Feature Development

   UI input URL, transcript result, loading, error, dan mock integration.

6. Phase 6 — Backend & Core Business Logic

   Transcript API, provider adapter, validasi, dan business logic.

7. Phase 7 — Frontend & Backend Integration

   Menghubungkan frontend dengan backend sebenarnya.

8. Phase 8 — Additional Features & Product Completion

   Language selection, timestamp, download TXT, SEO, dan UX improvements.

9. Phase 9 — Testing, Security & Optimization

   Testing, security audit, performance, dan bug fixing.

10. Phase 10 — Production Deployment & Release

    Deployment, HTTPS, monitoring, dan release.

### Aturan eksekusi

1. Jalankan hanya satu phase dalam satu waktu.
2. Berikan prompt phase yang bersangkutan kepada AI Coding Assistant.
3. Review semua perubahan.
4. Jalankan testing.
5. Pastikan acceptance criteria terpenuhi.
6. Buat Git checkpoint.
7. Baru lanjut ke phase berikutnya.

Jangan meminta AI menyelesaikan seluruh 10 phase sekaligus.

# PHASE 1 — Environment & Project Bootstrap

Foundation

## 1. Phase Objective

Menyiapkan development environment dan membuat project Next.js yang siap digunakan untuk pengembangan transscipt-yt.

Target:

- Node.js tersedia.
- Package manager tersedia.
- Next.js berhasil dibuat.
- TypeScript aktif.
- Tailwind CSS aktif.
- shadcn/ui terpasang.
- Git repository tersedia.
- Project dapat dijalankan secara lokal.

## 2. Scope of Work

| Task              | Description                    |
| ----------------- | ------------------------------ |
| Environment Setup | Persiapan Node.js dan Git      |
| Next.js Setup     | Membuat Next.js project        |
| TypeScript        | Konfigurasi TypeScript         |
| Tailwind CSS      | Styling framework              |
| shadcn/ui         | UI component system            |
| Dependencies      | Install dependencies dasar     |
| Git               | Version control                |
| Verification      | Menjalankan development server |

## 3. Detailed Implementation Steps

### Step 1 — Install Development Tools

Pastikan tersedia:

- Node.js LTS.
- npm.
- Git.
- VS Code.

Verifikasi:

```
node --version
npm --version
git --version
```

### Step 2 — Create Next.js Project

Gunakan:

```
npx create-next-app@latest transscipt-yt
```

Pilihan konfigurasi:

| Configuration | Value   |
| ------------- | ------- |
| TypeScript    | Yes     |
| ESLint        | Yes     |
| Tailwind CSS  | Yes     |
| src directory | Yes     |
| App Router    | Yes     |
| Turbopack     | Default |
| Import alias  | @/\*    |

### Step 3 — Enter Project

```
cd transscipt-yt
```

### Step 4 — Install Dependencies

```
npm install
```

### Step 5 — Initialize shadcn/ui

```
npx shadcn@latest init
```

Pilih konfigurasi yang sesuai dengan project.

### Step 6 — Install Initial Components

```
npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add card
npx shadcn@latest add textarea
npx shadcn@latest add alert
npx shadcn@latest add skeleton
```

Tambahkan komponen lain hanya jika dibutuhkan.

### Step 7 — Initialize Git

Jika repository belum memiliki Git:

```
git init
git branch -M main
```

Buat `.gitignore` dan pastikan file berikut tidak di-commit:

```
node_modules
.next
.env
.env.local
```

### Step 8 — Run Project

```
npm run dev
```

Buka:

```
http://localhost:3000
```

### Step 9 — Verify

```
npm run lint
npx tsc --noEmit
npm run build
```

Jika script lint berbeda pada versi Next.js yang digunakan, sesuaikan dengan script yang dihasilkan project.

## 4. Required Commands

Ringkasan:

```
npx create-next-app@latest transscipt-yt

cd transscipt-yt

npm install

npx shadcn@latest init

npx shadcn@latest add button input card textarea alert skeleton

git init

git branch -M main

npm run dev
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 1

# PHASE 1 — ENVIRONMENT & PROJECT BOOTSTRAP

Kamu bertindak sebagai Senior Fullstack Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Task: Implementasikan PHASE 1 saja.

## Konteks Project

transscipt-yt adalah website untuk mengambil transcript dari video YouTube berdasarkan URL dan menampilkan hasilnya agar dapat disalin.

Tech stack:

- Next.js App Router
- TypeScript
- Tailwind CSS
- shadcn/ui
- npm
- Git

## Objectives

1. Periksa environment dan kondisi repository.
2. Siapkan Next.js project.
3. Pastikan TypeScript aktif.
4. Pastikan Tailwind CSS berfungsi.
5. Setup shadcn/ui.
6. Install komponen dasar.
7. Setup Git.
8. Verifikasi project dapat dijalankan.

## Implementation Rules

- Baca PROJECT_PRD_AND_PLAYBOOK.md.
- Kerjakan Phase 1 saja.
- Jangan membuat fitur transcript.
- Jangan membuat backend API.
- Jangan menambahkan database.
- Jangan memasang Axios atau Zustand.
- Jangan menambahkan dependency yang belum dibutuhkan.
- Jangan menimpa file existing tanpa memeriksa isinya.
- Jika project sudah tersedia, lakukan audit sebelum menginisialisasi ulang.
- Gunakan versi dependency yang kompatibel.
- Jangan mengasumsikan command berhasil tanpa menjalankannya.

## Expected Deliverables

- Next.js project.
- TypeScript configuration.
- Tailwind CSS configuration.
- shadcn/ui configuration.
- Git initialization.
- Basic application entry point.

## Verification

Jalankan:

- npm run dev
- npm run lint
- npx tsc --noEmit
- npm run build

Laporkan hasil aktual setiap command.

## Acceptance Criteria

- Project berhasil dijalankan.
- Homepage dapat dibuka.
- TypeScript aktif.
- Tailwind aktif.
- shadcn/ui tersedia.
- Git repository tersedia.
- Tidak terdapat error build.

## Final Report

Berikan:

1. Summary.
2. Files created.
3. Files modified.
4. Dependencies installed.
5. Commands executed.
6. Testing results.
7. Known issues.
8. Git checkpoint readiness.

Jangan melanjutkan ke Phase 2.

## 6. Expected Deliverables

```
transscipt-yt/
├── public/
├── src/
│   └── app/
│       ├── favicon.ico
│       ├── globals.css
│       ├── layout.tsx
│       └── page.tsx
├── components.json
├── package.json
├── package-lock.json
├── tsconfig.json
├── next.config.ts
└── .gitignore
```

## 7. Acceptance Criteria

- Next.js berjalan.
- TypeScript aktif.
- Tailwind CSS berfungsi.
- shadcn/ui terkonfigurasi.
- Git tersedia.
- Homepage dapat diakses.
- Lint berhasil.
- Type check berhasil.
- Production build berhasil.

## 8. Testing Requirements

```
npm run dev
npm run lint
npx tsc --noEmit
npm run build
```

## 9. Git Checkpoint

```
git add .
git commit -m "phase 1: project bootstrap"
```

# PHASE 2 — Technical Planning & Architecture

Technical Design

## 1. Phase Objective

Menyusun arsitektur teknis final sebelum mengimplementasikan fitur.

Fokus utama phase ini adalah memvalidasi bagaimana transcript dari YouTube akan diperoleh.

Output utama:

`TECHNICAL_PLAN.md`

## 2. Scope of Work

- Analisis PRD.
- Validasi library/provider transcript.
- Menentukan arsitektur backend.
- Menentukan struktur folder.
- Menentukan API contract.
- Menentukan data flow.
- Menentukan error handling.
- Menentukan dependency.
- Mengidentifikasi technical risks.

## 3. Detailed Implementation Steps

### Step 1 — Analyze Existing Project

Periksa:

```
ls -la
cat package.json
```

Windows PowerShell:

```
Get-ChildItem
Get-Content package.json
```

### Step 2 — Research Transcript Provider

Evaluasi beberapa pilihan:

1. YouTube Data API.
2. Community transcript library untuk Node.js.
3. Third-party transcript API.
4. Dedicated Python service apabila diperlukan.

Jangan langsung mengasumsikan library tertentu kompatibel.

Buat tabel evaluasi:

| Criteria              | Requirement        |
| --------------------- | ------------------ |
| Public video          | Required           |
| Manual captions       | Required           |
| Auto captions         | Preferred          |
| Language selection    | Preferred          |
| Timestamp             | Preferred          |
| Node.js compatibility | Preferred          |
| Maintenance           | Important          |
| License               | Must be acceptable |
| Timeout handling      | Required           |

### Step 3 — Validate Provider

Buat proof of concept terpisah jika diperlukan.

Pengujian harus mencakup:

- Video dengan manual captions.
- Video dengan automatic captions.
- Video tanpa captions.
- Video dengan banyak pilihan bahasa.
- Video yang tidak tersedia.

Jangan menganggap satu video berhasil berarti provider sudah layak untuk production.

Jika provider membutuhkan akses berbayar atau autentikasi, dokumentasikan batasannya.

### Step 4 — Determine Architecture

Target:

```
Next.js
  |
  v
Route Handler
  |
  v
Transcript Application Service
  |
  v
Transcript Provider Adapter
  |
  v
Provider / Library
```

### Step 5 — Determine Folder Structure

Rancangan:

```
src/
├── app/
│   ├── api/
│   │   └── transcript/
│   │       └── route.ts
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
│
├── components/
│   ├── layout/
│   └── ui/
│
├── features/
│   └── transcript/
│       ├── components/
│       ├── hooks/
│       ├── application/
│       ├── domain/
│       ├── infrastructure/
│       ├── services/
│       └── utils/
│
├── lib/
│   ├── utils.ts
│   └── validations/
│
└── types/
```

Tidak semua folder harus memiliki file sejak awal.

### Step 6 — Determine API Contract

```
POST /api/transcript
```

Request:

```
{
  "videoId": "VIDEO_ID",
  "language": "en"
}
```

Tentukan:

- Request validation.
- Response structure.
- Error codes.
- Timeout.
- Provider error mapping.

### Step 7 — Determine Security

- URL validation.
- Video ID validation.
- SSRF prevention.
- Rate limiting.
- Request size limits.
- Secret management.

### Step 8 — Document Technical Decisions

Buat:

```
TECHNICAL_PLAN.md
```

Dokumen harus mencakup:

1. Architecture overview.
2. Technology decisions.
3. Provider evaluation.
4. Final provider decision.
5. Folder structure.
6. Data flow.
7. API contract.
8. Data model.
9. Error handling.
10. Security.
11. Dependency list.
12. Technical risks.
13. Implementation notes.

## 4. Required Commands

```
git status
```

Jika melakukan PoC:

```
npm install <validated-package>
```

Gunakan command instalasi hanya setelah package benar-benar dipilih.

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 2

# PHASE 2 — TECHNICAL PLANNING & ARCHITECTURE

Kamu bertindak sebagai Senior Software Architect dan Senior Fullstack Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Task: Selesaikan PHASE 2 saja.

## Objectives

Buat dokumen TECHNICAL_PLAN.md berdasarkan PRD.

Fokus utama adalah menentukan mekanisme pengambilan transcript YouTube yang benar-benar dapat digunakan.

## Required Analysis

1. Audit kondisi repository.
2. Analisis kebutuhan produk.
3. Evaluasi YouTube Data API.
4. Evaluasi community transcript library.
5. Evaluasi third-party transcript API.
6. Tentukan apakah Next.js Route Handlers cukup.
7. Tentukan apakah dedicated service diperlukan.
8. Tentukan folder structure.
9. Tentukan API contract.
10. Tentukan TypeScript data model.
11. Tentukan provider adapter.
12. Tentukan error handling.
13. Tentukan security strategy.
14. Tentukan dependency.
15. Identifikasi technical risks.

## Critical Rules

- Jangan implementasikan UI utama.
- Jangan membuat seluruh backend.
- Jangan membuat fitur production.
- Jangan menambahkan database.
- Jangan menggunakan Rust atau Tauri.
- Jangan mengasumsikan semua YouTube caption dapat diakses melalui API publik.
- Jangan memilih library hanya berdasarkan nama atau popularitas.
- Validasi dokumentasi, kompatibilitas runtime, lisensi, maintenance, dan kemampuan aktual provider.
- Jika memungkinkan, lakukan proof of concept terbatas.
- Jangan mengklaim provider berhasil tanpa pengujian aktual.
- Jika tidak dapat diverifikasi, tandai sebagai unresolved risk.

## Required Output

Buat TECHNICAL_PLAN.md dengan:

1. Architecture Overview.
2. Technology Decisions.
3. Transcript Provider Evaluation.
4. Final Provider Decision.
5. Folder Structure.
6. Data Flow.
7. API Specification.
8. Data Models.
9. Error Handling.
10. Security Strategy.
11. Dependencies.
12. Deployment Considerations.
13. Technical Risks.
14. Implementation Roadmap.

## Acceptance Criteria

- Semua keputusan teknis memiliki alasan.
- Provider memiliki hasil evaluasi.
- Arsitektur konsisten dengan PRD.
- Tidak ada database yang dipaksakan.
- Tidak ada dependency tanpa kebutuhan.
- API contract jelas.
- Technical risks terdokumentasi.

## Final Report

Laporkan file yang dibuat, keputusan provider, dependency, hasil PoC, risiko yang belum terselesaikan, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 3.

## 6. Expected Deliverables

```
TECHNICAL_PLAN.md
```

Opsional:

```
proof-of-concept/
```

PoC hanya digunakan jika dibutuhkan untuk memvalidasi provider.

## 7. Acceptance Criteria

- Arsitektur telah ditentukan.
- Transcript provider telah dievaluasi.
- Provider dapat dibuktikan atau risikonya terdokumentasi.
- Folder structure tersedia.
- API contract tersedia.
- Error handling ditentukan.
- Security strategy tersedia.
- TECHNICAL_PLAN.md selesai.

## 8. Testing Requirements

Jika membuat PoC:

- Test pengambilan transcript.
- Test video tanpa caption.
- Test bahasa.
- Test timestamp.
- Test timeout.
- Test error provider.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 2: technical architecture"
```

# PHASE 3 — UI/UX Planning & Design System

Design Planning

## 1. Phase Objective

Menyusun panduan visual dan UX sebelum implementasi frontend.

Output:

`UI_UX_GUIDE.md`

## 2. Scope of Work

- Design direction.
- Wireframe.
- Page layout.
- Navigation.
- Design tokens.
- Typography.
- Spacing.
- Component specification.
- Responsive behavior.
- Loading state.
- Empty state.
- Error state.
- Success state.

## 3. Detailed Implementation Steps

### Step 1 — Define Design Direction

Gunakan pendekatan:

- Minimalist.
- Productivity SaaS.
- Clean typography.
- White/neutral background.
- Indigo atau blue sebagai primary color.

### Step 2 — Define Homepage Layout

```
+---------------------------------------+
| Logo                     Navigation   |
+---------------------------------------+
|                                       |
|       YouTube Transcript Generator    |
|       Get transcript from any         |
|       supported YouTube video         |
|                                       |
|  +---------------------------------+  |
|  | Paste YouTube URL               |  |
|  +---------------------------------+  |
|  |          Get Transcript         |  |
|  +---------------------------------+  |
|                                       |
|             Result Area              |
|                                       |
|  +---------------------------------+  |
|  | Transcript                      |  |
|  |                                 |  |
|  | Text content...                 |  |
|  |                                 |  |
|  +---------------------------------+  |
|  | Copy Transcript                 |  |
|  +---------------------------------+  |
|                                       |
+---------------------------------------+
| Footer                                |
+---------------------------------------+
```

### Step 3 — Define Design Tokens

Buat:

- Colors.
- Typography.
- Spacing.
- Border radius.
- Shadows.
- Container width.
- Responsive breakpoints.

### Step 4 — Define Component Specification

| Component      | Specification           |
| -------------- | ----------------------- |
| Navbar         | Simple header           |
| Hero           | Heading and description |
| URL Input      | Input + submit          |
| Result Card    | Transcript container    |
| Copy Button    | Primary action          |
| Loading        | Spinner/skeleton        |
| Error          | Alert component         |
| Language Badge | Language indicator      |

### Step 5 — Define Responsive Design

Tentukan breakpoint:

- Mobile.
- Tablet.
- Desktop.

### Step 6 — Document

Buat `UI_UX_GUIDE.md`.

## 4. Required Commands

Tidak ada dependency tambahan yang diwajibkan.

```
git status
```

Jika menggunakan design references, dokumentasikan referensinya tanpa memasukkan library UI yang belum dibutuhkan.

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 3

# PHASE 3 — UI/UX PLANNING & DESIGN SYSTEM

Kamu bertindak sebagai Senior UI/UX Designer dan Frontend Architect.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

Task: Selesaikan PHASE 3 saja.

## Objectives

Buat UI_UX_GUIDE.md sebagai pedoman visual seluruh aplikasi.

## Required Work

1. Tentukan design direction.
2. Tentukan homepage layout.
3. Buat wireframe desktop.
4. Buat wireframe mobile.
5. Tentukan navigation.
6. Tentukan color palette.
7. Tentukan typography.
8. Tentukan spacing.
9. Tentukan border radius.
10. Tentukan reusable components.
11. Tentukan loading state.
12. Tentukan empty state.
13. Tentukan error state.
14. Tentukan success state.
15. Tentukan responsive behavior.
16. Tentukan accessibility requirements.

## Design Direction

Gunakan:

- Modern minimalist.
- Productivity-focused.
- Clean SaaS interface.
- Readability-first.
- Responsive design.

## Restrictions

- Jangan mengimplementasikan halaman.
- Jangan membuat business logic.
- Jangan mengimplementasikan API.
- Jangan menambahkan dependency tanpa alasan.
- Ikuti keputusan technical architecture.

## Expected Deliverable

UI_UX_GUIDE.md.

Dokumen harus dapat digunakan langsung sebagai referensi implementasi frontend.

## Acceptance Criteria

- Homepage wireframe tersedia.
- Desktop dan mobile layout ditentukan.
- Design tokens tersedia.
- Component specifications tersedia.
- UI states ditentukan.
- Accessibility diperhatikan.
- Konsisten dengan PRD dan TECHNICAL_PLAN.md.

## Final Report

Laporkan isi dokumen, keputusan design, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 4.

## 6. Expected Deliverables

```
UI_UX_GUIDE.md
```

## 7. Acceptance Criteria

- Wireframe desktop.
- Wireframe mobile.
- Design tokens.
- Typography.
- Spacing.
- Component specification.
- Loading/error/success states.
- Responsive guidelines.

## 8. Testing Requirements

Review konsistensi UI/UX terhadap PRD dan technical architecture.

Belum diperlukan automated test.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 3: ui ux planning"
```

# PHASE 4 — Frontend Foundation & Application Shell

Frontend Foundation

## 1. Phase Objective

Membangun struktur frontend dasar berdasarkan UI/UX Guide.

Fokus pada application shell dan reusable components.

## 2. Scope of Work

- Final folder structure.
- Root layout.
- Global CSS.
- Navbar.
- Footer.
- Homepage shell.
- Basic routing.
- Reusable UI components.
- Responsive foundation.

Tidak mengimplementasikan pengambilan transcript.

## 3. Detailed Implementation Steps

### Step 1 — Implement Folder Structure

Gunakan struktur dari `TECHNICAL_PLAN.md`.

### Step 2 — Implement Global Layout

File:

```
src/app/layout.tsx
```

Tanggung jawab:

- HTML root.
- Font.
- Metadata.
- Global layout.

### Step 3 — Implement Global CSS

File:

```
src/app/globals.css
```

Implementasikan:

- Color tokens.
- Typography.
- Background.
- Global spacing.
- Responsive defaults.

### Step 4 — Implement Navbar

File:

```
src/components/layout/navbar.tsx
```

### Step 5 — Implement Footer

File:

```
src/components/layout/footer.tsx
```

### Step 6 — Implement Homepage Shell

File:

```
src/app/page.tsx
```

Buat:

- Hero section.
- Main content container.
- Placeholder input section.
- Placeholder result section.

Belum ada business logic.

### Step 7 — Implement Reusable Components

Contoh:

```
src/features/transcript/components/
├── transcript-form.tsx
├── transcript-result.tsx
├── transcript-toolbar.tsx
├── transcript-empty.tsx
└── transcript-error.tsx
```

Komponen dapat berupa presentational component atau placeholder.

### Step 8 — Verify Responsive Design

Periksa:

- Mobile.
- Tablet.
- Desktop.

## 4. Required Commands

```
npm run dev
npm run lint
npx tsc --noEmit
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 4

# PHASE 4 — FRONTEND FOUNDATION & APPLICATION SHELL

Kamu bertindak sebagai Senior Frontend Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

UI reference: UI_UX_GUIDE.md

Task: Implementasikan PHASE 4 saja.

## Objectives

Bangun frontend foundation dan application shell.

## Implementation Tasks

1. Audit existing project.
2. Implementasikan final folder structure.
3. Implementasikan root layout.
4. Implementasikan global CSS.
5. Implementasikan design tokens.
6. Implementasikan Navbar.
7. Implementasikan Footer.
8. Implementasikan homepage shell.
9. Implementasikan reusable UI components.
10. Implementasikan basic routing.
11. Implementasikan responsive layout.
12. Implementasikan metadata dasar.

## Component Requirements

- Navbar.
- Footer.
- HeroSection.
- TranscriptForm placeholder.
- TranscriptResult placeholder.
- TranscriptToolbar placeholder.
- Loading placeholder.
- Error placeholder.

## Restrictions

- Jangan implementasikan YouTube transcript retrieval.
- Jangan membuat API backend.
- Jangan menghubungkan provider.
- Jangan membuat mock API.
- Jangan menambahkan database.
- Jangan mengimplementasikan fitur utama.
- Ikuti UI_UX_GUIDE.md.
- Jangan menggunakan Zustand tanpa kebutuhan.

## Expected Deliverables

Frontend application shell yang konsisten dengan design guide.

## Verification

Jalankan:

- npm run dev
- npm run lint
- npx tsc --noEmit
- npm run build

## Acceptance Criteria

- Homepage dapat dibuka.
- Navbar dan Footer berfungsi.
- Layout responsive.
- Komponen reusable tersedia.
- Tidak ada business logic transcript.
- Build berhasil.

## Final Report

Laporkan file dibuat/diubah, komponen yang tersedia, hasil testing, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 5.

## 6. Expected Deliverables

```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
│
├── components/
│   └── layout/
│       ├── navbar.tsx
│       └── footer.tsx
│
└── features/
    └── transcript/
        └── components/
```

## 7. Acceptance Criteria

- Folder structure sesuai technical plan.
- Global layout tersedia.
- Navbar tersedia.
- Footer tersedia.
- Homepage shell tersedia.
- Responsive design berfungsi.
- Reusable components tersedia.
- Tidak ada business logic transcript.

## 8. Testing Requirements

```
npm run lint
npx tsc --noEmit
npm run build
```

Visual testing desktop dan mobile.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 4: frontend foundation"
```

# PHASE 5 — Core Frontend Feature Development

Core UI

## 1. Phase Objective

Membangun seluruh interface utama untuk fitur transcript.

Frontend harus siap dihubungkan dengan backend pada Phase 7.

## 2. Scope of Work

- YouTube URL input.
- Client-side URL validation.
- Submit form.
- Transcript result.
- Copy transcript.
- Loading state.
- Error state.
- Success state.
- Empty state.
- React state management.

Gunakan mock data sementara.

## 3. Detailed Implementation Steps

### Step 1 — YouTube URL Utility

Buat:

```
src/features/transcript/utils/youtube-url.ts
```

Fungsi:

```
export function extractYouTubeVideoId(
  url: string
): string | null {
  // Validate and extract supported YouTube video ID
}
```

Dukung format:

- youtube.com/watch.
- youtu.be.
- youtube.com/shorts.
- youtube.com/embed.
- m.youtube.com/watch.

Gunakan URL parser, bukan sekadar substring.

### Step 2 — Implement Transcript Form

Komponen:

```
transcript-form.tsx
```

Fitur:

- Input URL.
- Validation.
- Submit.
- Loading.
- Error.

### Step 3 — Implement Result Component

Komponen:

```
transcript-result.tsx
```

Menampilkan:

- Transcript.
- Language.
- Copy button.
- Result container.

### Step 4 — Implement Mock Service

Gunakan mock data yang terpisah dari production service.

Contoh:

```
src/features/transcript/
└── mocks/
    └── transcript.mock.ts
```

Mock hanya digunakan pada Phase 5.

Jangan menanam mock data langsung di production API.

### Step 5 — Implement Copy Function

Gunakan:

```
await navigator.clipboard.writeText(transcript);
```

Tangani kemungkinan Clipboard API tidak tersedia atau gagal.

### Step 6 — Implement State

Gunakan React state.

State:

```
type TranscriptStatus =
  | "idle"
  | "loading"
  | "success"
  | "error";
```

### Step 7 — Implement UX States

- Idle.
- Loading.
- Success.
- Error.

### Step 8 — Implement Clear Action

Pengguna dapat membersihkan input dan hasil.

## 4. Required Commands

Jika belum ada:

```
npm install zod
```

Zod hanya digunakan jika memang dibutuhkan untuk validasi schema. Untuk validasi URL sederhana, native TypeScript cukup.

Testing:

```
npm run lint
npx tsc --noEmit
npm run build
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 5

# PHASE 5 — CORE FRONTEND FEATURE DEVELOPMENT

Kamu bertindak sebagai Senior Frontend Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

UI reference: UI_UX_GUIDE.md

Task: Implementasikan PHASE 5 saja.

## Objectives

Implementasikan seluruh core frontend features untuk YouTube Transcript Generator.

## Implementation Tasks

1. Buat YouTube URL validation utility.
2. Buat Video ID extraction utility.
3. Implementasikan TranscriptForm.
4. Implementasikan URL input.
5. Implementasikan submit interaction.
6. Implementasikan TranscriptResult.
7. Implementasikan Copy Transcript.
8. Implementasikan loading state.
9. Implementasikan error state.
10. Implementasikan empty state.
11. Implementasikan success state.
12. Implementasikan clear action.
13. Implementasikan React state management.
14. Buat mock transcript service untuk pengembangan frontend.

## Mock Data Rules

Mock data hanya untuk frontend development.

Pisahkan mock service dari production service.

Berikan penamaan dan dokumentasi yang jelas bahwa mock tidak digunakan pada production.

Jangan menganggap mock sebagai hasil pengambilan transcript sebenarnya.

## Technical Rules

- Gunakan TypeScript.
- Gunakan React state.
- Gunakan native fetch pada tahap integrasi nanti.
- Jangan memasang Zustand tanpa kebutuhan.
- Jangan membuat backend.
- Jangan membuat API route.
- Jangan mengintegrasikan provider.
- Jangan menambahkan fitur P1 atau P2.
- Gunakan reusable components.
- Pertahankan struktur project.

## Acceptance Criteria

- URL dapat divalidasi.
- Video ID dapat diekstrak.
- Form berfungsi.
- Mock result dapat ditampilkan.
- Copy berfungsi.
- Loading berfungsi.
- Error berfungsi.
- Clear action berfungsi.
- Responsive.
- Tidak ada TypeScript error.

## Testing

Buat unit test untuk URL validation dan Video ID extraction jika test framework sudah tersedia.

Jalankan lint, type check, dan build.

## Final Report

Laporkan komponen, utility, mock service, dependency, hasil testing, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 6.

## 6. Expected Deliverables

```
src/features/transcript/
├── components/
│   ├── transcript-form.tsx
│   ├── transcript-result.tsx
│   ├── transcript-toolbar.tsx
│   ├── transcript-error.tsx
│   └── transcript-loading.tsx
│
├── utils/
│   └── youtube-url.ts
│
├── mocks/
│   └── transcript.mock.ts
│
└── types/
    └── transcript.types.ts
```

## 7. Acceptance Criteria

- Input URL berfungsi.
- URL validation berfungsi.
- Video ID extraction berfungsi.
- Mock transcript ditampilkan.
- Copy transcript berfungsi.
- Loading state tersedia.
- Error state tersedia.
- Clear action berfungsi.
- Responsive.
- Mock terpisah dari production code.

## 8. Testing Requirements

Test minimal:

| Test              | Expected          |
| ----------------- | ----------------- |
| Valid YouTube URL | Pass              |
| Invalid domain    | Reject            |
| Empty URL         | Reject            |
| youtu.be          | Pass              |
| Shorts URL        | Pass              |
| Embed URL         | Pass              |
| Copy transcript   | Clipboard updated |
| Clear input       | Result cleared    |

## 9. Git Checkpoint

```
git add .
git commit -m "phase 5: core frontend features"
```

# PHASE 6 — Backend & Core Business Logic

Backend Implementation

## 1. Phase Objective

Mengimplementasikan backend sebenarnya untuk pengambilan transcript.

Phase ini merupakan inti fungsional aplikasi.

Dependency: Phase 2 harus sudah menghasilkan keputusan provider yang tervalidasi.

## 2. Scope of Work

- Transcript domain types.
- Provider adapter.
- Transcript service.
- API Route Handler.
- Input validation.
- Error mapping.
- Timeout.
- Logging.
- Rate limiting.
- Security.

## 3. Detailed Implementation Steps

### Step 1 — Implement Domain Types

Buat:

```
src/features/transcript/domain/
├── transcript.types.ts
└── transcript.errors.ts
```

### Step 2 — Implement Provider Interface

Contoh:

```
export interface TranscriptProvider {
  getTranscript(params: {
    videoId: string;
    language?: string;
  }): Promise<Transcript>;
}
```

Tambahkan interface untuk mengambil daftar bahasa apabila provider mendukungnya dan fitur tersebut diperlukan.

### Step 3 — Implement Provider Adapter

Buat:

```
src/features/transcript/infrastructure/
└── providers/
    └── youtube-transcript.provider.ts
```

Adapter bertanggung jawab:

- Memanggil library/provider.
- Mengubah response menjadi format internal.
- Menangani error provider.
- Mempertahankan urutan segmen.
- Mempertahankan timestamp jika tersedia.

Jangan biarkan format response library tersebar ke seluruh aplikasi.

### Step 4 — Implement Application Service

Buat:

```
src/features/transcript/application/
└── get-transcript.service.ts
```

Tanggung jawab:

- Menerima request.
- Memvalidasi Video ID.
- Memanggil provider.
- Menormalisasi hasil.
- Mengembalikan Transcript.

### Step 5 — Implement API Route

File:

```
src/app/api/transcript/route.ts
```

Method:

```
POST
```

Flow:

```
Request
  |
  v
Validate Body
  |
  v
Validate Video ID
  |
  v
Call Transcript Service
  |
  v
Normalize Result
  |
  v
Return JSON
```

### Step 6 — Implement Error Mapping

Map error provider menjadi error internal.

Contoh:

```
Provider Error
    |
    v
Application Error
    |
    v
API Error Response
```

### Step 7 — Implement Timeout

Request harus memiliki timeout.

Gunakan `AbortSignal.timeout()` jika kompatibel dengan runtime dan dependency yang dipilih, atau mekanisme timeout lain yang telah diverifikasi.

### Step 8 — Implement Logging

Log minimal:

- Request received.
- Video ID (hindari URL lengkap).
- Provider response status.
- Processing duration.
- Error code.

Jangan log transcript secara keseluruhan atau informasi sensitif.

### Step 9 — Implement Rate Limiting

Gunakan rate limiter yang sesuai dengan target deployment.

Untuk production multi-instance, gunakan shared storage.

### Step 10 — Remove Production Mock Dependency

Backend harus menggunakan provider nyata.

Jangan mengembalikan mock transcript apabila provider gagal.

## 4. Required Commands

Contoh:

```
npm install <validated-transcript-provider>
```

Perintah di atas harus diganti dengan package yang benar-benar dipilih pada Phase 2.

Jika menggunakan provider API:

```
npm install
```

Tidak perlu memasang package baru jika native `fetch` sudah mencukupi.

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 6

# PHASE 6 — BACKEND & CORE BUSINESS LOGIC

Kamu bertindak sebagai Senior Backend Developer dan Software Architect.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

Task: Implementasikan PHASE 6 saja.

## Prerequisites

Pastikan TECHNICAL_PLAN.md sudah tersedia dan provider transcript telah dievaluasi.

Jika provider belum memiliki keputusan yang dapat dipertanggungjawabkan, jangan mengarang integrasi. Laporkan blocker dan minta persetujuan.

## Objectives

Implementasikan backend untuk mengambil transcript YouTube.

## Implementation Tasks

1. Implementasikan domain types.
2. Implementasikan TranscriptProvider interface.
3. Implementasikan provider adapter.
4. Implementasikan Transcript Service.
5. Implementasikan POST /api/transcript.
6. Implementasikan request validation.
7. Implementasikan Video ID validation.
8. Implementasikan language validation jika diperlukan.
9. Implementasikan response normalization.
10. Implementasikan error mapping.
11. Implementasikan timeout.
12. Implementasikan logging.
13. Implementasikan rate limiting.
14. Implementasikan security protections.

## Architecture Rules

- Ikuti TECHNICAL_PLAN.md.
- Gunakan Next.js Route Handlers.
- Pisahkan provider adapter dari business logic.
- Gunakan TypeScript.
- Gunakan native fetch jika memadai.
- Jangan membuat database.
- Jangan menambahkan service terpisah tanpa kebutuhan.
- Jangan menggunakan mock response untuk menggantikan provider production.
- Jangan mengunduh audio atau video.
- Jangan mengklaim transcript tersedia jika provider gagal.

## Security Requirements

- Validasi semua request.
- Cegah SSRF.
- Batasi request.
- Implementasikan timeout.
- Jangan expose API key.
- Jangan expose stack trace.
- Jangan log seluruh transcript.
- Hindari retry tanpa batas.

## API Contract

POST /api/transcript

Request:\
{\
"videoId": "VIDEO_ID",\
"language": "en"\
}

Response harus mengikuti schema pada PRD dan TECHNICAL_PLAN.md.

## Acceptance Criteria

- API dapat menerima request valid.
- Provider benar-benar dipanggil.
- Transcript tersedia jika caption dapat diakses.
- Error mapping benar.
- Timeout berfungsi.
- Request invalid ditolak.
- Response konsisten.
- Tidak ada mock transcript pada production path.

## Testing Requirements

Test:

- Valid video.
- Invalid video ID.
- No captions.
- Provider error.
- Timeout.
- Invalid language.
- Malformed request.

Jalankan lint, type check, dan build.

## Final Report

Laporkan:

1. Backend architecture.
2. API endpoints.
3. Provider yang digunakan.
4. Dependency.
5. Files created/modified.
6. Test results aktual.
7. Security implementation.
8. Known limitations.
9. Git checkpoint readiness.

Jangan melanjutkan ke Phase 7.

## 6. Expected Deliverables

```
src/
├── app/
│   └── api/
│       └── transcript/
│           └── route.ts
│
└── features/
    └── transcript/
        ├── domain/
        │   ├── transcript.types.ts
        │   └── transcript.errors.ts
        │
        ├── application/
        │   └── get-transcript.service.ts
        │
        └── infrastructure/
            └── providers/
                └── youtube-transcript.provider.ts
```

## 7. Acceptance Criteria

- Provider nyata terintegrasi.
- API tersedia.
- Input divalidasi.
- Error mapping tersedia.
- Timeout berfungsi.
- Logging tersedia.
- Rate limiting diterapkan.
- Tidak menggunakan mock production.
- Security diperhatikan.
- API dapat mengambil transcript.

## 8. Testing Requirements

| Scenario                  | Expected      |
| ------------------------- | ------------- |
| Valid video with captions | Success       |
| Video without captions    | Error         |
| Invalid video ID          | 400           |
| Provider failure          | Mapped error  |
| Timeout                   | Timeout error |
| Invalid request           | 400           |
| Too many requests         | 429           |

## 9. Git Checkpoint

```
git add .
git commit -m "phase 6: backend implementation"
```

# PHASE 7 — Frontend & Backend Integration

End-to-End Integration

## 1. Phase Objective

Menghubungkan frontend dengan backend yang telah dibuat.

Menghapus penggunaan mock sebagai jalur utama aplikasi.

## 2. Scope of Work

- API client.
- Data fetching.
- Form submission.
- API response handling.
- State synchronization.
- Loading.
- Error mapping.
- Retry.
- End-to-end core flow.

## 3. Detailed Implementation Steps

### Step 1 — Implement API Client

Buat:

```
src/features/transcript/services/
└── transcript-api.ts
```

Contoh:

```
export async function getTranscript(
  videoId: string,
  language?: string
) {
  const response = await fetch("/api/transcript", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      videoId,
      language,
    }),
  });

  const result = await response.json();

  if (!response.ok) {
    throw new Error(
      result.error?.message ?? "Request failed"
    );
  }

  return result;
}
```

Implementasikan parsing error yang sesuai dengan API contract, bukan hanya menggunakan generic error seperti contoh sederhana di atas.

### Step 2 — Replace Mock Service

Hapus pemanggilan mock dari production flow.

Mock boleh tetap tersedia untuk testing.

### Step 3 — Connect Form

Alur:

```
Input URL
   |
   v
Extract Video ID
   |
   v
Submit
   |
   v
API Request
   |
   v
Backend
   |
   v
Transcript
   |
   v
Render Result
```

### Step 4 — Synchronize State

Ketika request dimulai:

- Loading true.
- Error direset.
- Result lama ditangani secara konsisten.

Ketika sukses:

- Loading false.
- Transcript diperbarui.
- Error direset.

Ketika gagal:

- Loading false.
- Error ditampilkan.
- Input tetap tersedia.

### Step 5 — Implement Retry

Pengguna dapat mencoba kembali tanpa harus memasukkan URL ulang.

Retry tidak boleh berlangsung otomatis tanpa batas.

### Step 6 — Test Core User Flow

Gunakan video pengujian yang diketahui memiliki caption dan dapat diakses.

## 4. Required Commands

```
npm run dev
npm run lint
npx tsc --noEmit
npm run build
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 7

# PHASE 7 — FRONTEND & BACKEND INTEGRATION

Kamu bertindak sebagai Senior Fullstack Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

Task: Implementasikan PHASE 7 saja.

## Objectives

Integrasikan frontend dengan backend transcript API.

## Implementation Tasks

1. Audit frontend dan backend.
2. Implementasikan API client.
3. Hubungkan form dengan API.
4. Ganti mock service dengan backend nyata.
5. Implementasikan response parsing.
6. Implementasikan error mapping.
7. Sinkronkan loading state.
8. Sinkronkan success state.
9. Sinkronkan error state.
10. Implementasikan retry action.
11. Pastikan copy transcript menggunakan data aktual.
12. Pastikan clear action berfungsi.
13. Verifikasi end-to-end user flow.

## Critical Rules

- Jangan menggunakan mock sebagai production data.
- Jangan mengubah API contract tanpa alasan dan dokumentasi.
- Jangan menambahkan database.
- Jangan memasang Axios tanpa kebutuhan.
- Jangan menambahkan fitur P1.
- Pertahankan UI yang sudah dibuat.
- Ikuti technical architecture.
- Jangan menghapus existing functionality.

## Expected User Flow

User opens website.\
User enters YouTube URL.\
User clicks Get Transcript.\
Frontend validates URL.\
Frontend calls API.\
Backend retrieves transcript.\
Frontend renders actual transcript.\
User copies transcript.

## Acceptance Criteria

- End-to-end core flow berhasil.
- Data berasal dari provider sebenarnya.
- Loading state benar.
- Error state benar.
- Retry berfungsi.
- Copy menggunakan transcript aktual.
- Tidak ada production mock dependency.
- Tidak ada TypeScript error.

## Testing Requirements

Test:

- Valid video.
- Invalid URL.
- Video without captions.
- Provider failure.
- Retry.
- Copy transcript.
- Clear result.

Jalankan lint, type check, dan build.

## Final Report

Laporkan integrasi yang dilakukan, files changed, mock yang dihapus dari production path, hasil testing aktual, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 8.

## 6. Expected Deliverables

```
src/features/transcript/
├── services/
│   └── transcript-api.ts
├── components/
│   ├── transcript-form.tsx
│   └── transcript-result.tsx
└── hooks/
    └── use-transcript.ts
```

Hook bersifat opsional apabila implementasi langsung pada component sudah cukup sederhana.

## 7. Acceptance Criteria

- Frontend terhubung ke API.
- Mock tidak digunakan pada production flow.
- Transcript aktual ditampilkan.
- Loading berjalan.
- Error mapping berjalan.
- Retry berfungsi.
- Copy berfungsi.
- Clear berfungsi.
- Core user flow end-to-end berhasil.

## 8. Testing Requirements

Lakukan integration testing untuk seluruh core user flow.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 7: frontend backend integration"
```

# PHASE 8 — Additional Features & Product Completion

Product Completion

## 1. Phase Objective

Menyelesaikan fitur pendukung P1 dan meningkatkan kualitas pengalaman pengguna.

Jangan implementasikan fitur P2.

## 2. Scope of Work

Prioritas:

1. Language selection.
2. Timestamp display.
3. Copy individual segment.
4. Download TXT.
5. Video metadata jika provider mendukungnya.
6. SEO.
7. Accessibility.
8. UX improvements.

## 3. Detailed Implementation Steps

### Step 1 — Language Selection

Jika provider mendukung daftar bahasa:

- Ambil daftar bahasa.
- Tampilkan pilihan bahasa.
- Pengguna memilih bahasa.
- Request ulang transcript berdasarkan bahasa.

Jangan menampilkan bahasa yang sebenarnya tidak tersedia.

### Step 2 — Timestamp Display

Format:

```
[00:00] Hello everyone

[00:05] Welcome to this video

[00:10] Today we will discuss...
```

Timestamp berasal dari offset segmen.

### Step 3 — Copy Individual Segment

Setiap segmen dapat disalin.

Fitur ini opsional apabila UI menjadi terlalu kompleks.

### Step 4 — Download TXT

Gunakan Blob API untuk membuat file `.txt` di browser.

Contoh nama:

```
youtube-transcript-VIDEO_ID.txt
```

Jangan mengirim transcript ke storage eksternal.

### Step 5 — SEO

Tambahkan:

- Page title.
- Meta description.
- Open Graph metadata.
- Semantic headings.
- Favicon.
- Sitemap jika diperlukan.
- Robots configuration.

### Step 6 — Accessibility

- Keyboard navigation.
- Focus states.
- Accessible labels.
- ARIA live region untuk status request.
- Kontras warna.

### Step 7 — Final UX Improvements

- Better empty state.
- Better loading state.
- Error recovery.
- Copy feedback.
- Text readability.

## 4. Required Commands

Tidak ada dependency tambahan yang diwajibkan.

```
npm run lint
npx tsc --noEmit
npm run build
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 8

# PHASE 8 — ADDITIONAL FEATURES & PRODUCT COMPLETION

Kamu bertindak sebagai Senior Fullstack Developer dan Product Engineer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

UI reference: UI_UX_GUIDE.md

Task: Implementasikan PHASE 8 saja.

## Objectives

Implementasikan fitur pendukung P1 dan selesaikan pengalaman produk.

## Priority Features

1. Language selection jika provider mendukungnya.
2. Timestamp display.
3. Copy individual segment.
4. Download TXT.
5. Video metadata jika tersedia.
6. SEO metadata.
7. Accessibility improvements.
8. UX improvements.

## Implementation Rules

- Implementasikan hanya fitur P1.
- Jangan implementasikan fitur AI.
- Jangan implementasikan AI Summary.
- Jangan implementasikan AI Translation.
- Jangan implementasikan AI Chat.
- Jangan membuat login.
- Jangan membuat database.
- Jangan membuat history.
- Jangan mengubah provider tanpa alasan.
- Jangan mengarang metadata atau bahasa.
- Jangan menambahkan dependency yang tidak diperlukan.

## Technical Requirements

- Pertahankan timestamp jika tersedia.
- Gunakan browser Blob API untuk TXT download.
- Pertahankan transcript asli.
- Pastikan language selection terhubung ke backend.
- Pastikan SEO metadata relevan.
- Perhatikan accessibility.

## Acceptance Criteria

- Fitur P1 yang disetujui berfungsi.
- Language selection sesuai ketersediaan provider.
- Timestamp akurat terhadap data provider.
- TXT download berhasil.
- SEO metadata tersedia.
- Accessibility diperbaiki.
- Core functionality tidak rusak.

## Testing Requirements

Test seluruh fitur tambahan dan regression test core flow.

Jalankan lint, type check, dan build.

## Final Report

Laporkan fitur yang selesai, fitur yang tidak diimplementasikan beserta alasannya, dependency, hasil testing, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 9.

## 6. Expected Deliverables

Fitur P1 yang telah diimplementasikan dan dokumentasi pendukung jika diperlukan.

## 7. Acceptance Criteria

- Language selection jika tersedia.
- Timestamp display.
- Copy segment.
- Download TXT.
- SEO metadata.
- Accessibility improvements.
- Tidak ada fitur P2.
- Core functionality tetap berjalan.

## 8. Testing Requirements

- Language selection.
- Timestamp rendering.
- Copy segment.
- TXT download.
- SEO metadata.
- Responsive regression.
- Core transcript flow.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 8: additional features"
```

# PHASE 9 — Testing, Security & Optimization

Quality Assurance

## 1. Phase Objective

Memastikan seluruh aplikasi stabil, aman, dan siap masuk production.

Output:

`TEST_REPORT.md`

Tidak boleh menambahkan fitur baru pada phase ini.

## 2. Scope of Work

- Functional testing.
- Unit testing.
- Integration testing.
- E2E testing jika relevan.
- Type checking.
- Linting.
- Production build.
- Responsive testing.
- Browser compatibility.
- Security audit.
- Performance optimization.
- Bug fixing.

## 3. Detailed Implementation Steps

### Step 1 — Audit Project

Periksa:

- Struktur folder.
- Dependencies.
- API routes.
- Provider integration.
- Environment variables.
- Error handling.

### Step 2 — Unit Testing

Prioritas:

- URL validation.
- Video ID extraction.
- Transcript normalization.
- Error mapping.

Jika belum ada testing framework, gunakan Vitest.

Install:

```
npm install -D vitest
```

Tambahkan konfigurasi yang sesuai dengan TypeScript dan Next.js project.

### Step 3 — Integration Testing

Test:

- API valid request.
- API invalid request.
- Provider success.
- Provider failure.
- Timeout.
- Rate limit.

Gunakan mock provider pada automated integration tests jika diperlukan, tetapi tetap lakukan pengujian terpisah terhadap provider sebenarnya.

### Step 4 — E2E Testing

Gunakan Playwright jika sesuai.

```
npm install -D @playwright/test
npx playwright install
```

Test:

- Homepage loads.
- Input URL.
- Submit.
- Loading.
- Transcript result.
- Copy.
- Error.

### Step 5 — Responsive Testing

Test ukuran:

| Device        | Width  |
| ------------- | ------ |
| Mobile Small  | 375px  |
| Mobile        | 390px  |
| Tablet        | 768px  |
| Desktop       | 1366px |
| Large Desktop | 1920px |

### Step 6 — Security Audit

Periksa:

- Input validation.
- SSRF.
- Rate limiting.
- Secret exposure.
- Error leakage.
- Dependency vulnerabilities.
- Request timeout.
- Response size.
- Abuse prevention.

Jalankan:

```
npm audit
```

Evaluasi hasilnya sebelum melakukan upgrade dependency.

### Step 7 — Performance Optimization

Periksa:

- Rendering.
- Bundle size.
- Unnecessary client components.
- Large transcript rendering.
- API response time.
- External provider latency.

### Step 8 — Build Verification

```
npm run lint
npx tsc --noEmit
npm run build
```

### Step 9 — Generate Test Report

Buat:

```
TEST_REPORT.md
```

Isi:

1. Testing scope.
2. Test environment.
3. Unit test results.
4. Integration test results.
5. E2E test results.
6. Security audit.
7. Performance review.
8. Bugs found.
9. Bugs fixed.
10. Known issues.
11. Release readiness.

## 4. Required Commands

```
npm run lint

npx tsc --noEmit

npm run build

npm audit
```

Jika Vitest tersedia:

```
npx vitest run
```

Jika Playwright tersedia:

```
npx playwright test
```

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 9

# PHASE 9 — TESTING, SECURITY & OPTIMIZATION

Kamu bertindak sebagai Senior QA Engineer, Security Engineer, dan Performance Engineer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

Task: Selesaikan PHASE 9 saja.

## Objectives

Melakukan pengujian menyeluruh terhadap aplikasi sebelum production deployment.

Buat TEST_REPORT.md.

## Testing Tasks

1. Audit keseluruhan project.
2. Unit testing.
3. Integration testing.
4. E2E testing jika tersedia.
5. Type checking.
6. Linting.
7. Production build.
8. Responsive testing.
9. Browser compatibility testing.
10. Security audit.
11. Dependency audit.
12. Performance review.
13. Bug fixing.

## Critical Rules

- Jangan menambahkan fitur baru.
- Jangan mengubah scope MVP.
- Jangan menghapus functionality.
- Jangan mengklaim test berhasil tanpa menjalankannya.
- Jangan menyembunyikan test failure.
- Jangan melakukan upgrade dependency besar tanpa alasan.
- Jangan mengubah arsitektur tanpa persetujuan.
- Jangan menganggap mock test sebagai bukti provider production berfungsi.
- Dokumentasikan known issues.

## Required Test Scenarios

Functional:

- Valid YouTube URL.
- Invalid URL.
- Supported URL formats.
- Video with captions.
- Video without captions.
- Provider error.
- Timeout.
- Copy transcript.
- Language selection jika tersedia.
- TXT download jika tersedia.

Security:

- Malformed input.
- Invalid Video ID.
- SSRF protection.
- Rate limiting.
- Environment secrets.
- Error leakage.
- Dependency vulnerabilities.

Performance:

- Initial page load.
- Transcript rendering.
- Large transcript handling.
- API latency.
- Bundle size.

## Required Output

TEST_REPORT.md.

Dokumen harus berisi hasil pengujian aktual, bukan estimasi.

## Acceptance Criteria

- Lint berhasil.
- Type check berhasil.
- Build berhasil.
- Core user flow teruji.
- Critical bugs diperbaiki.
- Security findings terdokumentasi.
- Known issues tercatat.
- Release readiness dinilai.

Jika ditemukan blocker, jangan menyatakan aplikasi siap production.

## Final Report

Laporkan hasil seluruh testing, bugs, security findings, optimizations, known issues, dan Git checkpoint readiness.

Jangan melanjutkan ke Phase 10.

## 6. Expected Deliverables

```
TEST_REPORT.md
```

## 7. Acceptance Criteria

- Unit test berhasil.
- Integration test berhasil.
- E2E test sesuai scope.
- Lint berhasil.
- Type check berhasil.
- Build berhasil.
- Security audit dilakukan.
- Responsive testing dilakukan.
- Bug kritis diperbaiki.
- TEST_REPORT.md tersedia.

## 8. Testing Requirements

Semua hasil testing harus berdasarkan eksekusi aktual.

Jika ada test yang belum dapat dilakukan, tandai sebagai Not Tested.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 9: testing and optimization"
```

# PHASE 10 — Production Deployment & Release

Production Release

## 1. Phase Objective

Melakukan deployment aplikasi ke production.

Target:

- Website dapat diakses publik.
- HTTPS aktif.
- Environment variables aman.
- Backend berfungsi.
- Provider berfungsi pada production.
- Monitoring tersedia.
- Dokumentasi deployment selesai.

## 2. Scope of Work

- Production configuration.
- Environment variables.
- Build optimization.
- Hosting.
- Domain.
- DNS.
- HTTPS.
- Deployment.
- Monitoring.
- Logging.
- Backup configuration jika diperlukan.
- Production smoke test.
- Release documentation.

## 3. Detailed Implementation Steps

### Step 1 — Determine Hosting

Untuk MVP, gunakan salah satu:

| Platform                | Use Case                                             |
| ----------------------- | ---------------------------------------------------- |
| Vercel                  | Deployment Next.js yang sederhana                    |
| VPS                     | Kontrol penuh terhadap runtime                       |
| Managed Node.js Hosting | Deployment Node.js tanpa mengelola server sepenuhnya |

Rekomendasi: Vercel jika provider transcript kompatibel dengan runtime dan batas waktu eksekusinya.

Gunakan VPS jika:

- Provider membutuhkan proses yang berjalan lebih lama.
- Library membutuhkan runtime khusus.
- Membutuhkan kontrol concurrency.
- Membutuhkan persistent process.
- Batasan serverless mengganggu operasi.

Jangan memindahkan backend ke VPS jika Next.js Route Handlers sudah mencukupi.

### Step 2 — Production Environment Variables

Siapkan:

```
NODE_ENV=production

TRANSCRIPT_PROVIDER=default

TRANSCRIPT_PROVIDER_API_KEY=

RATE_LIMIT_ENABLED=true
```

Hanya masukkan variabel yang benar-benar dibutuhkan.

### Step 3 — Build Verification

```
npm run build
```

### Step 4 — Deployment Using Vercel

Install CLI jika diperlukan:

```
npm install -g vercel
```

Login:

```
vercel login
```

Deploy preview:

```
vercel
```

Deploy production:

```
vercel --prod
```

Alternatifnya, hubungkan GitHub repository ke Vercel dan gunakan automated deployment.

### Step 5 — VPS Deployment (Alternative)

Jika menggunakan VPS:

Install Node.js dan Nginx sesuai OS server.

Build:

```
npm ci
npm run build
```

Jalankan:

```
npm start
```

Gunakan process manager seperti PM2 jika memang diperlukan.

Nginx digunakan sebagai reverse proxy.

Docker bersifat opsional, bukan persyaratan MVP.

### Step 6 — Configure Domain

Contoh:

```
transscipt-yt.com
```

Konfigurasikan:

- DNS A record atau CNAME.
- Domain verification.
- HTTPS.
- Redirect HTTP ke HTTPS.

Nama domain tersebut hanya contoh dan belum tentu tersedia.

### Step 7 — Monitoring

Monitor:

- Application errors.
- API errors.
- Provider failures.
- Request duration.
- Rate limit.
- Server resources.
- Deployment status.

Gunakan logging platform hosting atau layanan monitoring yang sesuai.

### Step 8 — Production Smoke Test

Test:

1. Homepage dapat dibuka.
2. HTTPS aktif.
3. URL valid dapat diproses.
4. Transcript ditampilkan.
5. Copy berfungsi.
6. Error handling berfungsi.
7. Provider dapat diakses.
8. Rate limiting aktif.
9. Tidak ada secret yang terekspos.

### Step 9 — Create Documentation

Buat:

```
DEPLOYMENT_GUIDE.md
RELEASE_NOTES.md
```

### DEPLOYMENT_GUIDE.md

Isi:

- Architecture.
- Hosting platform.
- Deployment steps.
- Environment variables.
- Domain configuration.
- HTTPS.
- Monitoring.
- Troubleshooting.
- Rollback.

### RELEASE_NOTES.md

Isi:

- Product overview.
- MVP features.
- Technical stack.
- Release version.
- Deployment date.
- Known limitations.
- Future improvements.

## 4. Required Commands

```
npm ci

npm run lint

npx tsc --noEmit

npm run build

npm start
```

Untuk Vercel:

```
vercel login
vercel
vercel --prod
```

Untuk VPS, gunakan prosedur deployment yang sesuai dengan hosting.

## 5. AI Coding Assistant Prompt

COPY PROMPT — PHASE 10

# PHASE 10 — PRODUCTION DEPLOYMENT & RELEASE

Kamu bertindak sebagai Senior DevOps Engineer dan Senior Fullstack Developer.

Project: transscipt-yt

Dokumen utama: PROJECT_PRD_AND_PLAYBOOK.md

Technical reference: TECHNICAL_PLAN.md

Testing reference: TEST_REPORT.md

Task: Selesaikan PHASE 10 saja.

## Objectives

Deploy aplikasi transscipt-yt ke production dan buat dokumentasi release.

## Implementation Tasks

1. Audit production readiness.
2. Tentukan hosting sesuai architecture.
3. Review production configuration.
4. Setup environment variables.
5. Verify production build.
6. Configure deployment.
7. Configure domain jika tersedia.
8. Configure HTTPS.
9. Configure monitoring.
10. Verify provider integration.
11. Execute production smoke test.
12. Document deployment.
13. Document release.

## Hosting Rules

Pertimbangkan Vercel terlebih dahulu jika kompatibel dengan provider dan runtime.

Gunakan VPS jika membutuhkan runtime atau resource yang tidak sesuai dengan serverless.

Jangan menambahkan Docker tanpa kebutuhan.

Jangan mengubah arsitektur backend tanpa alasan.

## Security Requirements

- Jangan commit .env.
- Jangan expose API key.
- Gunakan HTTPS.
- Pastikan rate limiting aktif.
- Pastikan production environment variables benar.
- Pastikan error tidak membocorkan informasi internal.

## Required Deliverables

1. DEPLOYMENT_GUIDE.md
2. RELEASE_NOTES.md

## Smoke Test

Test:

- Homepage.
- HTTPS.
- Transcript API.
- Valid video.
- Invalid video.
- Video without captions.
- Copy transcript.
- Error handling.
- Provider availability.
- Rate limiting.

## Acceptance Criteria

- Website dapat diakses publik.
- HTTPS aktif.
- Environment variables benar.
- Production build berhasil.
- Provider berfungsi.
- Smoke test berhasil.
- Monitoring tersedia.
- Deployment guide tersedia.
- Release notes tersedia.

Jangan mengklaim deployment berhasil jika tidak memiliki akses atau belum melakukan deployment aktual.

Jika deployment membutuhkan tindakan manual dari pemilik akun hosting, siapkan instruksi yang jelas.

## Final Report

Laporkan:

1. Hosting.
2. Production URL.
3. Deployment configuration.
4. Environment variables.
5. Domain dan HTTPS.
6. Monitoring.
7. Smoke test results.
8. Known limitations.
9. DEPLOYMENT_GUIDE.md.
10. RELEASE_NOTES.md.

Jangan membuat fitur tambahan.

## 6. Expected Deliverables

```
transscipt-yt/
├── DEPLOYMENT_GUIDE.md
└── RELEASE_NOTES.md
```

## 7. Acceptance Criteria

- Production build berhasil.
- Hosting tersedia.
- Website online.
- HTTPS aktif.
- Environment variables benar.
- Provider berfungsi.
- Monitoring tersedia.
- Smoke test berhasil.
- Deployment guide tersedia.
- Release notes tersedia.

## 8. Testing Requirements

Lakukan production smoke test.

Pastikan test menggunakan URL production, bukan hanya localhost.

## 9. Git Checkpoint

```
git add .
git commit -m "phase 10: production deployment"
```

# PART C — GLOBAL AI CODING RULES

Aturan berikut berlaku untuk seluruh 10 phase.

## 1. General Development Rules

1. Selalu membaca `PROJECT_PRD_AND_PLAYBOOK.md` sebelum bekerja.
2. Membaca `TECHNICAL_PLAN.md` setelah Phase 2 selesai.
3. Membaca `UI_UX_GUIDE.md` setelah Phase 3 selesai.
4. Memeriksa repository sebelum mengubah file.
5. Tidak menghapus pekerjaan existing tanpa alasan.
6. Tidak mengubah tech stack tanpa persetujuan.
7. Tidak menambahkan dependency tanpa kebutuhan.
8. Tidak mengerjakan phase berikutnya secara otomatis.
9. Tidak mengarang hasil testing.
10. Tidak menggunakan mock data untuk menggantikan production service setelah Phase 7.
11. Tidak membuat database tanpa kebutuhan produk.
12. Tidak membuat abstraction berlebihan.
13. Menggunakan TypeScript untuk application code.
14. Mempertahankan konsistensi naming dan folder structure.
15. Melaporkan seluruh perubahan.

## 2. Phase Execution Rules

Setiap phase wajib mengikuti urutan:

```
READ DOCUMENTATION
       |
       v
INSPECT REPOSITORY
       |
       v
ANALYZE REQUIREMENTS
       |
       v
IMPLEMENT CURRENT PHASE
       |
       v
RUN TESTS
       |
       v
REVIEW CHANGES
       |
       v
REPORT RESULTS
       |
       v
USER APPROVAL
       |
       v
GIT CHECKPOINT
       |
       v
NEXT PHASE
```

## 3. Dependency Management

Sebelum memasang dependency, AI harus menjelaskan:

- Nama package.
- Fungsi package.
- Mengapa dibutuhkan.
- Apakah tersedia alternatif native.
- Apakah kompatibel dengan project.

Contoh:

| Dependency   | Decision | Reason                            |
| ------------ | -------- | --------------------------------- |
| Next.js      | Required | Application framework             |
| TypeScript   | Required | Type safety                       |
| Tailwind CSS | Required | Styling                           |
| shadcn/ui    | Required | Reusable UI                       |
| Zustand      | Optional | Belum diperlukan untuk MVP        |
| Axios        | Optional | Native fetch mencukupi            |
| Zod          | Optional | Schema validation jika dibutuhkan |
| Vitest       | Phase 9  | Unit testing                      |
| Playwright   | Phase 9  | E2E testing                       |

## 4. Code Quality Rules

- Gunakan reusable components.
- Pisahkan UI dan business logic.
- Hindari file terlalu besar.
- Hindari duplikasi fungsi.
- Gunakan error handling yang konsisten.
- Gunakan strict TypeScript.
- Hindari `any` tanpa alasan.
- Gunakan semantic HTML.
- Gunakan naming yang deskriptif.
- Hindari premature optimization.

## 5. Mandatory Phase Report

Setelah menyelesaikan setiap phase, AI wajib memberikan laporan:

```
PHASE REPORT

Phase:
Phase Name:

1. Summary
   - Apa yang telah dikerjakan.

2. Files Created
   - Daftar file baru.

3. Files Modified
   - Daftar file yang diubah.

4. Dependencies
   - Dependency yang ditambahkan.
   - Alasan penambahan.

5. Commands Executed
   - Command yang benar-benar dijalankan.

6. Testing Results
   - Passed.
   - Failed.
   - Not Tested.

7. Known Issues
   - Kendala yang ditemukan.

8. Acceptance Criteria
   - Completed.
   - Not Completed.

9. Git Checkpoint
   - Ready / Not Ready.

10. Next Phase
    - Phase berikutnya yang direkomendasikan.
```

# PART D — GIT & EXECUTION RULES

## 1. Git Strategy

Gunakan Git sebagai checkpoint setiap phase.

Branch utama:

```
main
```

Untuk MVP solo developer, cukup menggunakan satu branch utama jika seluruh pekerjaan direview sebelum commit.

Branch terpisah dapat digunakan jika diperlukan.

## 2. Initial Git Setup

```
git init
git branch -M main
```

## 3. Git Checkpoint per Phase

| Phase    | Commit Message                          |
| -------- | --------------------------------------- |
| Phase 1  | `phase 1: project bootstrap`            |
| Phase 2  | `phase 2: technical architecture`       |
| Phase 3  | `phase 3: ui ux planning`               |
| Phase 4  | `phase 4: frontend foundation`          |
| Phase 5  | `phase 5: core frontend features`       |
| Phase 6  | `phase 6: backend implementation`       |
| Phase 7  | `phase 7: frontend backend integration` |
| Phase 8  | `phase 8: additional features`          |
| Phase 9  | `phase 9: testing and optimization`     |
| Phase 10 | `phase 10: production deployment`       |

## 4. Git Workflow

Sebelum commit:

```
git status
git diff
```

Stage perubahan:

```
git add .
```

Commit sesuai phase:

```
git commit -m "phase 1: project bootstrap"
```

Verifikasi:

```
git log --oneline
```

## 5. Git Rules

- Jangan commit jika testing wajib belum dijalankan.
- Jangan commit file `.env`.
- Jangan commit `node_modules`.
- Jangan commit build artifacts.
- Jangan memasukkan API keys.
- Jangan membuat commit untuk pekerjaan yang belum direview.
- Jika ada acceptance criteria yang belum terpenuhi, selesaikan atau dokumentasikan blocker sebelum checkpoint.

## 6. Phase Completion Tracking

Gunakan checklist:

```
# DEVELOPMENT PROGRESS

- [ ] Phase 1 — Environment & Project Bootstrap
- [ ] Phase 2 — Technical Planning & Architecture
- [ ] Phase 3 — UI/UX Planning & Design System
- [ ] Phase 4 — Frontend Foundation & Application Shell
- [ ] Phase 5 — Core Frontend Feature Development
- [ ] Phase 6 — Backend & Core Business Logic
- [ ] Phase 7 — Frontend & Backend Integration
- [ ] Phase 8 — Additional Features & Product Completion
- [ ] Phase 9 — Testing, Security & Optimization
- [ ] Phase 10 — Production Deployment & Release
```

Centang phase hanya setelah acceptance criteria terpenuhi dan Git checkpoint dibuat.

# PART E — COMMAND CHEAT SHEET

Kumpulan command yang digunakan selama development.

## 1. Environment

```
node --version
npm --version
git --version
```

## 2. Project Setup

```
npx create-next-app@latest transscipt-yt
cd transscipt-yt
npm install
```

## 3. shadcn/ui

```
npx shadcn@latest init

npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add card
npx shadcn@latest add textarea
npx shadcn@latest add alert
npx shadcn@latest add skeleton
```

## 4. Development

```
npm run dev
```

## 5. Code Quality

```
npm run lint

npx tsc --noEmit

npm run build
```

## 6. Testing

```
npm install -D vitest

npx vitest run
```

Playwright:

```
npm install -D @playwright/test

npx playwright install

npx playwright test
```

## 7. Dependency Audit

```
npm audit

npm outdated
```

## 8. Git

```
git status

git add .

git commit -m "phase 1: project bootstrap"

git log --oneline

git diff
```

## 9. Production

```
npm ci

npm run build

npm start
```

Vercel:

```
npm install -g vercel

vercel login

vercel

vercel --prod
```

# PART F — FINAL CHECKLIST & DOCUMENTATION

## 1. Final Project Structure

Target struktur akhir:

```
transscipt-yt/
│
├── public/
│   ├── favicon.ico
│   └── images/
│
├── src/
│   │
│   ├── app/
│   │   ├── api/
│   │   │   └── transcript/
│   │   │       └── route.ts
│   │   │
│   │   ├── privacy/
│   │   │   └── page.tsx
│   │   │
│   │   ├── terms/
│   │   │   └── page.tsx
│   │   │
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── navbar.tsx
│   │   │   └── footer.tsx
│   │   │
│   │   └── ui/
│   │
│   ├── features/
│   │   └── transcript/
│   │       │
│   │       ├── components/
│   │       │   ├── transcript-form.tsx
│   │       │   ├── transcript-result.tsx
│   │       │   ├── transcript-toolbar.tsx
│   │       │   ├── transcript-loading.tsx
│   │       │   └── transcript-error.tsx
│   │       │
│   │       ├── application/
│   │       │   └── get-transcript.service.ts
│   │       │
│   │       ├── domain/
│   │       │   ├── transcript.types.ts
│   │       │   └── transcript.errors.ts
│   │       │
│   │       ├── infrastructure/
│   │       │   └── providers/
│   │       │       └── youtube-transcript.provider.ts
│   │       │
│   │       ├── services/
│   │       │   └── transcript-api.ts
│   │       │
│   │       ├── utils/
│   │       │   └── youtube-url.ts
│   │       │
│   │       └── hooks/
│   │           └── use-transcript.ts
│   │
│   ├── lib/
│   │   └── utils.ts
│   │
│   └── types/
│
├── .env.example
├── .gitignore
├── components.json
├── next.config.ts
├── package.json
├── package-lock.json
├── tsconfig.json
│
├── PROJECT_PRD_AND_PLAYBOOK.md
├── TECHNICAL_PLAN.md
├── UI_UX_GUIDE.md
├── TEST_REPORT.md
├── DEPLOYMENT_GUIDE.md
└── RELEASE_NOTES.md
```

Catatan: Struktur tersebut merupakan target organisasi project, bukan kewajiban membuat semua file kosong sejak Phase 1. Implementasi mengikuti kebutuhan setiap phase.

## 2. Documentation Checklist

| Document                    | Created At |
| --------------------------- | ---------- |
| PROJECT_PRD_AND_PLAYBOOK.md | Initial    |
| TECHNICAL_PLAN.md           | Phase 2    |
| UI_UX_GUIDE.md              | Phase 3    |
| TEST_REPORT.md              | Phase 9    |
| DEPLOYMENT_GUIDE.md         | Phase 10   |
| RELEASE_NOTES.md            | Phase 10   |

## 3. Final Product Checklist

### Core Features

- YouTube URL input.
- URL validation.
- Video ID extraction.
- Transcript retrieval.
- Transcript display.
- Copy transcript.
- Loading state.
- Error handling.
- Language information.

### Additional Features

- Language selection jika tersedia.
- Timestamp.
- Copy individual segment.
- Download TXT.
- SEO.
- Responsive design.

### Technical

- Next.js.
- TypeScript.
- Tailwind CSS.
- shadcn/ui.
- Backend API.
- Provider adapter.
- Error mapping.
- Rate limiting.
- Security validation.
- No unnecessary database.

### Production

- Production build.
- HTTPS.
- Domain.
- Environment variables.
- Monitoring.
- Production smoke test.
- Deployment documentation.
- Release notes.

## 4. Final Definition of Done

Project dinyatakan selesai ketika:

1. Seluruh 10 phase telah diselesaikan.
2. Seluruh acceptance criteria telah dipenuhi.
3. Seluruh Git checkpoint tersedia.
4. Transcript provider dapat digunakan pada production.
5. Core user flow berfungsi end-to-end.
6. Tidak ada bug kritis yang diketahui.
7. Production deployment berhasil.
8. Dokumentasi tersedia.
9. Website dapat digunakan oleh publik.

## Rekomendasi urutan pengerjaan

Untuk menghindari kesalahan saat menggunakan AI Coding Assistant, saya menyarankan urutan berikut:

| Urutan | Yang dilakukan                                      |
| ------ | --------------------------------------------------- |
| 1      | Simpan dokumen utama di root repository             |
| 2      | Jalankan prompt Phase 1                             |
| 3      | Review hasil dan commit                             |
| 4      | Jalankan prompt Phase 2                             |
| 5      | Pastikan transcript provider sudah tervalidasi      |
| 6      | Lanjutkan Phase 3–5 untuk frontend                  |
| 7      | Lanjutkan Phase 6–7 untuk backend dan integrasi     |
| 8      | Lanjutkan Phase 8–10 untuk penyelesaian dan release |

Perhatian khusus: Jangan melewati validasi transcript provider pada Phase 2. Ini merupakan risiko teknis terbesar project transscipt-yt. UI dapat dibuat dengan mudah, tetapi keberhasilan produk bergantung pada kemampuan mengambil caption YouTube secara konsisten dan sesuai ketentuan layanan.

Dokumen ini menggunakan tepat 10 phase dan tidak memasukkan fitur AI, database, atau infrastruktur tambahan ke dalam MVP.

Langkah visual berikutnya yang paling relevan adalah membuat mockup UI homepage dan halaman hasil transcript sebelum menjalankan Phase 3.

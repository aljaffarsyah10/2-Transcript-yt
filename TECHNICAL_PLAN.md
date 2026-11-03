# TECHNICAL PLAN — transscipt-yt

**Document Version:** 1.0.0  
**Phase:** Phase 2 — Technical Planning & Architecture  
**Project:** `transscipt-yt` (YouTube Transcript Converter)  
**Status:** Approved & Verified via PoC

---

## 1. Architecture Overview

`transscipt-yt` mengadopsi arsitektur **Modular Monolith** berbasis **Next.js App Router** dengan **Hexagonal/Ports & Adapters Pattern** untuk integrasi eksternal.

### Prinsip Arsitektur:
1. **Separation of Concerns:** Pemisahan ketat antara lapisan presentasi (UI), orkestrasi aplikasi (Application Service), definisi domain & aturan bisnis (Domain), serta integrasi pihak ketiga (Infrastructure).
2. **Provider Agnostic (Adapter Pattern):** Pengambilan transcript YouTube dibungkus dalam `TranscriptProvider` interface sehingga library atau service eksternal dapat diganti tanpa merusak logika aplikasi atau UI.
3. **Stateless Backend:** Endpoint API bersifat murni stateless, tidak memerlukan persistensi database untuk MVP, dan hanya memproses video ID yang tervalidasi.
4. **Performance & Lightweight:** Menggunakan native `fetch` pada sisi klien dan React State lokal untuk UI tanpa state management eksternal yang berlebihan.

```
                    +---------------------------+
                    |        User Browser       |
                    +---------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
|                         Next.js App Router                            |
|                                                                       |
|  [Presentation Layer]                                                 |
|  - Hero Section, YouTubeInput Form, TranscriptResult, Toolbar         |
|  - React State (url, loading, transcript, error)                      |
|                                 |                                     |
|                                 v (native fetch POST /api/transcript) |
|  [API & Route Handler Layer]                                          |
|  - src/app/api/transcript/route.ts                                    |
|  - Input Validation & Rate Limiting                                   |
|                                 |                                     |
|                                 v                                     |
|  [Application Layer]                                                  |
|  - TranscriptApplicationService (src/features/transcript/application) |
|                                 |                                     |
|                                 v (uses Port/Interface)               |
|  [Domain Layer]                                                       |
|  - Transcript, TranscriptSegment, TranscriptError, Result Types       |
|                                 |                                     |
|                                 v (implemented by)                    |
|  [Infrastructure Layer]                                               |
|  - YoutubeTranscriptAdapter (src/features/transcript/infrastructure)  |
|                                 |                                     |
+---------------------------------|-------------------------------------+
                                  v
                     +--------------------------+
                     | YouTube TimedText/Captions|
                     +--------------------------+
```

---

## 2. Technology Decisions

| Komponen | Pilihan Teknologi | Alasan Pemilihan |
| :--- | :--- | :--- |
| **Framework** | Next.js 16+ (App Router) | Full-stack monorepo, rendering cepat dengan Turbopack, route handlers bawaan tanpa server express terpisah. |
| **Language** | TypeScript 5+ | Type safety end-to-end dari API response hingga komponen React. |
| **Styling** | Tailwind CSS v4 | Utility-first styling modern, zero-runtime overhead, responsif secara default. |
| **UI Components** | shadcn/ui | Komponen headless berbasis Radix UI, mudah di-customize, copy-paste ownership. |
| **Icons** | Lucide React | Standar modern shadcn/ui, ukuran bundle kecil dan pohon ikon lengkap. |
| **State Management** | Native React `useState` | Skala MVP terbatas pada single page flow; tidak memerlukan Zustand/Redux. |
| **HTTP Client** | Native `fetch` | Didukung langsung oleh browser modern dan Node.js runtime, zero-dependency. |
| **Database** | Tidak Ada (Stateless) | MVP tidak menyimpan riwayat, user profile, atau bookmark. |
| **Runtime Validation** | Zod | Skema validasi runtime yang aman untuk input API dan parsing URL/Video ID. |

---

## 3. Transcript Provider Evaluation

Kami mengevaluasi 4 pendekatan untuk mendapatkan transcript dari YouTube:

| Provider / Library | Kelebihan | Kekurangan / Batasan | Hasil Evaluasi |
| :--- | :--- | :--- | :--- |
| **1. YouTube Data API v3 (`captions.download`)** | API resmi Google / YouTube | Memerlukan OAuth 2.0 authorization dengan scope YouTube Partner/Channel Manager. Tidak dapat mengunduh caption video publik milik orang lain secara anonim. | **Ditolak:** Tidak memenuhi kebutuhan public anonymous video transcript. |
| **2. youtube-transcript (kakulukian v1.3.1)** | - Terbukti stabil dan zero-dependency tambahan.<br>- Langsung berjalan di Node.js runtime.<br>- Mendukung subtitle manual dan auto-generated.<br>- Error classes terdefinisi rapi. | Scraping internal player context; rentan jika YouTube mengubah response structure sewaktu-waktu. | **Dipilih sebagai Provider Utama:** Berhasil diverifikasi dalam PoC dengan response <1s pada multi-video. |
| **3. youtube-transcript-plus (v2.0.3)** | Menggunakan innertube player context. | Mengalami error connection terminated pada beberapa video populer (misal `dQw4w9WgXcQ`) dalam pengujian lokal. | **Ditolak sebagai primary:** Kurang stabil dibandingkan `youtube-transcript`. |
| **4. Third-Party API (e.g. RapidAPI / FlashData / Arcmira)** | Dikelola penyedia eksternal dengan proxy built-in. | Membutuhkan API Key berbayar, rate limit ketat pada free tier, latensi tambahan pihak ketiga. | **Dicadangkan:** Alternatif fallback jika YouTube melakukan IP blocking massal di cloud provider. |
| **5. Dedicated Python Service (`youtube-transcript-api`)** | Ekosistem Python sangat matang dalam parsing timedtext YouTube. | Menambah kompleksitas deployment (membutuhkan 2 runtime/container terpisah). | **Ditolak untuk MVP:** Terlalu berlebihan sebelum Node.js terbukti tidak memadai. |

---

## 4. Final Provider Decision

**Keputusan Final:**
1. Menggunakan library **`youtube-transcript`** sebagai implementasi adapter utama di `src/features/transcript/infrastructure/youtube-transcript.adapter.ts`.
2. Membungkus adapter di balik interface **`ITranscriptProvider`** pada domain layer.
3. Normalisasi unit: `youtube-transcript` mengembalikan `offset` dan `duration` dalam satuan milidetik (`ms`), sedangkan domain model transscipt-yt mendefinisikan waktu dalam detik (`seconds`). Adapter bertanggung jawab melakukan pembagian `1000` (`offset / 1000`, `duration / 1000`).
4. Menyediakan mekanisme timeout eksplisit (10.000 ms) menggunakan `Promise.race` untuk mencegah server hang.

---

## 5. Folder Structure

Struktur direktori modular dalam folder `src/`:

```
transscipt-yt/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   └── transcript/
│   │   │       └── route.ts              # Route handler POST /api/transcript
│   │   ├── layout.tsx                    # Root layout & meta tags
│   │   ├── page.tsx                      # Main single-page application
│   │   └── globals.css                   # Tailwind CSS imports & theme tokens
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── navbar.tsx                # Header navigasi & logo
│   │   │   └── footer.tsx                # Footer info & links
│   │   └── ui/                           # shadcn/ui primitives (button, input, card, etc.)
│   │
│   ├── features/
│   │   └── transcript/
│   │       ├── application/
│   │       │   └── transcript.service.ts # Use-case orchestration logic
│   │       ├── domain/
│   │       │   ├── transcript.entity.ts  # Domain interfaces & models
│   │       │   ├── transcript.types.ts   # Request/Response DTO types
│   │       │   └── transcript-errors.ts  # Standard error codes & classes
│   │       ├── infrastructure/
│   │       │   ├── transcript-provider.interface.ts # Port definition
│   │       │   └── youtube-transcript.adapter.ts    # Adapter implementation
│   │       ├── components/
│   │       │   ├── youtube-input-form.tsx # Input box with validation & clear button
│   │       │   ├── transcript-result.tsx  # Scrollable transcript display
│   │       │   ├── transcript-toolbar.tsx # Copy to clipboard & export buttons
│   │       │   ├── loading-state.tsx      # Skeleton & spinner indicators
│   │       │   └── error-state.tsx        # User-friendly error alert & retry action
│   │       └── utils/
│   │           ├── youtube-url-parser.ts # Strict regex & URL parser
│   │           └── timestamp-formatter.ts # Converts seconds to [MM:SS] or [HH:MM:SS]
│   │
│   └── lib/
│       ├── utils.ts                      # cn() helper
│       └── rate-limiter.ts               # In-memory sliding window limiter for Route Handler
│
├── proof-of-concept/
│   └── verify-transcript.mjs             # Empirically validated test script
├── PROJECT_PRD_AND_PLAYBOOK.md
├── TECHNICAL_PLAN.md
└── package.json
```

---

## 6. Data Flow

```
1. User memasukkan URL (misal: "https://youtu.be/dQw4w9WgXcQ")
2. Frontend (youtube-url-parser.ts) memvalidasi format URL dan mengekstrak videoId: "dQw4w9WgXcQ"
3. Jika URL tidak valid -> Muncul pesan error inline tanpa mengirim request.
4. Jika valid -> State `loading = true`, kirim POST request ke `/api/transcript`:
   { "videoId": "dQw4w9WgXcQ", "language": "en" }
5. Route Handler (/api/transcript/route.ts):
   a. Memeriksa rate limit IP client.
   b. Memvalidasi payload menggunakan schema (videoId wajib berformat YouTube ID 11 karakter).
   c. Memanggil TranscriptApplicationService.getTranscript(videoId, language).
6. TranscriptApplicationService memanggil ITranscriptProvider (YoutubeTranscriptAdapter).
7. YoutubeTranscriptAdapter mengambil raw timedtext dari YouTube dengan timeout 10 detik.
8. YoutubeTranscriptAdapter memetakan raw segments ke domain `TranscriptSegment` (mengubah ms ke detik, trim whitespace).
9. Response JSON dikirimkan ke frontend:
   { "success": true, "data": { "videoId": "...", "transcript": "...", "segments": [...] } }
10. Frontend menerima data -> State `loading = false`, `transcript = data`, `error = null`.
11. Komponen `TranscriptResult` menampilkan isi transcript dan mengaktifkan tombol Copy.
```

---

## 7. API Specification

### Endpoint: `POST /api/transcript`

#### Request Headers:
```http
Content-Type: application/json
```

#### Request Body Schema:
```json
{
  "videoId": "dQw4w9WgXcQ",
  "language": "en"
}
```
* `videoId` *(string, required)*: 11 karakter alfanumerik YouTube ID (`^[a-zA-Z0-9_-]{11}$`).
* `language` *(string, optional)*: Kode bahasa ISO (misal: `"en"`, `"id"`, `"es"`).

#### Success Response (`200 OK`):
```json
{
  "success": true,
  "data": {
    "videoId": "dQw4w9WgXcQ",
    "language": "en",
    "isGenerated": false,
    "segments": [
      {
        "text": "Never gonna give you up",
        "offset": 18.5,
        "duration": 2.4
      }
    ],
    "transcript": "Never gonna give you up ..."
  }
}
```

#### Error Response (`400 Bad Request` / `404 Not Found` / `429 Too Many Requests` / `500 Internal Error`):
```json
{
  "success": false,
  "error": {
    "code": "TRANSCRIPT_NOT_AVAILABLE",
    "message": "Transcript is not available for this video"
  }
}
```

---

## 8. Data Models (TypeScript)

File: `src/features/transcript/domain/transcript.types.ts`

```typescript
export interface TranscriptSegment {
  text: string;
  offset: number;     // Detik (desimal, e.g. 1.25)
  duration: number;   // Detik (desimal, e.g. 2.50)
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

export interface TranscriptResponseSuccess {
  success: true;
  data: Transcript;
}

export interface TranscriptResponseError {
  success: false;
  error: {
    code: TranscriptErrorCode;
    message: string;
  };
}

export type TranscriptApiResponse = TranscriptResponseSuccess | TranscriptResponseError;
```

---

## 9. Error Handling & Mapping

Tabel pemetaan error internal library ke kode error aplikasi:

| Error Internal / Skenario | Error Code HTTP | User Facing Message |
| :--- | :--- | :--- |
| Format URL tidak dikenali | `INVALID_URL` (400) | "Please enter a valid YouTube URL." |
| Video ID tidak valid / bukan 11 char | `INVALID_VIDEO_ID` (400) | "Invalid YouTube video ID." |
| `YoutubeTranscriptDisabledError` / tidak ada caption | `TRANSCRIPT_NOT_AVAILABLE` (404) | "Transcript is not available for this video." |
| Video private / deleted / region blocked | `VIDEO_UNAVAILABLE` (404) | "This video is unavailable or restricted." |
| Bahasa yang diminta tidak tersedia | `LANGUAGE_NOT_AVAILABLE` (404) | "Selected language is unavailable for this video." |
| Request YouTube melebihi 10 detik | `REQUEST_TIMEOUT` (504) | "Request timed out while fetching transcript. Please try again." |
| Client mengirim lebih dari 10 req/menit | `RATE_LIMITED` (429) | "Too many requests. Please slow down and try again later." |
| Error jaringan / unhandled exception | `PROVIDER_ERROR` (500) | "Unable to retrieve transcript at this moment." |

---

## 10. Security Strategy

1. **SSRF Prevention:** Backend tidak pernah melakukan request ke URL arbitrer yang dikirimkan user. Backend hanya menerima `videoId` terverifikasi (11 karakter regex `^[a-zA-Z0-9_-]{11}$`). URL YouTube dibangun secara deterministik di server.
2. **Rate Limiting:** Implementasi sliding window rate limiter di Route Handler (maksimal 15 request per IP per menit) untuk mencegah abuse.
3. **No Secret Exposure:** Tidak ada credential atau private key yang terekspos ke klien.
4. **Sanitasi Error:** Server tidak membocorkan call stack trace atau internal scraping logs kepada pengguna.
5. **No Data Retention:** Tidak ada database yang mencatat histori pencarian atau konten video pengguna.

---

## 11. Dependencies List

### Production Dependencies:
* `next`: Framework inti (16.4.4)
* `react` / `react-dom`: UI runtime (19.3.0)
* `lucide-react`: Ikon visual
* `clsx` & `tailwind-merge`: Class name merging utility
* `class-variance-authority`: UI variant handling
* `youtube-transcript`: Terbukti bekerja pada Node.js untuk ekstraksi timedtext YouTube.

### Dev Dependencies:
* `typescript`, `@types/node`, `@types/react`, `@types/react-dom`
* `tailwindcss` (^4), `@tailwindcss/turbopack`
* `eslint`, `eslint-config-next`

---

## 12. Deployment Considerations

* Proyek siap di-deploy langsung ke **Vercel** atau **Docker/Node.js VPS**.
* Tidak ada dependensi binary eksternal seperti Python, FFmpeg, atau Rust.
* Serverless runtime (Node.js runtime di Route Handler) sepenuhnya kompatibel dengan `youtube-transcript` karena hanya menggunakan HTTP requests tanpa puppeteer/headless browser.

---

## 13. Technical Risks & Mitigations

| Risiko Teknis | Tingkat Keparahan | Strategi Mitigasi |
| :--- | :--- | :--- |
| **YouTube Bot Detection / IP Block di Cloud Provider** | High | Menggunakan Header User-Agent browser modern, caching respon identik sementara di memory, dan isolasi adapter agar dapat dengan mudah disambungkan ke proxy atau fallback provider. |
| **Video tanpa Caption / Subtitle** | Medium | Error handling yang bersahabat: memberikan penjelasan jelas kepada user bahwa aplikasi mengambil subtitle yang disediakan pembuat video / auto-caption YouTube. |
| **Perubahan Format Player YouTube** | Medium | Seluruh logika parsing dibatasi di dalam `YoutubeTranscriptAdapter`. Jika mekanisme berubah, hanya 1 file adapter yang perlu disesuaikan tanpa memengaruhi UI. |
| **Transcript Sangat Panjang (e.g. Podcast 3 jam)** | Low | Komponen UI menggunakan container scrollable dengan fixed height & layout virtualizer jika diperlukan, serta tombol Copy yang memproses string langsung di web worker / background task. |

---

## 14. Implementation Roadmap (Phases 3 - 10)

* **Phase 3:** UI/UX Planning & Design System (`UI_UX_GUIDE.md`).
* **Phase 4:** Frontend Foundation & Application Shell (Layout, Navbar, Footer, Reusable Container).
* **Phase 5:** Core Frontend Feature Development (URL Input, Result Area, Skeleton Loading, Copy Button).
* **Phase 6:** Backend & Core Business Logic (Route Handler `/api/transcript`, Adapter, Service, Rate Limiter).
* **Phase 7:** Frontend & Backend Integration (Hubungkan UI dengan API nyata, E2E flow testing).
* **Phase 8:** Additional Features & UX Enhancements (Timestamp toggling, Segment copying, Download .txt).
* **Phase 9:** Testing, Security Audit & Performance Tuning (Unit tests, Edge cases, Mobile responsive check).
* **Phase 10:** Production Deployment & Release Check (Vercel deployment, HTTPS, final verification).

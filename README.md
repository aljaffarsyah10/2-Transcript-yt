# transscipt-yt — YouTube Transcript & Subtitle Converter

[![Next.js](https://img.shields.io/badge/Next.js-16.4.4-black?style=flat&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38bdf8?style=flat&logo=tailwindcss)](https://tailwindcss.com/)
[![Tests](https://img.shields.io/badge/Tests-21%20Passed-emerald?style=flat)](./tests/)
[![Architecture](https://img.shields.io/badge/Architecture-Modular_Monolith-indigo?style=flat)]()
[![Database](https://img.shields.io/badge/Database-Zero_DB_(Stateless)-orange?style=flat)]()
[![License](https://img.shields.io/badge/License-MIT-green?style=flat)]()

> **Fast, free, and privacy-friendly web application to extract, read, copy, and download clean transcripts and closed captions from YouTube videos.**

---

## 📌 Overview

**transscipt-yt** enables users to convert any publicly available YouTube video into clean, readable text instantly by simply pasting the video link.

* **Zero Sign-Up Required:** No account, no authentication, no friction.
* **Privacy-Friendly & Stateless:** Transcripts are never stored permanently in a database.
* **Instant Extraction:** Fetches YouTube's accessible timedtext captions in milliseconds without downloading bulky audio/video files or running heavy speech-to-text models.

---

## ✨ Features

- [x] **Universal YouTube URL Support:**
  - Standard links (`https://www.youtube.com/watch?v=VIDEO_ID`)
  - Shortened links (`https://youtu.be/VIDEO_ID`)
  - YouTube Shorts (`https://www.youtube.com/shorts/VIDEO_ID`)
  - Embed links (`https://www.youtube.com/embed/VIDEO_ID`)
  - Mobile links (`https://m.youtube.com/watch?v=VIDEO_ID`)
  - Raw 11-character Video IDs (`dQw4w9WgXcQ`)
  - Handles extra tracking and timestamp parameters (`&t=...`, `&list=...`, `?si=...`) seamlessly.
- [x] **Dual Reading Modes:**
  - **Continuous Text View:** Flowing narrative paragraphs optimized for readability (*line-height 1.75*).
  - **Timestamped View:** Line-by-line segments with clickable `[MM:SS]` (or `[HH:MM:SS]` for long podcasts).
- [x] **One-Click Clipboard Actions:**
  - **Copy All:** Copies the full transcript to clipboard with visual confirmation feedback.
  - **Copy Segment:** Individual copy button per sentence in timestamp mode.
- [x] **Local File Export:** Download full transcripts as clean `.txt` files directly in your browser.
- [x] **Language Selector:** Switch and re-fetch captions in available languages (English, Indonesian, Spanish, French, German, etc.).
- [x] **Video Preview Card:** Displays official video thumbnail and direct link to watch on YouTube.
- [x] **Security Hardened:**
  - Strict regex input sanitization (blocks SSRF and malicious URLs).
  - Built-in In-Memory Rate Limiting (15 requests/min per IP).
  - Clean error masking (no server stack traces leaked to clients).
- [x] **Fully Responsive:** Seamless layout across mobile (375px+), tablet, and widescreen desktop.

---

## 🛠️ Tech Stack

| Layer | Technology | Description |
| :--- | :--- | :--- |
| **Framework** | Next.js 16.4.4 (App Router) | React Server Components, Turbopack, and Dynamic Route Handlers |
| **Language** | TypeScript 5.x | End-to-end type safety |
| **Styling** | Tailwind CSS v4 | Modern, utility-first CSS engine |
| **UI Primitives** | shadcn/ui & Radix UI | Accessible, headless UI component architecture |
| **Icons** | Lucide React | Lightweight SVG icons |
| **Testing** | Node.js 22 Test Runner + `tsx` | Native zero-bloat automated test runner |
| **HTTP Client** | Native `fetch` | Zero external HTTP dependencies |

---

## 🏗️ Architecture

The project follows a **Modular Monolith** architecture with **Hexagonal / Ports & Adapters Pattern**:

```
src/
├── app/
│   ├── api/
│   │   └── transcript/
│   │       └── route.ts            # Dynamic POST /api/transcript handler
│   ├── layout.tsx                  # Root layout, fonts, and OpenGraph SEO
│   ├── page.tsx                    # Main interactive single-page app
│   ├── robots.ts                   # Search crawler directives
│   └── sitemap.ts                  # Dynamic sitemap index
│
├── components/
│   ├── layout/
│   │   ├── navbar.tsx              # Header with logo & GitHub link
│   │   └── footer.tsx              # Disclaimer & privacy notices
│   └── ui/                         # shadcn/ui primitives (button, card, input, etc.)
│
├── features/
│   └── transcript/
│       ├── application/
│       │   └── get-transcript.service.ts # Use-case orchestration service
│       ├── domain/
│       │   ├── transcript.types.ts       # Domain entities & API DTOs
│       │   ├── transcript.errors.ts      # Structured error hierarchy
│       │   └── transcript-provider.interface.ts # Port definition
│       ├── infrastructure/
│       │   └── youtube-transcript.adapter.ts # Adapter wrapping scraping library
│       ├── components/
│       │   ├── hero-section.tsx          # Display heading & format chips
│       │   ├── transcript-form.tsx       # URL input with inline validation
│       │   ├── transcript-result.tsx     # Scrollable transcript container
│       │   ├── transcript-toolbar.tsx    # Copy, Download .txt, and Reset buttons
│       │   ├── loading-state.tsx         # Shimmer skeleton loading indicator
│       │   └── error-state.tsx           # Error alert with Retry button
│       ├── hooks/
│       │   └── use-transcript.ts         # Reactive transcript custom hook
│       └── utils/
│           ├── youtube-url.ts            # Strict URL regex parser & extractor
│           └── format-timestamp.ts       # Adaptive [MM:SS] / [HH:MM:SS] formatter
│
├── lib/
│   ├── utils.ts                          # cn() class merge helper
│   └── rate-limiter.ts                   # In-memory sliding window rate limiter
│
└── tests/                                # 21 automated unit & integration tests
```

---

## 🚀 Getting Started

### Prerequisites
* **Node.js:** v20.x or v22.x LTS
* **npm:** v10.x or later
* **Git**

### Installation

1. **Clone repository:**
   ```bash
   git clone https://github.com/<username>/transscipt-yt.git
   cd transscipt-yt/transscipt-yt
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Start local development server:**
   ```bash
   npm run dev
   ```
   Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🧪 Testing & Code Quality

The project includes an automated test suite covering URL extraction, boundary conditions, SSRF prevention, timestamp formatting, rate limiting, and service integration.

```bash
# Run all automated tests (21 tests in 4 suites)
npm test

# Run TypeScript type check
npx tsc --noEmit

# Run ESLint check
npm run lint

# Build production bundle
npm run build

# Start production server
npm start
```

---

## 🔌 API Specification

### `POST /api/transcript`

Extracts subtitles and transcripts for a given YouTube video.

#### Request Headers:
```http
Content-Type: application/json
```

#### Request Body:
```json
{
  "videoId": "dQw4w9WgXcQ",
  "language": "en"
}
```
* `videoId` *(string, required)*: Exactly 11 alphanumeric characters (`^[a-zA-Z0-9_-]{11}$`).
* `language` *(string, optional)*: ISO language code (e.g. `"en"`, `"id"`, `"es"`).

#### Success Response (`200 OK`):
```json
{
  "success": true,
  "data": {
    "videoId": "dQw4w9WgXcQ",
    "language": "en",
    "isGenerated": null,
    "segments": [
      {
        "text": "Never gonna give you up",
        "offset": 43.0,
        "duration": 2.12
      }
    ],
    "transcript": "Never gonna give you up ..."
  }
}
```

#### Error Response:
```json
{
  "success": false,
  "error": {
    "code": "TRANSCRIPT_NOT_AVAILABLE",
    "message": "Transcript is not available for this video."
  }
}
```

#### Standard Error Codes:
| Error Code | HTTP Status | Description |
| :--- | :---: | :--- |
| `INVALID_VIDEO_ID` | 400 | Provided video ID is missing or invalid. |
| `LANGUAGE_NOT_AVAILABLE`| 400 | Selected language code is malformed or unavailable. |
| `TRANSCRIPT_NOT_AVAILABLE`| 404 | Video has no captions or owner disabled them. |
| `VIDEO_UNAVAILABLE` | 404 | Video is private, deleted, or geographically restricted. |
| `RATE_LIMITED` | 429 | Client exceeded 15 requests per minute limit. |
| `REQUEST_TIMEOUT` | 504 | Upstream request exceeded 10-second timeout threshold. |
| `PROVIDER_ERROR` | 500 | Unexpected error communicating with upstream provider. |

---

## 🌐 Deployment

Detailed deployment guides are available in [`DEPLOYMENT_GUIDE.md`](./DEPLOYMENT_GUIDE.md):

* **Vercel (Recommended):** Zero-configuration serverless deployment with automatic preview URLs and SSL.
  ```bash
  npx vercel
  ```
* **Linux VPS (PM2 + Nginx):** Full control over server IP, Node.js runtime, and Let's Encrypt SSL.
* **Docker Container:** Production-ready multi-stage Docker build.

---

## 📖 Project Documentation

Detailed design documents generated during the sequential development phases:

* [PRD & Playbook](./PROJECT_PRD_AND_PLAYBOOK.md) — Comprehensive product requirements & 10-phase execution playbook.
* [Technical Plan](./TECHNICAL_PLAN.md) — Technical architecture, provider evaluation, data models, and security plan.
* [UI/UX Guide](./UI_UX_GUIDE.md) — Design system, wireframes, typography tokens, and responsive matrix.
* [Test Report](./TEST_REPORT.md) — Full automated test results, security audit, and performance benchmarks.
* [Deployment Guide](./DEPLOYMENT_GUIDE.md) — Production setup for Vercel, VPS, and Docker.
* [Release Notes](./RELEASE_NOTES.md) — Release notes for Version 1.0.0 (MVP).

---

## ⚖️ License & Disclaimer

* **License:** [MIT License](./LICENSE)
* **Disclaimer:** `transscipt-yt` extracts publicly available closed captions provided by YouTube and creators. This project is not affiliated with, endorsed, or sponsored by YouTube, Google, or Alphabet Inc. All trademarks belong to their respective owners.

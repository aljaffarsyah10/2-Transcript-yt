# DEPLOYMENT GUIDE — transscipt-yt

**Document Version:** 1.0.0  
**Phase:** Phase 10 — Production Deployment & Release  
**Project:** `transscipt-yt` (YouTube Transcript Converter)  
**Status:** Approved for Production Deployment

---

## 1. Overview & Architecture Considerations

`transscipt-yt` dirancang sebagai **Stateless Modular Monolith** berbasis **Next.js App Router**:
* **Frontend:** React Server Components + Client Components yang teroptimasi secara statis (*Static Prerender*).
* **Backend:** Next.js Route Handler (`POST /api/transcript`) dinamis dengan rate limiting in-memory.
* **Database:** **Zero Database required** (tidak menyimpan data pengguna, histori, atau credential).
* **External Scraper:** `youtube-transcript` berjalan murni melalui HTTP request ke YouTube timedtext API tanpa dependensi binary berat (tanpa FFmpeg, tanpa Chromium/Puppeteer).

---

## 2. Option A: Deployment to Vercel (Recommended)

Platform paling optimal dan minim pemeliharaan untuk Next.js.

### Langkah 1: Push Repository ke GitHub
1. Pastikan seluruh commit telah berada di branch `main`:
   ```bash
   git status
   git remote add origin https://github.com/<username>/transscipt-yt.git
   git push -u origin main
   ```

### Langkah 2: Import Proyek di Vercel Dashboard
1. Buka [https://vercel.com/new](https://vercel.com/new) dan login dengan akun GitHub Anda.
2. Pilih repository `transscipt-yt`.
3. Jika repository memuat subdirektori (misal `transscipt-yt`), atur **Root Directory** ke `transscipt-yt`.
4. Framework Preset akan otomatis terdeteksi sebagai **Next.js**.
5. Build and Output Settings:
   * **Build Command:** `npm run build`
   * **Output Directory:** `.next`
   * **Install Command:** `npm install`

### Langkah 3: Konfigurasi Environment Variables
Tambahkan variabel berikut pada panel **Environment Variables**:
* `NODE_ENV`: `production`
* `NEXT_PUBLIC_APP_URL`: `https://your-custom-domain.com` (atau URL bawaan `.vercel.app`)

### Langkah 4: Deploy & Verifikasi
1. Klik tombol **Deploy**.
2. Vercel akan otomatis mengompilasi dan menyediakan URL produksi publik dengan sertifikat **HTTPS / SSL aktif**.

---

## 3. Option B: Deployment to Linux VPS (Ubuntu / Debian + PM2 + Nginx)

Cocok jika Anda menginginkan kontrol penuh atas IP server, concurrency, atau ingin menghindari batasan serverless shared IP.

### Prasyarat Server:
* Ubuntu 22.04 LTS atau 24.04 LTS
* Node.js v20.x atau v22.x LTS & npm
* Nginx Web Server & Certbot (Let's Encrypt)

### Langkah 1: Setup Node.js & Repository di Server
```bash
# Update sistem
sudo apt update && sudo apt upgrade -y

# Install Node.js 22 LTS via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs git nginx

# Install PM2 Process Manager secara global
sudo npm install -g pm2

# Clone repository
git clone https://github.com/<username>/transscipt-yt.git /var/www/transscipt-yt
cd /var/www/transscipt-yt/transscipt-yt

# Install dependensi dan build aplikasi
npm ci --omit=dev
npm run build
```

### Langkah 2: Jalankan Aplikasi Menggunakan PM2
```bash
# Jalankan Next.js produksi pada port 3000
pm2 start npm --name "transscipt-yt" -- start

# Simpan konfigurasi PM2 agar otomatis berjalan saat server restart
pm2 save
pm2 startup
```

### Langkah 3: Konfigurasi Nginx Reverse Proxy
Buat file konfigurasi Nginx:
```bash
sudo nano /etc/nginx/sites-available/transscipt-yt
```

Isi dengan konfigurasi berikut:
```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 60s;
        proxy_connect_timeout 60s;
    }
}
```

Aktifkan konfigurasi dan restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/transscipt-yt /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Langkah 4: Pasang SSL / HTTPS Gratis dengan Certbot
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

---

## 4. Option C: Docker Container Deployment

Jika menggunakan container platform seperti Railway, Fly.io, Google Cloud Run, atau AWS ECS:

### `Dockerfile`
```dockerfile
FROM node:22-alpine AS base

# 1. Install dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# 2. Rebuild the source code
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# 3. Production runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

USER nextjs
EXPOSE 3000
CMD ["npm", "start"]
```

---

## 5. Domain & DNS Configuration

Arahkan domain kustom Anda ke penyedia hosting:

| Host / Tipe | Nama Record | Target Nilai | Keterangan |
| :--- | :--- | :--- | :--- |
| **A Record** | `@` | IP VPS Anda (atau `76.76.21.21` untuk Vercel) | Root domain |
| **CNAME** | `www` | `cname.vercel-dns.com` (atau domain Anda) | Subdomain www |

---

## 6. Monitoring & Logging

1. **Health Check Endpoint:**
   * Halaman beranda `GET /` atau request status ke `GET /robots.txt` dapat digunakan oleh uptime monitor (misal UptimeRobot, BetterStack) untuk memverifikasi ketersediaan aplikasi setiap menit.
2. **Server Logs:**
   * Pada Vercel: Periksa menu **Logs & Monitoring** di Dashboard Vercel.
   * Pada VPS: Jalankan `pm2 logs transscipt-yt --lines 100`.
3. **Peringatan Kuota & Rate Limiting:**
   * Respons `429 Too Many Requests` menunjukkan rate limiter berfungsi melindungi server dari bot scraping.

---

## 7. Production Post-Deployment Smoke Test Checklist

Setelah aplikasi aktif di internet, jalankan pengujian berikut:

- [ ] **Akses HTTPS:** Buka `https://<domain-anda>` di browser, pastikan ikon gembok aman (SSL valid).
- [ ] **Pengujian Video Populer:** Masukkan `https://www.youtube.com/watch?v=dQw4w9WgXcQ` & klik **Get Transcript**. Pastikan lirik muncul < 1.5 detik.
- [ ] **Pengujian Salin Teks:** Klik tombol **Copy All**, paste ke text editor, pastikan seluruh transcript tersalin.
- [ ] **Pengujian Unduh Berkas:** Klik **Download .txt**, pastikan berkas teks terunduh ke komputer/HP.
- [ ] **Pengujian Video Tanpa Subtitle:** Masukkan video tanpa subtitle; pastikan muncul alert merah bersahabat `Transcript Not Available` tanpa crash.
- [ ] **Pengujian Mobile:** Buka website via smartphone, pastikan layout responsif dan tidak ada horizontal scrolling.

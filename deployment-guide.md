# AYA — Deployment Guide

## Architecture Overview

```
ayamatch.com (Vercel)          api.ayamatch.com (Railway/Render)
     ┌──────────┐                    ┌──────────────┐
     │ Next.js  │──── HTTPS/WSS ────▶│   NestJS     │
     │   PWA    │                    │   Backend    │
     └──────────┘                    └──────┬───────┘
                                           │
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                         PostgreSQL     Redis      Cloudinary
                         (Supabase)   (Upstash)   (Media CDN)
```

---

## 1. Prerequisites

- Node.js 20+ installed
- Git configured
- Accounts on: Vercel, Railway (or Render), Supabase, Upstash, Cloudinary
- Domain: `ayamatch.com` (or your domain)

---

## 2. Backend Deployment (Railway or Render)

### Option A: Railway

1. Connect your GitHub repo (`christian-be`) to Railway
2. Set the build command: `npm run build`
3. Set the start command: `npm run start:prod`
4. Add all environment variables (see section 4 below)
5. After deploy, run the migration:
   ```bash
   npx prisma migrate deploy
   ```
6. Seed the database (optional, for staging):
   ```bash
   npm run db:seed
   ```

### Option B: Render

1. Create a new Web Service from `christian-be` repo
2. Runtime: Node
3. Build command: `npm install && npx prisma generate && npm run build`
4. Start command: `npm run start:prod`
5. Add environment variables
6. Set health check path: `/api/v1`

### Post-Deploy

- Verify the API is reachable: `https://api.ayamatch.com/api/v1`
- Verify Swagger docs: `https://api.ayamatch.com/docs` (should be disabled in production)
- Test a health endpoint or login flow

---

## 3. Frontend Deployment (Vercel)

1. Connect your GitHub repo (`christian-fe`) to Vercel
2. Framework preset: Next.js (auto-detected)
3. Build command: `npm run build` (default)
4. Output directory: `.next` (default)
5. Add environment variables:

```
NEXT_PUBLIC_API_URL=https://api.ayamatch.com/api/v1
NEXT_PUBLIC_SOCKET_URL=https://api.ayamatch.com
```

6. Deploy
7. Set custom domain: `ayamatch.com` and `www.ayamatch.com`

### Vercel Settings

- Node.js version: 20.x
- Install command: `npm install`
- Build optimizations: enabled (default)

---

## 4. Environment Variables

### Backend (.env)

```bash
# App
NODE_ENV=production
PORT=5001

# Database (Supabase)
DATABASE_URL=postgresql://user:password@host:5432/postgres?pgbouncer=true
DIRECT_URL=postgresql://user:password@host:5432/postgres

# JWT
JWT_SECRET=<generate-a-64-char-random-string>
JWT_REFRESH_SECRET=<generate-another-64-char-random-string>
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# Frontend URL (for CORS and OAuth redirect)
FRONTEND_URL=https://ayamatch.com
ALLOWED_ORIGINS=https://ayamatch.com,https://www.ayamatch.com

# Google OAuth
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>
GOOGLE_CALLBACK_URL=https://api.ayamatch.com/api/v1/auth/google/callback

# Email (SMTP)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=<your-email>
SMTP_PASS=<your-app-password>
SMTP_FROM=noreply@ayamatch.com

# Cloudinary
CLOUDINARY_CLOUD_NAME=<your-cloud-name>
CLOUDINARY_API_KEY=<your-api-key>
CLOUDINARY_API_SECRET=<your-api-secret>

# Redis (Upstash)
REDIS_URL=rediss://<user>:<password>@<host>:<port>

# Web Push (VAPID)
VAPID_PUBLIC_KEY=<your-vapid-public-key>
VAPID_PRIVATE_KEY=<your-vapid-private-key>
VAPID_EMAIL=mailto:hello@ayamatch.com
```

### Frontend (.env.local → Vercel env vars)

```bash
NEXT_PUBLIC_API_URL=https://api.ayamatch.com/api/v1
NEXT_PUBLIC_SOCKET_URL=https://api.ayamatch.com
```

### Generating Secrets

```bash
# JWT secrets
openssl rand -base64 48

# VAPID keys
npx web-push generate-vapid-keys
```

---

## 5. Database Setup (Supabase)

1. Create a new Supabase project
2. Copy the connection strings:
   - `DATABASE_URL` — use the **pooled** connection (port 6543, pgbouncer)
   - `DIRECT_URL` — use the **direct** connection (port 5432)
3. Run migrations:
   ```bash
   npx prisma migrate deploy
   ```
4. (Optional) Seed data:
   ```bash
   npm run db:seed
   ```

### Important Supabase Settings

- Enable Row Level Security (RLS) if using Supabase client directly (not needed for Prisma)
- Set connection pool size to match your backend instances

---

## 6. Redis Setup (Upstash)

1. Create an Upstash Redis database
2. Choose the region closest to your backend
3. Copy the `REDIS_URL` (use the TLS URL: `rediss://...`)
4. Used for: rate limiting (ThrottlerModule), Bull job queues

---

## 7. Cloudinary Setup

1. Create a Cloudinary account
2. Copy: Cloud Name, API Key, API Secret
3. Create upload presets:
   - `aya_profile_photos` — auto crop, face detection, max 2MB
   - `aya_gallery` — auto format, max 5MB
   - `aya_videos` — max 15 seconds, max 20MB

---

## 8. Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create OAuth 2.0 credentials
3. Authorized redirect URI: `https://api.ayamatch.com/api/v1/auth/google/callback`
4. Authorized JavaScript origin: `https://ayamatch.com`
5. Copy Client ID and Client Secret to backend env vars

---

## 9. Domain & SSL

### Vercel (Frontend)

1. Add `ayamatch.com` as custom domain in Vercel dashboard
2. Point DNS:
   - `A` record → Vercel IP (shown in dashboard)
   - `CNAME` for `www` → `cname.vercel-dns.com`
3. SSL is automatic

### Railway/Render (Backend)

1. Add `api.ayamatch.com` as custom domain
2. Point DNS:
   - `CNAME` → your Railway/Render subdomain
3. SSL is automatic
4. Update `FRONTEND_URL` and `ALLOWED_ORIGINS` in env vars

---

## 10. Post-Deployment Checklist

### Verify Backend

- [ ] API responds at `https://api.ayamatch.com/api/v1`
- [ ] Swagger docs are disabled (or protected) in production
- [ ] Database is migrated and accessible
- [ ] Redis is connected (rate limiting works)
- [ ] CORS allows only `ayamatch.com`
- [ ] WebSocket connects from frontend
- [ ] Email sending works (test signup → verification email)
- [ ] Cloudinary uploads work
- [ ] Google OAuth flow completes

### Verify Frontend

- [ ] Site loads at `https://ayamatch.com`
- [ ] Landing page renders correctly with Aya branding
- [ ] Signup → email verification → login flow works
- [ ] Discover page shows profiles
- [ ] Swiping works (API calls succeed)
- [ ] Chat messages send and receive in real-time
- [ ] Push notifications prompt appears after login
- [ ] Push notifications received when app is in background
- [ ] PWA installable (Add to Home Screen works)
- [ ] Manifest shows correct name and theme color

### Security

- [ ] HTTPS enforced (no HTTP access)
- [ ] JWT secrets are unique and not committed to git
- [ ] `.env` is in `.gitignore`
- [ ] Rate limiting is active on auth endpoints
- [ ] CORS rejects unknown origins
- [ ] WebSocket only accepts known origins

---

## 11. Monitoring (Optional)

### Sentry (Error Tracking)

1. Create a Sentry project
2. Add `SENTRY_DSN` to backend env vars
3. Install `@sentry/node` and configure in `main.ts`

### Uptime Monitoring

- Use UptimeRobot or Better Uptime
- Monitor: `https://api.ayamatch.com/api/v1` (backend health)
- Monitor: `https://ayamatch.com` (frontend)

---

## 12. Scaling Notes

### When to Scale

| Signal | Action |
|--------|--------|
| API response > 500ms | Add backend instances |
| DB connections maxed | Increase Supabase pool size |
| Redis memory > 80% | Upgrade Upstash plan |
| 10K+ concurrent WebSocket | Add Redis adapter for Socket.io |
| Image uploads slow | Enable Cloudinary eager transforms |

### Multi-Instance WebSocket

If running multiple backend instances, Socket.io needs a Redis adapter:

```bash
npm install @socket.io/redis-adapter
```

Configure in the chat gateway to share events across instances.

# AYA — Church Boys & Girls Dating App

## Overview

AYA is a faith-based dating platform designed for Christian singles. It features a PWA frontend, a NestJS backend, and a React Native mobile app — all sharing the same API and real-time infrastructure.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                             │
│  Next.js 15 PWA (Vercel) — Progressive Web App              │
│  React Native / Expo (iOS + Android)                        │
│  - Tailwind CSS + Radix UI (web)                            │
│  - Socket.io client for real-time                           │
│  - Zustand + React Query for state                          │
└─────────────────────────────────────────────────────────────┘
                           ↓ HTTPS / WSS
┌─────────────────────────────────────────────────────────────┐
│                     API LAYER                                │
│  NestJS Backend (Railway/Render)                            │
│  - RESTful API endpoints                                     │
│  - WebSocket Gateway (Socket.io)                            │
│  - JWT Authentication (Passport)                             │
│  - Bull Queue for background jobs                           │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                   DATA LAYER                                 │
│  PostgreSQL (Neon/Supabase) — Primary Database              │
│  Redis (Upstash) — Caching + Job Queue                      │
│  Cloudinary — Media Storage (images/videos)                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Unique / Faith-Focused Features

- **Denomination Matching:** 11 Christian denominations as filter criteria
- **Talking Stages:** Temple (1-3 weeks) → Courtship (1-3 months) → Committed (3+ months)
- **Sunday Check-ins:** Track church attendance with GPS location + photo
- **Celibacy Tracker:** Track abstinence streaks with daily check-ins
- **Mission/Volunteer Uploads:** Log mission work with photos
- **Achievements & Badges:** Gamified milestones for faith engagement
- **Daily Devotionals:** Bible verse + question of the day (14 rotating)
- **Counseling Integration:** Built-in couples counselor request system
- **Content Moderation:** Faith-specific trolling detection + PII filtering
- **Lifestyle Tags:** Sober, waiting until marriage, family-oriented, etc.

---

## Completion Status

| Component | Status | Notes |
|-----------|--------|-------|
| **Backend** | ~75-80% | Core features complete. Missing: tests, push notifications, AI moderation, admin dashboard |
| **Frontend (PWA)** | ~85-90% | Most pages built. Missing: settings pages, video upload UI, devotional UI, counseling UI |
| **Mobile** | ~60-70% | Core flows work. Missing: form completions, settings stubs, push notifications, image uploads |

---

## Shared Gaps Across All Platforms

- **No tests** written for any layer (Jest configured in backend & frontend)
- **Push notifications** scaffolded (OneSignal) but not implemented
- **Talking stage management** UI not built (backend supports it)
- **Devotional/icebreaker UI** not built on frontend or mobile
- **Counseling UI** not built on frontend or mobile
- **Admin/moderation dashboard** not built
- **Environment configuration** for staging/production not set up (hardcoded dev URLs in mobile)
- **Analytics & error tracking** not integrated (Sentry DSN exists in config)

---

## Directory Structure

```
aya/
├── backend/      # NestJS API server
├── frontend/     # Next.js 15 PWA
├── mobile/       # React Native / Expo app
├── docs/         # Project documentation (this folder)
└── project-prompt.md   # Original technical specification
```

---

## Detailed Documentation

- [Backend Documentation](backend.md)
- [Frontend Documentation](frontend.md)
- [Mobile Documentation](mobile.md)

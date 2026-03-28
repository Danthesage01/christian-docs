# Backend (NestJS) — ~75-80% Complete

## Tech Stack

- **Framework:** NestJS 10.4.0 (TypeScript 5.5, Node.js 20+)
- **Database & ORM:** PostgreSQL (Neon/Supabase) + Prisma 5.18.0
- **Auth:** JWT (access + refresh tokens), Passport.js (Local, Google OAuth 2.0), Bcrypt
- **Real-time:** Socket.io 4.7.5 via NestJS WebSocket Gateway
- **File Storage:** Cloudinary 1.41.0 + Multer
- **Email:** Nodemailer 7.0.12 / SendGrid
- **Caching & Queues:** Redis (ioredis 5.4.1) + Bull 4.15.0
- **Security:** Helmet, CORS, Rate Limiting (ThrottlerModule)
- **API Docs:** Swagger/OpenAPI (available at `/docs` in non-production)
- **Testing:** Jest (configured, no tests written yet)

---

## Project Structure

```
backend/
├── src/
│   ├── main.ts                    # Bootstrap
│   ├── app.module.ts              # Root module (11 feature modules)
│   ├── app.factory.ts             # Middleware, CORS, Swagger config
│   ├── app.controller.ts / app.service.ts
│   ├── config/                    # App, database, JWT, Cloudinary, Redis configs
│   ├── database/                  # Prisma service & module
│   ├── common/
│   │   ├── decorators/            # CurrentUser, Public, Roles
│   │   ├── guards/                # JwtAuthGuard, WebSocket JWT guard
│   │   ├── filters/               # HTTP/WebSocket exception handlers
│   │   ├── interceptors/          # Transform, Logging
│   │   ├── interfaces/            # JwtPayload, ApiResponse
│   │   └── constants/             # Error messages, app constants
│   ├── modules/
│   │   ├── auth/                  # Signup, login, OAuth, email verification, password reset
│   │   ├── users/                 # Profile CRUD, preferences, deletion
│   │   ├── matching/              # Swiping, match discovery
│   │   ├── chat/                  # Conversations, messages, WebSocket gateway
│   │   ├── search/                # User search, prayer partners, church partners
│   │   ├── milestones/            # Sunday check-in, celibacy, missions, achievements
│   │   ├── counseling/            # Session management
│   │   ├── moderation/            # Content moderation, reporting
│   │   ├── notifications/         # Notification management
│   │   ├── devotional/            # Daily questions
│   │   ├── upload/                # Cloudinary file uploads
│   │   └── email/                 # SMTP email service
│   └── utils/                     # Distance calculator, date utilities, string utilities
├── prisma/                        # schema.prisma, migrations, seed script
├── package.json
├── tsconfig.json
├── nest-cli.json
└── .env.example
```

---

## Implemented Features

### Authentication Module (`/src/modules/auth/`)
- Signup with email verification (6-digit code, 15-minute expiry)
- Login with email/password
- Google OAuth 2.0 integration
- Email verification with resend capability
- Password reset flow with token-based recovery
- JWT token management (access + refresh tokens)
- Logout with token invalidation
- Profile completion checks for OAuth users

### Users Module (`/src/modules/users/`)
- User profile management (get, update, delete)
- Public profile endpoint for matching
- Match preferences updates
- Account deletion (soft delete via `deletedAt`)
- Profile completeness tracking

### Matching Module (`/src/modules/matching/`)
- Swipe functionality (LIKE, PASS, SUPER_LIKE actions)
- Get potential matches with:
  - Age range filtering
  - Gender preference
  - Denomination preference
  - Distance-based filtering (maxDistanceKm)
  - Exclude already-swiped profiles
  - Only show active, complete profiles
- View all mutual matches
- View likes received
- Pagination support

### Search Module (`/src/modules/search/`)
- Advanced user search with filters: denomination(s), gender, age range
- "Searching for" category (partner, prayer partner, counselor, church partner)
- City/state location filters
- Prayer partner finder
- Church partner finder by city
- Pagination

### Chat Module (`/src/modules/chat/`)
- Real-time messaging via WebSocket (Socket.io)
- Conversation management
- Message history retrieval with pagination
- Read status tracking
- Typing indicators
- WebSocket authentication (JWT-based)
- Connection/disconnection handling
- Message notifications emit to recipients

### Milestones Module (`/src/modules/milestones/`)
- **Sunday Check-ins:** Track church attendance with GPS location + photo
- **Celibacy Tracker:** Track celibacy streaks (daily check-ins, reset capability)
- **Mission/Volunteer Uploads:** Upload mission work with photos
- **Achievements:** Unlock badges for Sunday streaks, celibacy milestones, profile completion, first match, mission uploads
- Statistics dashboards for each milestone type

### Counseling Module (`/src/modules/counseling/`)
- Request counselor sessions for conversations
- Session management (PENDING, ACTIVE, COMPLETED, CANCELLED)
- Schedule counseling appointments
- Counselor contact info storage
- Session notes from counselors

### Moderation Module (`/src/modules/moderation/`)
- Content moderation with pattern matching (PII detection, blocked terms)
- Tiered severity system
- Reporting system with categories: SAFETY_LEGAL, INTEGRITY, FAITH_SPECIFIC
- Report statuses: PENDING, UNDER_REVIEW, RESOLVED, DISMISSED
- Shadow banning, account banning
- Flag counter per user

### Notifications Module (`/src/modules/notifications/`)
- Create/retrieve notifications
- Types: MATCH, MESSAGE, SUNDAY_REMINDER, MILESTONE, COUNSELING, SYSTEM
- Read status tracking, unread count, bulk mark-as-read

### Devotional Module (`/src/modules/devotional/`)
- Daily devotional questions (14 default questions rotating)
- Get today's question with Bible verse reference
- Auto-generates deterministically based on day of year

### Upload Module (`/src/modules/upload/`)
- Profile photo upload (single file)
- Additional photos (gallery, multiple images)
- Photo removal
- Testimony video upload (15-second max)
- Mission photos (bulk upload, up to 10 files)
- Sunday check-in photos
- Cloudinary integration for all uploads

### Email Module (`/src/modules/email/`)
- Email verification templates (6-digit code)
- Password reset email with token link
- Welcome email post-verification
- HTML email templates with Church branding

---

## Database Schema (Prisma)

### Core Models
- **User:** 40+ fields — auth, personal info, faith profile, location, media, status tracking
- **Match:** Swipe records with unique constraint (user1, user2)
- **Conversation:** 1:1 chats with talking stage tracking (TEMPLE, COURTSHIP, COMMITTED)
- **Message:** Records with read status + moderation flags
- **RefreshToken:** Token storage for session management

### Milestone Models
- **SundayCheckIn:** Church attendance with GPS + photo + verification
- **CelibacyTracker:** Streak tracking with longest streak records
- **MissionUpload:** Mission work with photos + type classification
- **Achievement:** Badge unlocks with milestone values
- **DailyDevotional:** Question + verse per day

### Moderation Models
- **Report:** User reports with category, evidence, moderator notes, status
- **Notification:** Message + status with JSON metadata

### Enums
- Gender (MALE, FEMALE)
- 11 Christian denominations
- Church attendance frequency
- Prayer styles
- 8 lifestyle tags (sober, waiting until marriage, family-oriented, etc.)
- 4 "searching for" categories
- 3 talking stage levels
- User status (active, shadow-banned, banned, under-review, deactivated)
- Report/notification types

---

## Environment Variables Required

```
Database:   DATABASE_URL, DIRECT_URL
Auth:       JWT_SECRET, JWT_REFRESH_SECRET, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
Email:      SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS
Media:      CLOUDINARY_CLOUD_NAME, CLOUDINARY_API_KEY, CLOUDINARY_API_SECRET
Cache:      REDIS_URL, REDIS_HOST, REDIS_PORT
Optional:   SENTRY_DSN, ONESIGNAL_APP_ID, ONESIGNAL_API_KEY, OPENAI_API_KEY
```

---

## Security Features

- JWT with expiring access tokens (15m default) + refresh tokens (7d)
- Password hashing with bcrypt
- Email verification codes
- Rate limiting (3 req/sec short, 20 req/10s medium, 100 req/min long)
- Helmet security headers
- CORS configured for frontend URL
- Global validation pipes (whitelist, transform)
- Soft deletes for user accounts

---

## Not Yet Implemented

- Unit/e2e tests (Jest configured but no test files)
- OpenAI integration for advanced content moderation
- Push notification service (OneSignal config exists but not implemented)
- Admin dashboard for moderation
- Consistent distance-based filtering across all queries
- Sentry error tracking integration
- Community Events (model defined in schema but no controller/service)
- Advanced analytics

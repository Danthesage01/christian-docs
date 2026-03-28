# Frontend (Next.js PWA) — ~85-90% Complete

## Tech Stack

- **Framework:** Next.js 15.1 (React 19, App Router)
- **Language:** TypeScript 5.7 (strict mode)
- **Styling:** Tailwind CSS 3.4 with custom color palette
- **State Management:** Zustand 5.0 (auth + chat stores)
- **Server State:** TanStack React Query 5.62
- **Real-time:** Socket.io-client 4.7.5
- **UI Components:** Radix UI primitives (Avatar, Dialog, Checkbox, Select, Toast, Tabs, Dropdown, Label, Progress)
- **Forms:** React Hook Form 7.54 + Zod 3.24 validation
- **Icons:** Lucide React 0.468 (500+ icons)
- **Animations:** Framer Motion 11.15
- **Toasts:** Sonner 2.0.7
- **File Uploads:** react-dropzone 14.3
- **HTTP Client:** Axios 1.7 with auth interceptors
- **PWA:** next-pwa 5.6 (service workers + manifest)

---

## Project Structure

```
frontend/
├── app/                              # Next.js App Router
│   ├── (auth)/                       # Auth route group
│   │   ├── login/
│   │   ├── signup/                   # 4-step multi-form signup wizard
│   │   ├── verify-email/
│   │   ├── forgot-password/
│   │   ├── reset-password/
│   │   ├── callback/                 # OAuth callback
│   │   └── layout.tsx
│   ├── (dashboard)/                  # Protected dashboard routes
│   │   ├── discover/                 # Card-swiping discovery page
│   │   ├── matches/                  # Matches & likes received tabs
│   │   ├── chat/                     # Conversation list
│   │   │   └── [id]/                 # Individual chat thread
│   │   ├── profile/                  # User profile view
│   │   │   └── edit/                 # Profile editing
│   │   ├── search/                   # Advanced filtering & search
│   │   ├── milestones/               # Faith milestones dashboard
│   │   └── layout.tsx                # Dashboard nav + sidebar
│   ├── page.tsx                      # Landing page
│   ├── layout.tsx                    # Root layout with metadata & PWA
│   └── providers.tsx                 # React Query + Socket provider wrapper
├── components/
│   ├── ui/                           # Radix-based primitives (button, input, card, avatar, label)
│   ├── shared/                       # NotificationToast
│   ├── auth/
│   ├── chat/
│   ├── matching/
│   ├── profile/
│   ├── search/
│   └── milestones/
├── lib/
│   ├── api/                          # API layer (12 modules)
│   │   ├── client.ts                 # Axios instance + interceptors
│   │   ├── auth.ts                   # Login, signup, password reset, Google OAuth
│   │   ├── matching.ts               # Swipe, matches, likes-received
│   │   ├── chat.ts                   # Conversations, messages, mark read
│   │   ├── users.ts                  # Profile CRUD
│   │   ├── search.ts                 # Advanced search
│   │   ├── milestones.ts             # Sunday check-in, celibacy, missions, achievements
│   │   ├── devotional.ts             # Daily devotionals/icebreakers
│   │   ├── counseling.ts             # Counseling services
│   │   ├── notifications.ts          # Notification history
│   │   ├── upload.ts                 # File uploads to Cloudinary
│   │   └── index.ts                  # Exports
│   ├── socket/
│   │   ├── socket.ts                 # Socket.io connection factory
│   │   ├── SocketProvider.tsx        # React context for Socket + notifications
│   │   └── index.ts
│   ├── store/
│   │   ├── auth.store.ts             # Zustand — user state + login/logout/fetch
│   │   ├── chat.store.ts             # Zustand — conversation state
│   │   └── index.ts
│   ├── hooks/                        # Custom hooks (useSocket, useAuth, etc.)
│   ├── utils/
│   │   ├── cn.ts                     # Classname merger (clsx + twMerge)
│   │   └── formatters/               # Date, denomination, initials, etc.
│   ├── constants/                    # App constants
│   └── validations/                  # Zod schemas
├── types/
│   └── index.ts                      # 150+ lines of TS types (User, Auth, Match, Chat, etc.)
├── styles/
│   └── globals.css                   # Tailwind + custom CSS variables
├── public/
│   ├── icons/                        # PWA icons (192x192, 512x512)
│   ├── images/                       # Static images
│   ├── manifest.json                 # PWA manifest
│   └── sw.js / workbox-*.js          # Service worker
├── package.json
├── tsconfig.json                     # Path alias: @/* -> ./*
├── tailwind.config.ts                # Extended theme with custom colors
├── next.config.ts                    # PWA + image optimization
└── .env.local                        # API_URL, WS_URL, Cloudinary config
```

---

## Implemented Features

### Landing Page
- Hero section with CTA
- Features showcase
- Testimonials section
- Responsive layout

### Authentication & Onboarding
- Email/password registration with 4-step form wizard
- Email verification with 6-digit code input
- Login with email/password
- Google OAuth integration
- Forgot password / reset password flows
- OAuth callback handling

### Discovery & Matching
- Card-stack swiping interface (Framer Motion animations)
- Swipe actions: Like (right), Pass (left), Super Like (up)
- Match notifications with toast alerts
- Filter potential matches

### Matches & Connections
- View mutual matches with connection dates
- View likes received / "interested in you" section
- Quick message buttons
- Tabbed interface (Matches vs Likes)

### Real-time Chat
- Conversation list with last message preview
- Individual chat thread view with message history
- Send/receive messages in real-time (Socket.io)
- Mark conversations as read
- Unread message badges on nav

### User Profile
- Public profile view with photos, bio, faith info
- Edit profile form with all user fields
- Photo gallery (multiple photos + profile photo)
- Bio with character counter
- Faith-specific fields: denomination, church, prayer style, life verse, Bible character
- Location (city, state, country)
- Matching preferences

### Search & Discovery
- Advanced search with filters: denomination, age range, distance, city, lifestyle tags
- Search results grid
- Refinement and filtering

### Faith Milestones
- **Sunday Check-in:** Track weekly church attendance with streak counter
- **Celibacy Tracker:** Track abstinence commitment with streak counter
- **Mission Work:** Upload photos and details of mission/volunteer work
- **Achievements:** Unlock badges (Sunday streaks, profile completion, first match, etc.)

### Navigation & Layout
- Desktop sidebar navigation (collapsible)
- Mobile bottom tab navigation
- Dashboard layout with protected routes
- Auth layout with centered forms

### Real-time Notifications
- Socket.io context provider
- Toast notifications for matches, messages, etc.
- Notification history storage (50 max in memory)

---

## UI/UX Design

- **Color Palette:** Purple primary (#d946ef), Gold accent (#f59e0b), Faith blue, clean grays
- **Typography:** Inter font family (Google Fonts)
- **Animations:** Smooth transitions, card swipe with rotation/opacity
- **Responsive:** Mobile-first, dedicated bottom nav, collapsible sidebar
- **Accessibility:** Radix UI primitives provide ARIA compliance
- **Icons:** Lucide React for consistent iconography

---

## Configuration

```
API Backend:    http://localhost:5001/api/v1
WebSocket:      http://localhost:3000
Image Hosting:  Cloudinary (configured, presets need setup)
Google OAuth:   Configured in auth flow
Build Target:   Node.js 20+
```

---

## Not Yet Implemented

- Preferences/privacy/notification settings pages (links exist, pages don't)
- Video upload/viewing UI (testimonyVideoUrl field exists but no upload UI)
- Talking stage management UI (types exist: TEMPLE_STAGE, COURTSHIP_STAGE, COMMITTED_STAGE)
- Devotional/daily icebreaker UI (API exists, UI not built)
- Counseling UI (API exists, UI not built)
- Advanced geolocation/distance-based matching in frontend
- Tests

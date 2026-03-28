# Mobile (React Native / Expo) — ~60-70% Complete

## Tech Stack

- **Framework:** React Native 0.81.5 + Expo 54.0.32
- **React:** 19.1.0
- **Language:** TypeScript 5.9.2 (strict mode)
- **Routing:** Expo Router 6.0.22 (file-based)
- **State Management:** Zustand 5.0.10 (auth, chat, socket stores)
- **Server State:** TanStack React Query 5.90.19
- **HTTP Client:** Axios 1.13.2 with token interceptors
- **Real-time:** Socket.io-client 4.8.3
- **Storage:** AsyncStorage 2.2.0
- **Auth:** Expo Auth Session 7.0.10 (OAuth), Expo Crypto 15.0.8
- **Location:** Expo Location 17.0.1
- **Media:** Expo Image Picker 15.0.7
- **Icons:** Expo Vector Icons (Ionicons)
- **Animations:** React Native Gesture Handler + Reanimated
- **Platforms:** iOS (with tablet), Android (edge-to-edge), Web

---

## Project Structure

```
mobile/
├── app/                              # Expo Router screens (file-based routing)
│   ├── (auth)/                       # Authentication flow
│   │   ├── login.tsx                 # Email/password login + Google OAuth
│   │   ├── register.tsx              # New user signup
│   │   ├── verify-email.tsx          # Email verification
│   │   ├── forgot-password.tsx       # Password recovery
│   │   └── _layout.tsx              # Auth stack layout
│   ├── (tabs)/                       # Main app (5 tabs)
│   │   ├── discover.tsx              # Swipeable profile discovery
│   │   ├── matches.tsx               # Matched connections
│   │   ├── chat/                     # Messaging
│   │   │   ├── index.tsx             # Conversation list
│   │   │   ├── [id].tsx              # Chat detail screen
│   │   │   └── _layout.tsx
│   │   ├── milestones/               # Faith journey tracking
│   │   │   ├── index.tsx             # Dashboard with stats
│   │   │   ├── sunday-checkin.tsx
│   │   │   ├── mission-upload.tsx
│   │   │   └── _layout.tsx
│   │   ├── profile/                  # User profile
│   │   │   ├── index.tsx             # Profile view
│   │   │   ├── edit.tsx              # Profile editing
│   │   │   ├── preferences.tsx       # Match preferences
│   │   │   └── _layout.tsx
│   │   └── _layout.tsx               # Bottom tab navigator
│   ├── _layout.tsx                   # Root layout (auth guard + query provider)
│   └── index.tsx                     # Entry point (redirects to auth/tabs)
├── lib/
│   ├── api/                          # API clients
│   │   ├── client.ts                 # Axios client with interceptors
│   │   ├── auth.ts                   # Authentication endpoints
│   │   ├── matching.ts               # Swiping & matching
│   │   ├── chat.ts                   # Messaging endpoints
│   │   ├── users.ts                  # User profile endpoints
│   │   ├── milestones.ts             # Milestones & achievements
│   │   ├── upload.ts                 # File upload utilities
│   │   └── index.ts
│   └── socket.ts                     # Socket.io config & event handlers
├── store/                            # Zustand stores
│   ├── authStore.ts                  # Auth state & hydration
│   ├── chatStore.ts                  # Chat conversations & messages
│   ├── socketStore.ts                # WebSocket state & notifications
│   └── index.ts
├── types/
│   └── index.ts                      # TypeScript type definitions (346 lines)
├── components/
│   └── ui/                           # Reusable UI components (currently minimal)
├── assets/                           # Icons, splash screen, favicon
├── app.json                          # Expo config (branding: "Church Boys & Girls")
├── tsconfig.json                     # Path aliases (@/*)
└── package.json
```

---

## Implemented Features

### Authentication System (Complete)
- Email/password login & signup
- Email verification with code
- Forgot password / password reset
- Google OAuth integration
- Token management with auto-refresh
- Persistent auth state via AsyncStorage

### Discovery/Swiping (Complete)
- Tinder-like card swiping interface
- Three action types: LIKE (green), PASS (red), SUPER_LIKE (gold)
- Animated pan responder for gesture support
- Profile card displays: photo/initials, name & age, location, denomination & church, bio & life verse
- Info toggle to show/hide details
- Pagination (20 profiles per load)
- Refresh to reload deck
- Match detection with alert modal

### Matches List (Partial)
- Shows matched connections
- Quick view of match info (name, age, denomination, location)
- Chat button to navigate to conversation
- Like counter display
- Pull-to-refresh support

### Real-time Chat (Complete)
- Conversation list with avatar, name, last message preview, timestamps
- Individual chat screen with message bubbles (own vs other styling)
- Real-time message sending via API
- Socket.io integration for incoming messages
- Typing indicators (infrastructure in place)
- Mark as read functionality
- Unread message badge on tab

### Profile Management (Complete)
- View full user profile: avatar, basic info, denomination, church, location, bio, life verse
- Settings menu: Edit Profile, Match Preferences, Notifications, Privacy, Help & Support
- Logout with confirmation

### Milestones/Journey Tracking (Mostly Complete)
- Stats dashboard: Sunday streak, celibacy days, missions count, achievement count
- **Sunday Check-in:** Location-based church check-in, Sunday-only gating
- **Celibacy Tracker:** Start/reset, current & longest streak, daily check-in, private tracking
- **Mission/Volunteer Uploads:** Navigation to upload screen
- **Achievements:** Horizontal scroll view of earned badges with icon, name, description

---

## Data Flow Architecture

### State Management
1. **AuthStore** — User session, login/logout, profile hydration
2. **ChatStore** — Conversations, messages, typing indicators
3. **SocketStore** — Socket connection status, notifications, unread count
4. **React Query** — Server-side data caching (matches, conversations, milestones)

### Real-time Events (Socket.io)
- `connect` / `disconnect` — Connection lifecycle
- `new_message` — Incoming messages in active chat
- `new_notification` — System notifications
- `typing` — Typing indicators
- `mark_read` — Read receipts
- `join_conversation` / `leave_conversation` — Room management

### API Layers
- `authApi` — Login, signup, verification, token management
- `matchingApi` — Swipe, matches, likes
- `chatApi` — Conversations, messages, read status
- `milestonesApi` — Stats, celibacy tracker, check-ins
- `usersApi` — Profile updates
- All wrapped with axios client that auto-injects tokens

---

## Visual Design

- **Primary color:** Magenta/Purple (#d946ef)
- **Neutral palette:** Grays for secondary text
- **Action colors:** Green (like/approve), Red (pass), Gold (super-like)
- **Cards:** White on light gray backgrounds (#f9fafb), 16px rounded corners, shadow elevations
- **Patterns:** Tab navigation, pull-to-refresh, animated swiping with spring physics, loading indicators, empty states, alert modals

---

## Configuration (Development)

```
API URL:        http://localhost:5001
Socket URL:     http://localhost:5001/chat
OAuth Redirect: exp://localhost:8081/(auth)/login
```

All hardcoded — need environment configuration for production.

---

## Not Yet Implemented

- Edit Profile form (navigation exists, form incomplete)
- Match Preferences screen (exists but minimal)
- Notification settings (navigation only, stub)
- Privacy settings (navigation only, stub)
- Help & Support (navigation only, stub)
- Sunday check-in detail form (stub)
- Mission upload form (stub)
- Push notifications
- Image picker not wired up to upload screens
- Typing indicator UI (socket events emitted, no visual)
- Error boundaries / offline support
- Analytics integration
- Reusable UI component library (screens contain inline styles)
- Tests
- Environment configuration for staging/production

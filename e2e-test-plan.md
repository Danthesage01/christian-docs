# AYA — End-to-End Test Plan

This document covers how to manually test every core flow and every fix applied in the audit.

**Prerequisites:**
- Backend running: `cd backend && npm run dev` (port 5001)
- Frontend running: `cd frontend && npm run dev` (port 3000)
- PostgreSQL database connected and migrated (`npx prisma migrate dev`)
- Redis running (for Bull queues / rate limiting)
- `.env` configured with all required variables

---

## 1. Authentication

### 1.1 Email/Password Signup
1. Go to `http://localhost:3000/signup`
2. Fill in Step 1: email, password, confirm password
3. Verify the **eye icon** toggles password visibility on all 3 fields
4. Complete all 4 steps and submit
5. **Expected:** Redirects to verify-email page, email received with 6-digit code

### 1.2 Email Verification
1. Enter the 6-digit code from email
2. **Expected:** Account verified, redirected to discover page
3. **Test rate limiting:** Enter wrong codes rapidly 4+ times within 1 minute
4. **Expected:** 429 Too Many Requests after 3 attempts/min

### 1.3 Login
1. Go to `/login`, enter credentials
2. **Expected:** Logged in, redirected to discover
3. **Test rate limiting:** Enter wrong password 4+ times within 1 minute
4. **Expected:** 429 Too Many Requests after 3 attempts/min

### 1.4 Forgot Password
1. Go to `/forgot-password`, enter email
2. **Expected:** Email received with reset link
3. Click the link, set new password
4. **Expected:** Password updated, can log in with new password
5. **Test reuse:** Try using the same reset link again
6. **Expected:** "Invalid or expired reset token" (token was hashed and cleared)

### 1.5 Google OAuth
1. Click "Sign in with Google"
2. **For new user with unique email:** Account created, welcome email received
3. **For existing email (already registered with password):**
4. **Expected:** Error: "An account already exists with this email. Please log in first to link."
5. Verify tokens are NOT in the redirect URL (check browser URL bar)
6. **Expected:** URL only has `?success=true&isProfileComplete=...`, tokens in HTTP-only cookies

### 1.6 Logout
1. Click logout
2. **Expected:** Redirected to login, tokens cleared
3. Try accessing `/discover` directly
4. **Expected:** Redirected to login

### 1.7 Banned/Deactivated User
1. In database, set a user's `status` to `SHADOW_BANNED`, `UNDER_REVIEW`, or `DEACTIVATED`
2. Try to use the app with that user's token
3. **Expected:** 401 Unauthorized on all API calls

---

## 2. Profile

### 2.1 View Profile
1. Go to `/profile`
2. **Expected:** Shows all profile info — name, age, denomination, church, bio, life verse, photos
3. Version text shows "Aya v1.0.0" (not "Church Boys & Girls")

### 2.2 Edit Profile
1. Go to `/profile/edit`
2. Change bio, denomination, photos
3. Save
4. **Expected:** Changes reflected on profile page

---

## 3. Matching / Swiping

### 3.1 Discover Page
1. Go to `/discover`
2. **Expected:** Profile cards appear with name, age, city, denomination, life verse
3. Swipe right (or tap heart) → LIKE
4. Swipe left (or tap X) → PASS
5. Swipe up (or tap star) → SUPER_LIKE
6. **Expected:** Each swipe calls API, card advances on success

### 3.2 Swipe Error Handling
1. Disconnect backend (stop the server)
2. Try swiping on a card
3. **Expected:** Error toast "Failed to record swipe. Please try again." — card does NOT advance
4. Restart backend, swipe again
5. **Expected:** Works normally

### 3.3 Self-Swipe Prevention
1. Using API directly (Postman/curl):
   ```bash
   POST /api/v1/matching/swipe
   Body: { "targetUserId": "<your-own-user-id>", "action": "LIKE" }
   ```
2. **Expected:** 400 Bad Request: "Cannot swipe on yourself"

### 3.4 Double-Swipe Prevention
1. Swipe LIKE on a user
2. Using API directly, try swiping on the same user again:
   ```bash
   POST /api/v1/matching/swipe
   Body: { "targetUserId": "<same-user-id>", "action": "PASS" }
   ```
3. **Expected:** 400 Bad Request: "You have already swiped on this user"

### 3.5 Invalid Target User
1. Using API directly:
   ```bash
   POST /api/v1/matching/swipe
   Body: { "targetUserId": "00000000-0000-0000-0000-000000000000", "action": "LIKE" }
   ```
2. **Expected:** 404 Not Found: "Target user not found"

### 3.6 DTO Validation
1. Using API directly:
   ```bash
   POST /api/v1/matching/swipe
   Body: { "targetUserId": "not-a-uuid", "action": "INVALID" }
   ```
2. **Expected:** 400 Bad Request with validation error messages

### 3.7 Mutual Match
1. Create two test users (User A, User B)
2. User A swipes LIKE on User B
3. User B swipes LIKE on User A
4. **Expected:** Both users see "It's a match!" notification. Conversation created automatically.

### 3.8 Distance Filtering
1. Set User A's location to Lagos (lat: 6.5244, lng: 3.3792) with maxDistanceKm: 50
2. Set User B's location to Lagos (lat: 6.5, lng: 3.4) — within 50km
3. Set User C's location to Abuja (lat: 9.0579, lng: 7.4951) — far away
4. User A opens discover page
5. **Expected:** User B appears, User C does NOT appear

### 3.9 SUPER_LIKE Priority
1. User B sends LIKE to User A
2. User C sends SUPER_LIKE to User A
3. User A checks "Likes Received"
4. **Expected:** User C (SUPER_LIKE) appears before User B (LIKE)

---

## 4. Chat / Messaging

### 4.1 Send & Receive Messages
1. Create a match between User A and User B (both swipe LIKE)
2. User A opens the conversation
3. User A types a message and sends
4. **Expected:** Message appears in User A's chat immediately
5. Open User B's chat
6. **Expected:** Message appears in User B's chat (real-time via WebSocket)

### 4.2 Mark as Read
1. User A sends a message to User B
2. User B opens the conversation
3. **Expected:** Messages automatically marked as read on page load

### 4.3 Message Deduplication
1. Send a message in a conversation
2. **Expected:** Message appears only ONCE (not duplicated by socket + REST response)

### 4.4 Authorization — Read Others' Messages
1. User C is NOT part of User A + User B's conversation
2. Using API directly as User C:
   ```bash
   GET /api/v1/chat/conversations/<A-B-conversation-id>/messages
   Authorization: Bearer <user-c-token>
   ```
3. **Expected:** 403 Forbidden: "You are not part of this conversation"

### 4.5 Authorization — Send to Others' Conversation
1. Using API directly as User C:
   ```bash
   POST /api/v1/chat/conversations/<A-B-conversation-id>/messages
   Authorization: Bearer <user-c-token>
   Body: { "content": "Unauthorized message" }
   ```
2. **Expected:** 403 Forbidden: "You cannot send messages in this conversation"

### 4.6 WebSocket — Join Others' Conversation
1. Connect to WebSocket as User C
2. Emit `join_conversation` with User A + B's conversation ID
3. **Expected:** WsException: "Not authorized to join this conversation"

### 4.7 Empty Message Validation
1. Try sending an empty message or whitespace-only:
   ```bash
   POST /api/v1/chat/conversations/<id>/messages
   Body: { "content": "" }
   ```
2. **Expected:** 400 Bad Request

### 4.8 Message Length Limit
1. Try sending a message longer than 5000 characters
2. **Expected:** 400 Bad Request (MaxLength validation)

### 4.9 Rate Limiting
1. Send 31+ messages within 1 minute
2. **Expected:** 429 Too Many Requests after 30 messages/min

### 4.10 Chat Error UI
1. Stop the backend
2. Navigate to a chat conversation
3. **Expected:** Error screen with "Something went wrong" and "Go back" button (not blank page)

---

## 5. WebSocket Security

### 5.1 Token Authentication
1. Connect to WebSocket without a token
2. **Expected:** Connection rejected

### 5.2 No Query Param Tokens
1. Try connecting with token in query params: `ws://localhost:5001/chat?token=xxx`
2. **Expected:** Connection rejected (query param extraction removed)

### 5.3 Socket Reconnection
1. Log in and open a chat
2. Stop the backend for 5 seconds, then restart
3. **Expected:** Socket auto-reconnects with fresh token from localStorage

---

## 6. Frontend Branding

### 6.1 Verify Rebrand
Check these locations show "Aya" (not "Church Boys & Girls" or "ChurchConnect"):

- [ ] Landing page header logo + text
- [ ] Landing page footer logo + text + copyright
- [ ] Auth layout header + footer
- [ ] Dashboard sidebar logo + text
- [ ] Profile page version text
- [ ] Browser tab title: "Aya - Faith-Based Dating"
- [ ] PWA manifest (`/manifest.json`): name "Aya", short_name "Aya"
- [ ] Theme color: `#5A3D8C` (royal purple, not magenta)

### 6.2 Color Palette
- [ ] Primary buttons/links use royal purple (#5A3D8C), not magenta (#d946ef)
- [ ] Accent uses vibrant violet (#9B28FF)
- [ ] Faith/sky blue uses #8BD5F1 tones
- [ ] CTA sections use purple gradients

### 6.3 Logo
- [ ] Logo image (`/images/logo.png`) displays correctly in header, footer, sidebar, auth pages
- [ ] PWA icon uses purple (not magenta) — check `/icons/icon.svg`

### 6.4 Landing Page Variants
- [ ] `/home-a` loads — bold visual with card stack hero
- [ ] `/home-b` loads — warm/story-driven with couple testimonials
- [ ] `/home-c` loads — premium dark with phone mockup

---

## 7. Landing Page Sections

### 7.1 Main Page (`/`)
Verify all sections render and interactions work:

- [ ] Header — scrolls to glass blur
- [ ] Hero — phone mockup cycles through profiles, floating notification cards animate
- [ ] Stats bar — numbers animate in on scroll
- [ ] Profile cards — 3 mock profiles, middle one larger, hover lifts
- [ ] Features — 6 cards fade in on scroll, icons scale on hover
- [ ] How It Works — dark section, 3 steps with dashed connectors
- [ ] Talking Stages — left/right layout, stages slide in, dashboard cards
- [ ] Testimonials — auto-rotates every 5s, dot navigation works
- [ ] Values — 4 cards with colored top borders, hover shadow
- [ ] Counselor (Pro) — counselor cards with verified badges, Pro badge
- [ ] Comparison table — dark section, checkmarks/X/minus icons correct
- [ ] FAQ accordion — all 8 questions expand/collapse, only one open at a time (or multiple OK)
- [ ] Install CTA — iPhone + Android instructions, phone mockups render
- [ ] Final CTA — gradient section with button
- [ ] Footer — 3 columns, links work

### 7.2 Password Visibility
- [ ] Login page — eye icon toggles password field
- [ ] Signup page — eye icon on both password and confirm password fields

---

## 8. PWA

### 8.1 Install Test (requires production build)
```bash
cd frontend && npm run build && npm start
```
1. Open `http://localhost:3000` in Chrome
2. **Expected:** "Install" prompt appears in address bar (or use DevTools → Application → Manifest)
3. Install the app
4. **Expected:** Opens in standalone mode, no browser chrome
5. Check the app icon matches the Aya purple branding

### 8.2 Manifest Check
1. Open `http://localhost:3000/manifest.json`
2. **Expected:**
   - name: "Aya"
   - short_name: "Aya"
   - theme_color: "#5A3D8C"
   - Icons: 192x192 and 512x512

---

## 9. Email

### 9.1 Welcome Email
1. Sign up with email/password → verify email
2. **Expected:** Welcome email received after verification
3. Sign up with Google OAuth (new email)
4. **Expected:** Welcome email received after account creation

### 9.2 Verification Email
1. Sign up with email
2. **Expected:** Verification email with 6-digit code
3. Click "Resend" on verify page
4. **Expected:** New verification email sent

### 9.3 Password Reset Email
1. Request password reset
2. **Expected:** Email with reset link containing token

---

## Quick Smoke Test Checklist

For a fast sanity check, run through this:

1. [ ] Sign up with email → verify → lands on discover
2. [ ] See profile cards with swipe buttons
3. [ ] Swipe LIKE on someone → no errors
4. [ ] Create second user, swipe LIKE back → match notification
5. [ ] Open chat → send message → appears instantly
6. [ ] Open second user's chat → message visible
7. [ ] Edit profile → changes save
8. [ ] Log out → redirected to login
9. [ ] Log back in → discover page loads
10. [ ] Landing page loads with all sections, correct branding

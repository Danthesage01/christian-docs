# AYA — Future Features Roadmap

This document tracks planned features for future development.

---

## Pro / Premium Features

### 1. Rewind (Undo Pass)
**Priority:** High
**Tier:** Pro

Allow users to undo their last PASS and re-see that profile.

**How it works:**
- "Rewind" button appears on the discover screen (replaces the passed card)
- Free users: no rewinds (pass is final)
- Pro users: unlimited rewinds (or X per day)

**Backend changes needed:**
- New endpoint: `POST /matching/rewind` — deletes the most recent PASS record for the user
- Add `rewindable` window (e.g., only rewind within 30 seconds of passing)
- Track rewind count per day for limit enforcement
- Return the rewound profile so the frontend can re-display it

**Frontend changes needed:**
- "Undo" toast/button that appears for 5 seconds after a PASS
- Tapping it calls the rewind endpoint and puts the card back on top
- Pro gate: show upgrade prompt if free user taps rewind

**Database changes:**
- Add `rewindCount` (daily) to User or a separate UserQuota table
- Add `lastRewindAt` timestamp

---

### 2. In-App Counselor Sessions (Pro)
**Priority:** High
**Tier:** Pro

Connect couples with verified, certified married counselors for faith-based relationship guidance.

**Counselor requirements:**
- Verified married couple with 10+ years of marriage
- Certified in faith-based relationship counseling
- Vetted and approved by Aya team

**How it works:**
- User requests a session from their conversation (Committed stage only, or any stage for Pro)
- Selects from available counselors (filtered by specialty)
- Books a time slot
- Session happens via in-app video call or chat
- Counselor can add notes after session
- Both users can rate the session

**Backend changes needed:**
- Counselor role/user type (separate from regular users)
- Counselor profile model: specialties, availability, credentials, rating
- Counselor verification workflow (admin approval)
- Session booking system with time slots
- Session model: status (REQUESTED, SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED)
- Session notes (counselor-only)
- Rating/review system
- Payment integration (Stripe/Paystack) for Pro subscription

**Frontend changes needed:**
- Counselor directory page (browse available counselors)
- Counselor profile card (photo, name, years married, specialty, rating)
- Booking flow (select counselor → pick time → confirm)
- Session history page
- Rating modal after session
- Pro gate: upgrade prompt for free users

**Existing scaffolding:**
- Backend `counseling` module exists with basic CRUD
- Landing page already has Pro counselor section
- Database has CounselingSession model

---

### 3. See Who Liked You (Pro)
**Priority:** High
**Tier:** Pro

Free users see a blurred count of likes. Pro users see full profiles.

**How it works:**
- Free: "X people liked you" with blurred cards
- Pro: Full profile cards with ability to like back instantly

**Backend changes needed:**
- Existing `getLikesReceived` endpoint already returns this data
- Add a `isPro` check — free users get count only, Pro users get full profiles
- Subscription/payment model

**Frontend changes needed:**
- Blurred card overlay for free users with "Upgrade to see" CTA
- Full cards for Pro users
- Likes tab already exists in matches page

---

## Free Features

### 4. Daily Devotional Icebreakers
**Priority:** Medium
**Tier:** Free

Bible-based conversation starter shown daily to spark meaningful first messages.

**How it works:**
- One devotional question per day (rotates from pool of 365)
- Shown in chat as a suggested first message
- Also shown on discover cards as a prompt
- Users can tap to send it as their first message

**Backend changes needed:**
- Devotional module already exists with 14 rotating questions
- Expand to 365 questions
- Add endpoint to get today's devotional as a chat prompt
- Track which devotionals a user has used

**Frontend changes needed:**
- Devotional card on the home/discover page
- "Send as icebreaker" button in new conversations
- Devotional history in profile

---

### 5. Video Testimony
**Priority:** Medium
**Tier:** Free

15-second video testimony on profiles — share your faith story.

**How it works:**
- Record or upload a 15-second video
- Displayed on your profile card
- Other users can view it before swiping

**Backend changes needed:**
- Upload endpoint exists (`uploadApi.uploadTestimonyVideo`)
- `testimonyVideoUrl` field exists on User model
- Add video compression/transcoding (Cloudinary handles this)

**Frontend changes needed:**
- Video recording UI (camera access)
- Video player on profile cards
- Upload progress indicator
- Video preview before saving

---

### 6. Profile Verification Badge
**Priority:** Medium
**Tier:** Free

Selfie-based verification to prove you're a real person.

**How it works:**
- User takes a selfie matching a specific pose
- AI or manual review compares to profile photos
- Verified badge shown on profile

**Backend changes needed:**
- Verification request endpoint
- Photo comparison logic (manual review initially, AI later)
- `isVerified` field on User model
- Admin review queue

**Frontend changes needed:**
- Verification flow (camera → pose → submit)
- Badge display on profile and discover cards

---

### 7. Events & Community
**Priority:** Low
**Tier:** Free

Local church events and community gatherings.

**How it works:**
- Churches/organizers post events
- Users discover events near them
- RSVP and see who else is going
- Potential matching at events

**Backend changes needed:**
- CommunityEvent model exists in schema (scaffolded, not implemented)
- Event CRUD endpoints
- RSVP system
- Location-based event discovery

**Frontend changes needed:**
- Events tab or section
- Event cards with details, location, RSVP button
- Attendee list
- Map view (optional)

---

### 8. Prayer Partner Matching
**Priority:** Medium
**Tier:** Free

Dedicated matching for prayer partners (not romantic).

**How it works:**
- User sets "searching for" to PRAYER_PARTNER
- Matched with others seeking the same
- Separate conversation thread marked as "Prayer"
- Daily prayer reminders

**Backend changes needed:**
- Search module already has prayer partner finder
- Add dedicated prayer partner matching flow
- Prayer reminder notifications (cron job)

**Frontend changes needed:**
- Prayer partner toggle in preferences
- Distinct UI for prayer conversations (different color/badge)
- Prayer request sharing within conversation

---

### 9. Read Receipts & Typing Indicators (Visual)
**Priority:** Medium
**Tier:** Free

Show message delivery status and typing animation.

**How it works:**
- Single check: sent
- Double check: delivered
- Blue double check: read
- Animated dots when other user is typing

**Backend changes needed:**
- Typing events already implemented in WebSocket gateway
- Read status already tracked (`isRead`, `readAt`)
- Add `deliveredAt` field to Message model

**Frontend changes needed:**
- Check mark icons on each message
- Typing indicator animation (three bouncing dots)
- Status updates via WebSocket

---

### 10. Push Notification Preferences
**Priority:** Low
**Tier:** Free

Granular control over which notifications to receive.

**How it works:**
- Toggle per notification type: messages, matches, likes, Sunday reminders, milestones
- Quiet hours setting
- Email vs push preference

**Backend changes needed:**
- NotificationPreferences model on User
- Check preferences before sending push
- Quiet hours enforcement

**Frontend changes needed:**
- Notification settings page (currently a stub at /profile/notifications)
- Toggle switches per type
- Quiet hours time picker

---

## Infrastructure / Technical

### 11. Analytics Dashboard
**Priority:** Low
**Tier:** Internal

Track user engagement, match rates, conversion funnels.

### 12. Admin Moderation Dashboard
**Priority:** High
**Tier:** Internal

Review reports, ban users, manage content moderation queue.

### 13. A/B Testing Framework
**Priority:** Low
**Tier:** Internal

Test landing page variants, feature flags, pricing.

### 14. Mobile App (React Native)
**Priority:** Medium
**Tier:** Free

Native iOS + Android app (codebase exists at `/mobile`, ~60-70% complete).

---

## Monetization

### Subscription Tiers (Planned)

| Feature | Free | Pro |
|---------|------|-----|
| Swiping | ✅ | ✅ |
| Matching | ✅ | ✅ |
| Messaging | ✅ | ✅ |
| Super Likes | 1/day | Unlimited |
| Rewind (undo pass) | ❌ | ✅ |
| See who liked you | Count only | Full profiles |
| Counselor sessions | ❌ | ✅ |
| Profile boost | ❌ | ✅ |
| Advanced filters | Basic | All |
| Read receipts | ❌ | ✅ |
| Ad-free | ❌ | ✅ |

### Payment Integration
- Stripe (international)
- Paystack (Nigeria/Africa)
- Apple/Google IAP (when mobile app launches)

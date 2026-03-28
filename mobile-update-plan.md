# Mobile App Update Plan

Bring the React Native/Expo mobile app fully up to date with all frontend and backend changes made on 2026-03-28.

---

## Execution Strategy

Run in **4 parallel batches** to maximise speed. Each batch has independent files.

---

## Batch 1 — Rebrand + Config (5 files)

### 1.1 `app.json`
- Change `"name"` from `"Church Boys & Girls"` to `"Aya"`
- Change `"slug"` to `"aya"`
- Update `"scheme"` to `"aya"`
- Update description

### 1.2 `lib/api/client.ts`
- Change hardcoded `http://localhost:5001` to read from env/constants
- Add same token refresh interceptor logic as frontend (if not already present)

### 1.3 `lib/socket.ts`
- Change hardcoded `http://localhost:5001` to configurable URL
- Set `reconnection: false` (match frontend fix — no infinite loop)
- On `connect_error`: disconnect cleanly, don't retry with stale token

### 1.4 `assets/` — App icons
- Replace with Aya purple branding (if logo assets available)

### 1.5 `app.json` colors
- Update `"backgroundColor"` to `"#5A3D8C"`
- Update `"primaryColor"` to `"#5A3D8C"`
- Update splash screen color

---

## Batch 2 — API Layer (7 files)

### 2.1 `lib/api/matching.ts`
- Add `getDailyLimits()` method:
  ```typescript
  getDailyLimits: async () => {
    const response = await apiClient.get('/matching/daily-limits');
    return response.data;
  }
  ```
- Ensure `swipe()` sends `{ targetUserId, action }` matching backend SwipeDto

### 2.2 `lib/api/chat.ts`
- Add `markAsRead(conversationId)` method if missing
- Ensure `sendMessage()` matches backend SendMessageDto
- Ensure `getMessages()` pagination params are sent

### 2.3 `lib/api/auth.ts`
- Ensure signup/login/verify/reset flows match backend changes
- Google OAuth: handle new error "Account already exists with this email"

### 2.4 `lib/api/search.ts` (create if missing)
- Add `searchUsers(filters)` with `query` param support
- Add `findPrayerPartners()`, `findChurchPartners()`
- Include `searchingFor` filter param

### 2.5 `lib/api/notifications.ts` (create if missing)
- Add `getNotifications(unreadOnly)`, `getUnreadCount()`
- Add `markAsRead(id)`, `markAllAsRead()`
- Add `getVapidKey()`, `subscribePush()`, `unsubscribePush()`

### 2.6 `lib/api/upload.ts`
- Verify upload methods match backend endpoints
- Ensure Cloudinary integration works

### 2.7 `lib/api/index.ts`
- Export all new API modules

---

## Batch 3 — Store + Socket (4 files)

### 3.1 `store/authStore.ts`
- Ensure token storage uses AsyncStorage correctly
- Add `isHydrated` flag if missing

### 3.2 `store/chatStore.ts`
- Add `error: string | null` field
- Add `clearError()` action
- Add message deduplication in `addMessage()` (check by ID before appending)
- Only update `lastMessageAt` if new message is more recent

### 3.3 `store/socketStore.ts`
- Load unread notifications from server on connect (match frontend SocketProvider)
- Track `isConnected` state
- Handle disconnect without infinite retry

### 3.4 `lib/socket.ts` (continued from 1.3)
- Remove `reconnection: true`
- Add `connect_error` handler that disconnects cleanly
- Remove query param token support

---

## Batch 4 — Screens (12 files)

### 4.1 `app/(auth)/login.tsx`
- Update branding (logo, name "Aya")
- Add toast/Alert on success ("Welcome back!") and error
- Handle Google OAuth "account exists" error from backend

### 4.2 `app/(auth)/register.tsx`
- Update branding
- Add toast on success ("Account created! Check your email")
- Add toast on error

### 4.3 `app/(auth)/verify-email.tsx`
- Add toast on verification success/error
- Add toast on resend success/error

### 4.4 `app/(auth)/forgot-password.tsx`
- Add toast on success/error

### 4.5 `app/(tabs)/discover.tsx`
**Major changes:**
- Add daily limits display (remaining swipes + super likes in header)
- Add daily limit reached screen with "Upgrade to Pro" CTA
- Disable Super Like button when limit reached (gray out, show "Used")
- Add action labels below buttons ("Pass", "Super Like", "Like")
- Handle "already swiped" error (skip card silently)
- Handle "daily limit" error (show limit screen)
- Remove refresh button from header
- Update "no profiles" empty state ("You're all caught up!")
- Fetch daily limits after each swipe (`getDailyLimits()`)

### 4.6 `app/(tabs)/matches.tsx`
**Major changes:**
- Add tab navigation: "Matches" | "Likes" with underline indicator
- Likes tab: show Like Back + Pass buttons on each card
- Like Back calls `matchingApi.swipe({ targetUserId, action: 'LIKE' })`
- Pass calls `matchingApi.swipe({ targetUserId, action: 'PASS' })`
- Show "It's a Match!" alert on successful like-back
- Show Super Like badge on likes cards
- Show match count and likes count on tabs

### 4.7 `app/(tabs)/chat/index.tsx`
- Load unread notifications from server on mount
- Show unread count badge per conversation
- Bold text for unread conversations
- Online status dot on avatars
- Remove talking stage badge from list (too noisy)
- Add skeleton loading rows

### 4.8 `app/(tabs)/chat/[id].tsx`
**Major changes:**
- Replace `TextInput` with multiline input (auto-resize)
- Add emoji picker tray (quick emojis row)
- Add heart quick-send when input is empty
- Add icebreaker prompts on empty conversation
- Messages: `whitespace-pre-wrap` equivalent (handle `\n` in messages)
- Mark messages as read on page mount (`chatApi.markAsRead`)
- Mark server notifications as read (`notificationsApi.markAsRead`)
- Message deduplication (check ID before adding from socket)
- Use `messagesRef` pattern to avoid stale closure

### 4.9 `app/(tabs)/milestones/index.tsx`
- Increase spacing (more padding, larger stat cards)
- Larger progress rings
- Wider achievement badges

### 4.10 `app/(tabs)/profile/index.tsx`
- Update branding ("Aya v1.0.0")
- Stats row (Matches, Likes, Views)
- Avatar ring styling
- Faith blockquote for life verse
- Styled menu items with icons + descriptions
- Toast on logout

### 4.11 `app/(tabs)/profile/edit.tsx`
- Toast on save success/error
- Verify form fields match backend expectations

### 4.12 `app/(tabs)/profile/preferences.tsx`
- Verify preference fields match backend (age range, distance, denominations)

---

## Batch 5 — New Screens (2 files)

### 5.1 `app/(tabs)/search.tsx` (create or update)
- Search bar with text query
- "Looking For" filter (Life Partner, Prayer Partner, Church Partner)
- Denomination filter
- Age range filter
- Correct lifestyle tags (match Prisma enum)
- Load More pagination
- Result cards with View Profile link

### 5.2 `app/profile/[id].tsx` (create)
- Public profile view for other users
- Avatar, name, age, location
- Bio, faith section, lifestyle tags
- Back button

---

## Batch 6 — Types + Constants (2 files)

### 6.1 `types/index.ts`
- Add `DailyLimits` type
- Ensure `SwipeRequest` matches backend `SwipeDto`
- Add `SearchFilters`, `SearchResult` types
- Ensure all types match the frontend `types/index.ts`

### 6.2 Constants file (create if needed)
- API URL, Socket URL (configurable)
- Color palette constants (#5A3D8C, #9B28FF, #8BD5F1)
- Daily limit constants (20 swipes, 1 super like)

---

## Color Palette Reference

```
Primary (Royal Purple): #5A3D8C
Accent (Vibrant Violet): #9B28FF
Faith (Sky Blue): #8BD5F1
Dark: #111111
White: #FFFFFF
```

---

## Testing Checklist (after implementation)

- [ ] App opens, shows Aya branding
- [ ] Login/signup with toast feedback
- [ ] Discover shows profiles with daily limit badges
- [ ] Swipe LIKE/PASS/SUPER_LIKE works with limits enforced
- [ ] "Daily limit reached" screen shows when swipes exhausted
- [ ] Super Like button grays out after use
- [ ] Matches tab shows matches, Likes tab shows likes
- [ ] Like Back / Pass buttons work on Likes tab
- [ ] Chat list shows unread badges
- [ ] Chat input: multiline, emoji tray, heart quick-send
- [ ] Chat icebreakers on empty conversation
- [ ] Messages marked as read on open
- [ ] Search with text query + filters + Load More
- [ ] Profile shows stats, logout toast
- [ ] No socket infinite reconnection loop

---

## Estimated Execution Time

With parallel batches and pre-approval:
- Batch 1 (config): ~5 min
- Batch 2 (API): ~10 min
- Batch 3 (store/socket): ~5 min
- Batch 4 (screens): ~20 min
- Batch 5 (new screens): ~10 min
- Batch 6 (types): ~5 min
- **Total: ~40-50 minutes**

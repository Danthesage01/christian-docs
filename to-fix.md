# AYA — Issues To Fix

Audit conducted: 2026-03-28

---

## Phase 1 — Blockers ✅ DONE

### 1. Chat Authorization Checks
- [x] `getMessages()` — verify user is conversation participant before returning messages
- [x] `sendMessage()` — verify sender is conversation participant + validate content not empty
- [x] WebSocket `join_conversation` — verify user is participant before joining room
- [x] WebSocket CORS — changed from `origin: '*'` to env-based allowed origins

### 2. Matching Transaction Safety
- [x] Wrap swipe + conversation creation in `prisma.$transaction()`
- [x] Add self-swipe validation (`userId !== targetUserId`)
- [x] Add target user existence check before creating match
- [x] Add DTO validation with class-validator (`SwipeDto` with `@IsUUID`, `@IsIn`)

### 3. Rate Limiting on Auth Endpoints
- [x] Add `@Throttle()` to login (3 attempts/min)
- [x] Add `@Throttle()` to verify-email (3 attempts/min)
- [x] Add `@Throttle()` to forgot-password (3 attempts/min)
- [x] Add `@Throttle()` to reset-password (3 attempts/min)
- [x] Add `@Throttle()` to signup (5 attempts/min)

### 4. Token & Code Security
- [x] Use `crypto.randomInt()` instead of `Math.random()` for verification codes
- [x] Fix Google OAuth auto-linking — now throws error instead of silently linking

### 5. Fix Frontend Socket & Error Handling
- [x] Replace hardcoded `http://localhost:5001` with `process.env.NEXT_PUBLIC_SOCKET_URL`
- [x] Add `onError` to swipe mutation — shows toast, doesn't advance card
- [x] Add message deduplication in chat (check by ID before adding)
- [x] Add `markAsRead()` call when chat page mounts

---

## Phase 2 — High Priority ✅ DONE

### 6. DTO Validation on Chat
- [x] Create `SendMessageDto` with `@IsNotEmpty()`, `@IsString()`, `@MaxLength(5000)`
- [x] Apply to chat controller endpoint

### 7. Auth Hardening
- [x] Hash refresh tokens with bcrypt before DB storage
- [x] Hash password reset tokens with bcrypt before DB storage
- [x] Remove query param token support from WebSocket guard
- [x] Add non-ACTIVE status rejection in JWT strategy (covers SHADOW_BANNED, UNDER_REVIEW, DEACTIVATED)

### 8. Chat Rate Limiting
- [x] Add `@Throttle()` to sendMessage (30 messages/min)

---

## Phase 3 — Polish ✅ DONE

### 9. Matching Enhancements
- [x] Implement distance filtering using lat/lng and `maxDistanceKm`
- [x] SUPER_LIKE prioritization in getLikesReceived (sorted before LIKE)
- [x] Prevent double-swipe — throws error if already swiped

### 10. Frontend Polish
- [x] Add error field to chat store with `clearError()` action
- [x] Socket auto-reconnect with fresh token on forced disconnect
- [x] Show error UI in chat page when messages fail to load

### 11. OAuth & Auth Cleanup
- [x] Move OAuth callback tokens to HTTP-only cookies instead of URL params

---

## Remaining (Future Backlog)

- [ ] Access token blacklist on logout (Redis JTI or token version increment)
- [ ] Account lockout after 5 failed login attempts
- [ ] Conversation deletion/archiving
- [ ] Muting enforcement (DB fields exist but not checked)
- [ ] Chat message pagination UI (load more / infinite scroll)
- [ ] SUPER_LIKE daily limit enforcement

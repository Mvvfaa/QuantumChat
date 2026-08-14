# QuantumChat — Requirements

Product brief: **Real-time. End-to-end encrypted. Client-held keys.**

This document describes the **current** QuantumChat messenger (MERN + Socket.IO + tweetnacl), not the older widget / multi-tenant admin Control Center brief.

---

## 1. Product vision

QuantumChat is a privacy-first messaging app where:

- Messages, attachments, and reactions are **encrypted in the browser** before they reach the server.
- The server stores and relays **ciphertext only** — it has no private keys and cannot read chat content.
- Each account uses a **fixed 5-key X25519 pool** published as public keys; private keys live only on the device (local keyring + `keys.txt` backup).
- Optional **QuantumAI** assists users with **explicit, client opt-in context** — AI never receives plaintext from the QuantumChat server’s message store.

---

## 2. System units

| Unit | Role | Typical local URL |
|------|------|-------------------|
| QuantumChat backend | Auth, users, messages, groups, stories, attachments, Socket.IO | `http://localhost:5000` |
| QuantumChat frontend | Messenger UI + all crypto | `http://localhost:5173` |
| QuantumAI backend *(companion)* | AI APIs (separate service) | `http://localhost:5001` |
| QuantumAI frontend *(optional)* | Standalone AI UI; shares QuantumChat JWT | `http://localhost:5175` |

CORS: set `CLIENT_URL` on the QuantumChat backend to include both `5173` and `5175` when running QuantumAI locally.

---

## 3. Feature requirements & status

### 3.1 Core messaging

| Requirement | Status | Notes |
|-------------|--------|--------|
| Real-time DMs | **Done** | Socket.IO + REST; sealed-box envelopes (`forRecipient` / `forSender`) |
| Real-time groups | **Done** | Create/update/delete, members, admins, invites, pin, posting rules |
| Typing indicators | **Done** | Peer typing (DM-oriented socket flow) |
| Online / last-seen presence | **Done** | Privacy-gated (`privacy.online`, `privacy.lastSeen`) |
| Delivery & read receipts | **Done** | `deliveredAt` / `readAt`; privacy toggle for read receipts |
| Reply, forward, edit, delete | **Done** | Forward re-encrypts; delete-for-everyone + delete-for-me |
| Reactions | **Done** | Sealed emoji reactions |
| Pins & stars | **Done** | Group pins server-side; stars local |
| Mentions (groups) | **Done** | Encoded in group message payload |
| In-thread message search | **Done** | Client-side over decrypted thread |
| Conversation list search / filters | **Done** | Unread, groups, archived, etc. |
| Mute / archive / hide chat | **Done** | Local prefs; also exposed from public profile sheet |
| Block users | **Done** | Server block list + UI |
| Polls & events (groups) | **Done** | Encrypted group kinds + poll votes |
| Announcements (groups) | **Done** | Admin-oriented group message kind |

### 3.2 Media & stories

| Requirement | Status | Notes |
|-------------|--------|--------|
| Encrypted file attachments | **Done** | Client seals bytes; server stores ciphertext blobs |
| Voice notes | **Done** | MediaRecorder (~60s), encrypted as attachments |
| Camera photo capture | **Done** | Still capture (not calls) |
| Drag-and-drop upload | **Done** | Composer drop zone |
| Stories (24h) | **Done** | Image/video/audio + caption; **not** sealed like chat messages |

### 3.3 Auth, profile & privacy

| Requirement | Status | Notes |
|-------------|--------|--------|
| Register / login (email + password) | **Done** | JWT session |
| Email verification | **Done** | Verify + resend |
| Forgot / reset password | **Done** | Tokenized email flow |
| Public profile fields | **Done** | `displayName`, `bio`, `avatar`, username |
| Phone number | **Done** | Private to self only |
| Privacy controls | **Done** | Last seen, online, read receipts |
| Public profile viewer | **Done** | Click DM header → fetch `GET /users/:id` |
| Account export / delete | **Done** | Settings → Data |
| Themes | **Done** | Light / dark via theme context |

### 3.4 End-to-end cryptography

| Requirement | Status | Notes |
|-------------|--------|--------|
| Client-side key generation | **Done** | 5-key pool at registration (`tweetnacl` / X25519 sealed box) |
| Server never receives private keys | **Done** | Only public keys + ciphertext |
| Dual envelopes (sender + recipient) | **Done** | Both parties can decrypt their own copies |
| Group sealed / secretbox payloads | **Done** | Group message & attachment modes |
| Download `keys.txt` backup | **Done** | Offered at register / regenerate |
| Import keys; reject wrong account file | **Done** | Derived publics must match account `publicKeys` |
| Require keys after every login | **Done** | Login clears local keyring; unlock UI before chat |
| Regenerate keys (recovery) | **Done** | Replaces published pool; old history unreadable |

### 3.5 QuantumAI

| Requirement | Status | Notes |
|-------------|--------|--------|
| System QuantumAI user | **Done** | Seeded / verified AI contact |
| AI panel in chat | **Done** | Streamed responses via `VITE_AI_API_URL` |
| Publish AI replies as sealed messages | **Done** | When applicable in DM/group flows |
| Group AI policy / limits | **Done** | Group settings for enablement, context window, daily limit |
| Server plaintext blind to AI by default | **Done** | E2E; only client-opt-in context may leave the device |

### 3.6 Security & quality

| Requirement | Status | Notes |
|-------------|--------|--------|
| JWT auth on REST + sockets | **Done** | Socket JWT algorithms restricted (`HS256`) |
| Helmet, CORS, rate limits | **Done** | Auth limiter; JSON body size limit |
| Security test suites / CI lanes | **Done** | Crypto, auth, API, transport, canary, nightly |
| Platform admin Console (view users’ chats) | **Out of scope** | No operator inbox; group admins only. Server cannot read E2E content |

### 3.7 Roadmap (not done yet)

| Requirement | Priority | Intent |
|-------------|----------|--------|
| Voice & video calls (WebRTC) | P0 | Complete “real messenger” surface |
| OS / push notifications | P0 | Away-from-tab alerts (beyond sounds/title) |
| Encrypted cloud key vault | P0 | Passphrase-wrapped backup; less reliance on loose `keys.txt` |
| Multi-device sessions (list / revoke) | P0 | Controlled second devices |
| Disappearing messages | P1 | Per-chat / per-message TTL compatible with E2E |
| E2E stories | P1 | Align story media with chat crypto strength |
| Group typing indicators | P1 | Match DM UX |
| 2FA / passkeys | P1 | Harder account takeover |
| Sealed AI context capsules + consent UI | P1 | Cryptographic opt-in of what AI may see |
| Client-only semantic search | P2 | On-device index; server never sees plaintext/embeddings |
| Post-quantum hybrid KEM | P2 | Brand-aligned crypto upgrade path |
| Scheduled messages, stickers, link previews | P3 | Convenience |

---

## 4. Non-negotiable security rules

1. **Private keys never leave the client** (except the user’s own `keys.txt` / future passphrase vault they control).
2. **Message bodies and chat attachments are sealed client-side** before upload/relay.
3. **Login authentication ≠ message unlock** — a valid JWT alone must not decrypt history; the matching key pool is required.
4. **Wrong-account or stale key files must fail import**, not silently load.
5. **Admins / operators must not have a backdoor** to read personal conversations (cryptographically impossible for sealed content).
6. **QuantumAI must not receive silent full-history dumps** from the QuantumChat message store; context is client opt-in.

---

## 5. User experience requirements

### 5.1 Messenger

- Conversation list (left) + active thread (right); mobile sidebar pattern.
- DM header shows peer avatar, name, presence; **click opens public profile**.
- Profile actions: mute, archive, hide, block (where applicable).
- Group header opens group settings when permitted.

### 5.2 First-run & return login

- **Register**: generate 5-key pool, publish public keys, prompt download of `keys.txt`.
- **Every login**: clear that account’s local keyring and show unlock gate:
  - Primary: **Import keys.txt for this account**
  - Secondary: **Generate new set** (warn: old ciphertext stays unreadable)

### 5.3 Settings

Tabs include profile, privacy, security (keys), blocked users, and data (export/delete).

---

## 6. Public vs private profile fields

| Field | Visibility |
|-------|------------|
| Username, display name, bio, avatar | Public (`toPublicJSON` / `GET /users/:id`) |
| Last seen / online | Public only if privacy allows |
| Email, phone, blocked list | Self only |
| Private keys | Device only — never API |

---

## 7. How to run (local)

```powershell
# QuantumChat backend
cd backend
npm install
cp .env.example .env   # set MONGODB_URI, JWT_SECRET, CLIENT_URL
npm run dev            # http://localhost:5000

# QuantumChat frontend
cd frontend
npm install
cp .env.example .env   # VITE_API_URL, optional VITE_AI_API_URL
npm run dev            # http://localhost:5173
```

| App | URL |
|-----|-----|
| Messenger | http://localhost:5173 |
| QuantumChat API + Socket.IO | http://localhost:5000 |
| QuantumAI API *(optional)* | http://localhost:5001 |

Useful backend scripts: `npm test`, `npm run test:security`, `npm run test:e2e-crypto`.

---

## 8. Document history

| Date | Change |
|------|--------|
| 2026-07-19 | Replaced obsolete widget/admin Control Center requirements with current E2E messenger + QuantumAI status and roadmap |

Older references to embeddable widgets, per-site API keys, admin analytics dashboards on `:5174`, or API on `:4000` do **not** apply to this codebase.

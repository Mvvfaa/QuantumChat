# QuantumChat

A MERN messaging app with real end-to-end encryption. Every account gets an X25519 keypair generated in the browser: the **public key** is stored in MongoDB, the **private key** never leaves the device. Messages and file attachments are encrypted client-side with [`tweetnacl`](https://github.com/dchest/tweetnacl-js)'s `nacl.box` (Curve25519-XSalsa20-Poly1305) — the server only ever stores and relays ciphertext.

## How the encryption works

- **Keys are real, not random strings.** `nacl.box.keyPair()` produces a 32-byte public key and a 32-byte private key that are mathematically linked (X25519). Hex-encoded, each is exactly 64 characters.
- **Public keys rotate every 30 minutes** while the app is open. The client generates a fresh keypair, publishes the new public half to the server, and keeps every keypair it has ever held in a local **keyring** (`localStorage`, namespaced per account) — never just the latest one.
- **Every message and attachment stores a snapshot** of both participants' public keys *as of the moment it was sent*. Decrypting a message always uses the exact keypair version that was active then, not whichever key is current now — so rotating keys doesn't make old history unreadable.
- **The server never has a private key.** It can't decrypt anything; it stores ciphertext + nonce and relays it over Socket.IO.
- **Losing local storage means losing history.** If a device's keyring is wiped, there's no way to recover previously received messages — that's inherent to true E2E encryption, not a bug. The app lets you generate a fresh keypair to keep chatting going forward.

## Tech stack

- **Backend**: Node.js, Express, MongoDB/Mongoose, Socket.IO, JWT auth, `helmet` + rate limiting.
- **Frontend**: React 18, Vite, React Router, Axios, Socket.IO client, `tweetnacl`.

## Project structure

```
backend/
  server.js                  # entry point
  src/
    app.js                   # express app, middleware, routes
    config/db.js              # mongoose connection
    models/                   # User, Message, Attachment
    controllers/               # auth, users, messages, attachments
    routes/
    middleware/                # auth (JWT), upload (multer), rateLimiter
    socket/index.js            # authenticated Socket.IO wiring
frontend/
  src/
    crypto/keys.js             # keypair generation + nacl.box encrypt/decrypt
    crypto/keyStorage.js       # local keyring + session storage
    hooks/useKeyRotation.js    # 30-minute rotation loop
    context/AuthContext.jsx    # register/login/rotate/logout
    pages/                     # Register, Login, Chat
    components/                 # UserList, MessageBubble, AttachmentBubble
```

## Setup

### Backend

```bash
cd backend
npm install
cp .env.example .env   # then edit MONGODB_URI / JWT_SECRET
npm run dev             # http://localhost:5000
```

### Frontend

```bash
cd frontend
npm install
cp .env.example .env   # VITE_API_URL, defaults to http://localhost:5000
npm run dev             # http://localhost:5173
```

You'll need a MongoDB instance running (local `mongod`, Docker, or Atlas) — point `MONGODB_URI` at it.

### Environment variables (backend)

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | 5000 | HTTP/Socket.IO port |
| `MONGODB_URI` | — | Mongo connection string |
| `JWT_SECRET` | — | JWT signing secret — set a long random value |
| `JWT_EXPIRES_IN` | 7d | Token lifetime |
| `CLIENT_URL` | http://localhost:5173 | Allowed CORS origin |
| `UPLOAD_DIR` | uploads | Where encrypted attachment blobs are stored on disk |

## API reference

Base URL: `http://localhost:5000/api`. Authenticated routes take `Authorization: Bearer <jwt>`.

### Auth

- `POST /auth/register` — `{ username, email, password, publicKey }` (publicKey: 64-char hex). Returns `{ token, user }`.
- `POST /auth/login` — `{ email, password }`. Updates `lastLoginAt`. Returns `{ token, user }`.
- `GET /auth/me` — current user profile.

### Users

- `GET /users` — everyone except yourself, with each user's current `publicKey` and `lastLoginAt`.
- `GET /users/:id` — a single user's public profile.
- `PATCH /users/me/public-key` — `{ publicKey }`. Publishes a rotated key (used automatically every 30 minutes, and manually to recover a wiped device).

### Messages

- `POST /messages` — `{ to, ciphertext, nonce, senderPublicKey, recipientPublicKey, attachmentId? }`. Broadcasts `message:new` to the recipient over Socket.IO.
- `GET /messages/:userId` — full conversation history with that user, oldest first, with attachment metadata populated.

### Attachments

- `POST /attachments` — `multipart/form-data`: `file` (pre-encrypted ciphertext bytes), `recipientId`, `nonce`, `senderPublicKey`, `recipientPublicKey`. Returns `{ id, filename, mimetype, size }`.
- `GET /attachments/:id/raw` — raw encrypted bytes, only for the sender or recipient. Decryption happens client-side.

### Socket.IO

Connect with `{ auth: { token: <jwt> } }`. Each user joins a room named after their own id; `message:new` events carry the full message document (ciphertext, nonce, key snapshots — decrypt client-side).

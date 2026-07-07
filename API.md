# Quantum Chat API Reference

Base URL: `http://localhost:4000/api/v1`

## Authentication

### Headers

```
Authorization: Bearer <jwt_token>     # Authenticated routes
X-Api-Key: <website_api_key>        # Widget/public routes
X-Website-Id: <website_id>          # Optional tenant hint
```

## Response Format

```json
{
  "success": true,
  "data": { },
  "message": "Optional message",
  "error": "Error description"
}
```

## Paginated Response

```json
{
  "success": true,
  "data": {
    "data": [],
    "page": 1,
    "limit": 20,
    "total": 100,
    "hasMore": true
  }
}
```

## Endpoints

### POST /auth/widget

Authenticate widget user. Requires API key.

```json
{
  "email": "user@example.com",
  "displayName": "John Doe",
  "externalId": "optional-host-id",
  "avatarUrl": "https://..."
}
```

### POST /auth/login

```json
{
  "email": "user@example.com",
  "password": "password",
  "websiteId": "optional"
}
```

### POST /conversations

```json
{
  "participantId": "user_object_id"
}
```

### POST /messages

```json
{
  "conversationId": "conv_id",
  "content": "Hello!",
  "replyTo": "optional_message_id",
  "attachmentIds": ["attachment_id"]
}
```

### PATCH /messages/:id

```json
{
  "content": "Updated message"
}
```

### POST /messages/:id/react

```json
{
  "emoji": "👍"
}
```

### POST /conversations/:id/read

```json
{
  "messageIds": ["optional_specific_ids"]
}
```

### POST /attachments

`multipart/form-data` with field `file`.

Supported types: JPEG, PNG, GIF, WebP, MP4, WebM, PDF, DOC, DOCX, XLS, XLSX, TXT.

Max size: 25MB (configurable per website).

## Socket.IO

Connect with auth token:

```javascript
import { io } from 'socket.io-client';

const socket = io('http://localhost:4000', {
  auth: { token: 'your_jwt' },
});
```

See `packages/shared/src/socket-events.ts` for all event names.

## Roles

| Role | Permissions |
|------|------------|
| `user` | Send/receive messages, manage own conversations |
| `moderator` | Delete any message in tenant |
| `admin` | Manage websites, users, moderate content |
| `super_admin` | Full platform access |

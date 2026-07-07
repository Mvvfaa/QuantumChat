# Quantum Chat — Installation Guide

## Table of Contents

1. [Server Setup](#server-setup)
2. [Register a Website](#register-a-website)
3. [npm Package Integration](#npm-package-integration)
4. [Script Tag Integration](#script-tag-integration)
5. [Host App Token Auth](#host-app-token-auth)
6. [Widget Configuration](#widget-configuration)
7. [Environment Variables](#environment-variables)

---

## Server Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Environment

```bash
cp packages/server/.env.example packages/server/.env
```

Edit `packages/server/.env`:

```env
NODE_ENV=production
PORT=4000
MONGODB_URI=mongodb://your-mongo-host:27017/quantum-chat
JWT_SECRET=generate-a-strong-random-secret
CORS_ORIGINS=https://admin.yourdomain.com
```

### 3. Start MongoDB & Seed

```bash
# Start MongoDB (if local)
mongod

# Seed admin user and demo website
npm run seed -w @quantum-chat/server
```

Save the output — you'll need the **Website ID** and **API Key**.

### 4. Build & Deploy

```bash
npm run build
npm start
```

Deploy behind a reverse proxy (nginx) with HTTPS and WebSocket support:

```nginx
location / {
    proxy_pass http://localhost:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

---

## Register a Website

### Via Admin Dashboard

1. Open http://localhost:5174
2. Login with admin credentials
3. Go to **Websites** → **Add Website**
4. Enter name and domain (e.g. `acme.com`)
5. Click **Verify** after DNS/domain confirmation
6. Copy the **API Key** and **Website ID**

### Via API

```bash
curl -X POST http://localhost:4000/api/v1/admin/websites \
  -H "Authorization: Bearer ADMIN_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme Corp", "domain": "acme.com"}'
```

---

## npm Package Integration

### Install

```bash
npm install @quantum-chat/widget
```

### React Integration

```tsx
// App.tsx
import { useEffect } from 'react';
import { init, destroy } from '@quantum-chat/widget';

function App() {
  useEffect(() => {
    init({
      websiteId: process.env.REACT_APP_QC_WEBSITE_ID!,
      apiKey: process.env.REACT_APP_QC_API_KEY!,
      apiUrl: process.env.REACT_APP_QC_API_URL!,
      user: {
        externalId: currentUser.id,
        email: currentUser.email,
        displayName: currentUser.name,
        avatarUrl: currentUser.avatar,
      },
      theme: {
        primaryColor: '#0A66C2',
        welcomeMessage: 'Need help? Chat with us!',
        position: 'bottom-right',
      },
      onReady: () => console.log('Chat widget ready'),
      onUnreadCount: (count) => {
        document.title = count > 0 ? `(${count}) My App` : 'My App';
      },
    });

    return () => destroy();
  }, []);

  return <YourApp />;
}
```

### Next.js Integration

```tsx
// components/QuantumChat.tsx
'use client';
import { useEffect } from 'react';

export function QuantumChat() {
  useEffect(() => {
    import('@quantum-chat/widget').then(({ init }) => {
      init({
        websiteId: process.env.NEXT_PUBLIC_QC_WEBSITE_ID!,
        apiKey: process.env.NEXT_PUBLIC_QC_API_KEY!,
        apiUrl: process.env.NEXT_PUBLIC_QC_API_URL!,
        user: { /* from your auth context */ },
      });
    });
  }, []);

  return null;
}
```

---

## Script Tag Integration

Add to any HTML page:

```html
<!DOCTYPE html>
<html>
<head>
  <title>My Website</title>
</head>
<body>
  <!-- Your website content -->

  <script src="https://cdn.yourdomain.com/quantum-chat-sdk.umd.js"></script>
  <script>
    const chat = createQuantumChat({
      websiteId: 'YOUR_WEBSITE_ID',
      apiKey: 'YOUR_API_KEY',
      apiUrl: 'https://api.yourdomain.com',
      user: {
        externalId: 'visitor-001',
        email: 'visitor@example.com',
        displayName: 'Website Visitor',
      },
      theme: {
        primaryColor: '#FF5722',
        position: 'bottom-left',
        welcomeMessage: 'Hello! How can we help?',
      },
      onReady: function() {
        console.log('Quantum Chat is ready');
      },
      onUnreadCount: function(count) {
        console.log('Unread messages:', count);
      },
    });

    // Programmatic control
    // chat.open();
    // chat.close();
    // chat.toggle();
    // chat.destroy();
  </script>
</body>
</html>
```

---

## Host App Token Auth

If your application already issues JWTs, pass them directly:

```javascript
init({
  websiteId: 'YOUR_WEBSITE_ID',
  apiKey: 'YOUR_API_KEY',
  token: existingJwtFromYourApp,
});
```

For widget auth without a pre-existing token, provide user info and the widget will authenticate via `/api/v1/auth/widget`:

```javascript
init({
  websiteId: 'YOUR_WEBSITE_ID',
  apiKey: 'YOUR_API_KEY',
  user: {
    externalId: 'user-123',      // Your app's user ID
    email: 'user@company.com',
    displayName: 'Jane Doe',
    avatarUrl: 'https://...',
  },
});
```

---

## Widget Configuration

| Option | Type | Description |
|--------|------|-------------|
| `websiteId` | string | **Required.** Website tenant ID |
| `apiKey` | string | **Required.** Website API key |
| `apiUrl` | string | Backend URL (default: `http://localhost:4000`) |
| `token` | string | Pre-existing JWT token |
| `user` | object | User info for widget auth |
| `theme.primaryColor` | string | Brand primary color |
| `theme.secondaryColor` | string | Background color |
| `theme.accentColor` | string | Accent color |
| `theme.logoUrl` | string | Logo URL |
| `theme.welcomeMessage` | string | Header welcome text |
| `theme.position` | string | `bottom-right`, `bottom-left`, `top-right`, `top-left` |
| `theme.fontFamily` | string | Custom font family |
| `onReady` | function | Called when widget is initialized |
| `onUnreadCount` | function | Called when unread count changes |
| `onMessage` | function | Called on new incoming message |

---

## Environment Variables

### Server (`packages/server/.env`)

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | 4000 | Server port |
| `MONGODB_URI` | localhost | MongoDB connection string |
| `JWT_SECRET` | — | JWT signing secret |
| `JWT_EXPIRES_IN` | 7d | Token expiration |
| `CORS_ORIGINS` | localhost | Allowed CORS origins |
| `UPLOAD_DIR` | ./uploads | File upload directory |
| `MAX_FILE_SIZE_MB` | 25 | Max upload size |
| `RATE_LIMIT_MAX` | 200 | Requests per window |
| `ADMIN_EMAIL` | admin@quantumchat.io | Seed admin email |
| `ADMIN_PASSWORD` | Admin123! | Seed admin password |

### Widget Dev (`packages/widget/.env`)

```env
VITE_API_URL=http://localhost:4000
VITE_API_KEY=your-api-key-from-seed
VITE_WEBSITE_ID=your-website-id-from-seed
```

### Admin (`packages/admin/.env`)

```env
VITE_API_URL=http://localhost:4000
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| CORS errors | Add your domain to the website record and verify it in admin |
| WebSocket fails | Ensure proxy passes `Upgrade` and `Connection` headers |
| Widget not loading | Check API key and website ID in browser console |
| Auth fails | Verify API key header and user email format |
| Upload fails | Check file type/size against website settings |

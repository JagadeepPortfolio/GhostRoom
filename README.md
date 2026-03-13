# GhostRoom

Ephemeral group chat. No login. No history. Messages burn 10 seconds after reading.

## Architecture

```
ghostroom/
  frontend/    Next.js (App Router) + Tailwind + Framer Motion
  backend/     Express + Socket.io
```

- **In-memory only** — messages live in a `Map()` on the server. Server restart = everything gone.
- **Per-user burn** — each user gets an independent 10s countdown after reading a message.
- **No database, no auth** — just `localStorage` for display names.

## Setup

### Prerequisites
- Node.js 18+
- npm

### Install

```bash
cd backend && npm install
cd ../frontend && npm install
```

### Run locally

Terminal 1 (backend):
```bash
cd backend
npm run dev
```

Terminal 2 (frontend):
```bash
cd frontend
npm run dev
```

- Frontend: http://localhost:3000
- Backend: http://localhost:3001

### Environment Variables

**Frontend** (`frontend/.env.local`):
```
NEXT_PUBLIC_SERVER_URL=http://localhost:3001
```

**Backend** (`backend/.env`):
```
PORT=3001
CORS_ORIGIN=http://localhost:3000
```

## Deployment

### Frontend (Vercel)
1. Connect the `frontend/` directory to Vercel.
2. Set `NEXT_PUBLIC_SERVER_URL` to your backend URL.

### Backend (Fly.io / Railway)
1. Deploy the `backend/` directory.
2. Set `PORT` and `CORS_ORIGIN` (your Vercel frontend URL).
3. Run `npm run build && npm start`.

## Socket Events

| Direction | Event | Description |
|-----------|-------|-------------|
| C -> S | `create_room` | Create a new room |
| C -> S | `join_room` | Join with username |
| C -> S | `send_message` | Send a message |
| C -> S | `message_read` | Mark message as read |
| C -> S | `typing_start/stop` | Typing indicators |
| S -> C | `new_message` | Broadcast message |
| S -> C | `message_read_ack` | Updated readBy |
| S -> C | `room_users` | Online user list |
| S -> C | `rate_limited` | Rate limit warning |

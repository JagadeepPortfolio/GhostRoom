# Product Requirements Document: GhostRoom

## 1. Vision
A browser-based "digital darkroom" for friends. Messages are ephemeral and self-destructing. Friction is zero.

## 2. User Journey
1. **Landing:** User visits `/`. Sees a "Create Room" button.
2. **Room Created:** A random room ID is generated. User gets a shareable link (e.g., `/room/xv-392-p`).
3. **Identity:** On first visit, a sleek modal prompts "Enter your name". Name saved to `localStorage`. Username must be unique per room — duplicates are blocked.
4. **Presence:** User enters the room and sees who else is online (Socket-based presence).
5. **Chatting:** User sends a message. It appears for everyone in the room.
6. **The Burn (Per-User):**
   - When a user sees a message, a `message_read` event fires.
   - A 10s countdown starts **only for that user** — other users still see the message.
   - When 10s hits, the message "burns" with a fire/smoke animation and is removed from that user's local state.
   - Server clears the message from the `Map` once ALL current room users have read it.
   - If a user disconnects before reading, they are removed from the "required readers" count.

## 3. Core Features
- **Landing Page:** "Create Room" button generates random room ID, shows shareable link.
- **Ephemeral Rooms:** Random string IDs (e.g., `ghostroom.app/room/xv-392-p`).
- **Realtime Sync:** Socket.io for messages, typing indicators, and presence.
- **Per-User Burn:** Independent 10s countdown per reader. Bomb icon with countdown (10...9...8).
- **Burn Animation:** Framer Motion fire/smoke exit animations (~1s duration).
- **@Mentions:** Typing `@` shows a user suggestion popup. Mentions are visual-only (highlight style).
- **Typing Indicator:** "Raj is typing..." disappears after 3s of inactivity.
- **Anti-Spam:** 5 messages per 10 seconds per user. Exceeded = "Slow down." warning.
- **User Presence:** Online user list in the header.

## 4. Explicitly NOT Supported
- No replies, quotes, threading, or message history search.
- No persistent storage or database.
- No login, JWTs, or cookies.

## 5. Message Structure
```json
{
  "id": "uuid",
  "sender": "string",
  "text": "string",
  "roomId": "string",
  "createdAt": "timestamp",
  "readBy": { "Jagadeep": 1710000000 }
}
```

## 6. Technical Constraints
- **Zero Storage:** Messages exist in a `Map()` on the Express server.
- **No History:** Joining a room late means you see NOTHING of the past.
- **Stateless Auth:** Only `localStorage` for names. No JWTs, no cookies.

## 7. UI Design
- **Theme:** Glassmorphism (Tailwind + Framer Motion).
- **Layout:** Header (room name + online users) | Chat area (message bubbles) | Footer (input).
- **Message bubble:** Sender name, text, mention highlights, bomb countdown.

## 8. Deployment
- **Frontend:** Vercel-ready Next.js app.
- **Backend:** Fly.io / Railway-ready Express server.
- **Env vars:** `NEXT_PUBLIC_SERVER_URL` for the backend URL.

## 9. Success Metrics
- Average time from URL click to first message: < 5 seconds.
- 0% persistence of messages on server restart.

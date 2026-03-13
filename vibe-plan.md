# GhostRoom — Vibe Plan

## Architecture Overview

```
ghostroom/
  frontend/          # Next.js (App Router)
    src/
      app/
        page.tsx                  # Landing page — "Create Room" button
        room/
          [id]/
            page.tsx              # Chat room page
      components/
        username-modal.tsx        # First-visit name prompt
        chat-area.tsx             # Message list with scroll management
        message-bubble.tsx        # Single message: sender, text, countdown, burn
        message-input.tsx         # Input bar with @mention trigger
        mention-popup.tsx         # @mention user suggestion list
        typing-indicator.tsx      # "Raj is typing..."
        online-users.tsx          # Sidebar/header user list
        burn-animation.tsx        # Framer Motion fire/smoke exit
      hooks/
        use-socket.ts             # Socket.io client connection + event handlers
        use-burn-timer.ts         # Per-message 10s countdown logic
      lib/
        sanitize.ts               # XSS sanitization utility
        room-id.ts                # Random room ID generator
      types/
        index.ts                  # Shared TypeScript types (Message, User, etc.)
    .env.local                    # NEXT_PUBLIC_SERVER_URL=http://localhost:3001
    package.json
    tailwind.config.ts

  backend/
    src/
      index.ts                    # Express + Socket.io server entry
      socket/
        handlers.ts               # All socket event handlers
        room-store.ts             # In-memory Map for rooms + messages
        rate-limiter.ts           # 5 msgs / 10s per user
    .env                          # PORT=3001, CORS_ORIGIN
    package.json
```

## Socket.io Event Map

### Client -> Server
| Event              | Payload                              | Description                    |
|--------------------|--------------------------------------|--------------------------------|
| `join_room`        | `{ roomId, username }`               | Join a room                    |
| `leave_room`       | `{ roomId, username }`               | Leave a room                   |
| `send_message`     | `{ roomId, sender, text }`           | Send a message                 |
| `message_read`     | `{ roomId, messageId, username }`    | Mark message as read           |
| `typing_start`     | `{ roomId, username }`               | User started typing            |
| `typing_stop`      | `{ roomId, username }`               | User stopped typing            |
| `check_username`   | `{ roomId, username }`               | Check if username is available |

### Server -> Client
| Event              | Payload                              | Description                    |
|--------------------|--------------------------------------|--------------------------------|
| `room_users`       | `string[]`                           | Updated online user list       |
| `new_message`      | `Message`                            | Broadcast new message          |
| `message_read_ack` | `{ messageId, readBy }`              | Updated readBy for a message   |
| `user_typing`      | `{ username }`                       | Someone is typing              |
| `user_stop_typing` | `{ username }`                       | Someone stopped typing         |
| `username_taken`   | `{ available: boolean }`             | Username check result          |
| `rate_limited`     | `{}`                                 | User hit rate limit            |
| `user_joined`      | `{ username }`                       | Someone joined the room        |
| `user_left`        | `{ username }`                       | Someone left the room          |

## Build Phases

### Phase 1: Backend Core
1. Express server + Socket.io setup with CORS
2. In-memory room store (`Map<roomId, { users: Set, messages: Map }>`)
3. Socket handlers: `join_room`, `leave_room`, `send_message`, `message_read`
4. Username uniqueness check per room
5. Rate limiter (5 msgs / 10s)
6. Burn rule: delete message from Map when `readBy` keys >= current room users
7. Disconnect cleanup: remove user from room, update required readers count

### Phase 2: Frontend Shell
1. Next.js App Router setup + Tailwind + Framer Motion
2. Landing page (`/`) — "Create Room" button, generates random ID, shows link
3. Room page (`/room/[id]`) — layout skeleton (header, chat, input)
4. Username modal — prompt on first visit, save to localStorage
5. Socket.io client hook (`use-socket.ts`)

### Phase 3: Chat UI
1. Message input with send functionality
2. Message bubbles (sender, text, timestamp)
3. Chat area with auto-scroll
4. Online users display in header

### Phase 4: Burn Mechanics
1. `message_read` event emission on message visibility
2. Per-user 10s countdown timer (`use-burn-timer.ts`)
3. Countdown display on bubbles (bomb icon + seconds)
4. Framer Motion exit animation (fire/smoke)
5. Remove message from local state after burn

### Phase 5: Polish
1. @mention popup — trigger on `@`, show online users, insert on click
2. Mention highlighting in message text
3. Typing indicator (3s debounce)
4. Glassmorphism theme (backdrop-blur, semi-transparent cards)
5. Smooth scroll handling on message add/remove
6. XSS sanitization

### Phase 6: Deployment Prep
1. Environment variable setup
2. README.md with setup/deploy instructions
3. Final testing

## Key Design Decisions
- **Per-user burn is client-driven:** Server tracks `readBy` but the 10s timer runs on each client independently. No server-side timers needed.
- **No auto-create rooms:** Only the landing page creates rooms. Visiting a non-existent `/room/[id]` shows "Room not found".
- **Disconnect = remove from readers:** When a user disconnects, they're removed from the room's user set. Existing unread messages recalculate burn eligibility against remaining users only.
- **Sender auto-reads:** The sender's own messages are auto-marked as read (they obviously saw it).

---
*Status: AWAITING "VIBE ON"*

# GhostRoom Project Constitution

## High-Level Context
- **Role:** You are a Senior Full-Stack Engineer specializing in Real-Time systems.
- **Goal:** Build "GhostRoom" — an ephemeral, no-login, WebSocket-based group chat.
- **Stack:** Next.js (Frontend), Express/Node (Backend), Socket.io (Realtime).

## Vibe-Coding Guardrails (THE BOTTLENECK FIX)
1. **MEMORY ONLY:** NEVER suggest Prisma, Mongoose, or any Database. If the server restarts, everything dies. This is a feature, not a bug.
2. **THE BURN RULE:** Messages burn **per-user** — each user's 10s countdown is independent. Server deletes from `Map` only once ALL current room users have read it. If a user disconnects unread, they are removed from the "required readers" count.
3. **PLANNING:** Before writing code for a feature, create a `vibe-plan.md`. Do not start coding until I say "VIBE ON".
4. **COMPACTION:** Every 15-20 messages, run a summary of the current "mental model" of the WebSocket events to prevent logic drift.

## Room Rules
- **Landing page** at `/` with a "Create Room" button generates a random ID and shows the shareable link.
- Visiting `/room/[id]` directly does NOT auto-create — must be created from landing page.
- **Username uniqueness** is enforced per room. Duplicate names are blocked with a warning.

## Message Structure
```json
{
  "id": "uuid",
  "sender": "string",
  "text": "string",
  "roomId": "string",
  "createdAt": "timestamp",
  "readBy": { "username": "timestamp" }
}
```

## Features Checklist
- [x] Ephemeral rooms via landing page
- [x] Per-user burn countdown (10s)
- [x] @mentions with user suggestion popup
- [x] Typing indicator (3s timeout)
- [x] Rate limiting (5 msgs / 10s per user)
- [x] User presence (online list)
- [x] Burn animation (fire/smoke via Framer Motion)
- [x] Countdown indicator on message bubbles
- [ ] NO replies, quotes, threading, or history search

## Critical Commands
- **Install:** `npm install` in both `/frontend` and `/backend`.
- **Dev:** `npm run dev` (Frontend: 3000, Backend: 3001).
- **Test:** `npm test`.

## Technical Standards
- **Naming:** `kebab-case` for files. `PascalCase` for Components.
- **UI:** Tailwind + Framer Motion. Use "Glassmorphism" for the theme.
- **Safety:** Always sanitize `text` inputs with a basic regex to prevent XSS.
- **Deployment:** Frontend on Vercel, Backend on Fly.io/Railway. Use env vars for server URL.

---
*Last Updated: 2026-03-13 | Current Mood: Minimalist & Private.*

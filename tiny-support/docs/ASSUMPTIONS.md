# Tiny Support — Assumptions

Assumptions made during MVP development. These are documented here for review.

## Identity & Auth
- **PID is self-reported**: Customers enter their own PID. There is no server-side verification that the PID maps to a real account. In production, this would integrate with the company's user database.
- **Agent auth is a simple password gate**: A single shared password via `AGENT_PASSWORD` env var. No per-agent accounts, sessions, or tokens. Sufficient for MVP/internal testing.
- **No session persistence**: If the browser refreshes during a call, the user must rejoin. The WebRTC connection and Socket.IO session are not recoverable.

## Rooms
- **Rooms are ephemeral**: In-memory only. Server restart clears all rooms. No room history or audit trail.
- **Room codes are random**: 6-character alphanumeric codes generated server-side. No collision handling beyond a simple retry loop.
- **Rooms don't auto-expire**: A waiting room with no agent will persist indefinitely until the server restarts. Production would add TTLs.
- **One agent per room is typical**: The system allows multiple agents (up to the 4-person cap), but the UI doesn't have special multi-agent features.

## WebRTC
- **STUN only**: Using Google's public STUN servers. No TURN server configured, which means the app may not work behind symmetric NATs or strict firewalls. Production would add a TURN server.
- **Mesh topology**: Every participant connects to every other participant. Fine for ≤4 people but does not scale beyond that.
- **Screen share detection by label**: Remote screen shares are identified by checking video track labels for keywords like "screen", "window", "monitor". This heuristic works in Chrome/Firefox/Safari but may not be 100% reliable.
- **No simulcast or bandwidth adaptation**: All tracks are sent at their native resolution/framerate. No quality adaptation based on network conditions.

## UI/UX
- **Desktop-first**: The layout is optimized for desktop/laptop screens. Mobile is usable but not optimized (no responsive breakpoints for the in-call view).
- **No dark mode**: Single light theme only.
- **Customer role is default**: Anyone joining via `/join/:roomId` is assigned the `customer` role. Only the agent queue flow assigns the `agent` role.
- **Guest role assigned by URL convention**: The spec mentions guests, but currently all `/join/:roomId` users are customers. To add a guest, the agent would share the join link and the guest would join as a customer. A future enhancement could add a `?role=guest` URL param.

## Chat
- **Messages are not persisted**: Chat messages exist only in-memory during the session. They're lost when the room ends.
- **No rich text or markdown**: Plain text only. URLs are not auto-linked.
- **No file attachments**: Text-only chat.

## Privacy
- **PID is not logged server-side**: The server console logs display names and roles but omits PIDs.
- **PID visibility is role-based**: Agents see all PIDs. Non-agents see only their own. This is enforced server-side via `sanitizeRoomForRole()`.
- **No encryption beyond WebRTC defaults**: WebRTC media is encrypted via DTLS-SRTP by default. Signaling is over plain WebSocket (not WSS) in dev mode.

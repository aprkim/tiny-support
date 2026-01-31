# Tiny Support — Architecture

## System Overview

```
┌─────────────────────────────────────────────────────┐
│                   Browser (Client)                   │
│                                                      │
│  Next.js App (React)                                 │
│  ├── Pages (App Router)                              │
│  ├── useRoom hook (state management)                 │
│  ├── Socket.IO client (signaling)                    │
│  └── WebRTC (RTCPeerConnection mesh)                 │
│                                                      │
└───────────────────────┬─────────────────────────────┘
                        │ Socket.IO (WebSocket)
                        │ path: /api/socketio
┌───────────────────────┴─────────────────────────────┐
│                   Server (Node.js)                   │
│                                                      │
│  Custom HTTP server                                  │
│  ├── Next.js request handler                         │
│  ├── Socket.IO server (signaling)                    │
│  └── In-memory Room Store                            │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## WebRTC Topology: Mesh

Since max participants is 4, we use a full mesh topology:
- Each participant maintains a direct RTCPeerConnection to every other participant
- No SFU/MCU needed at this scale
- Each peer sends its camera + optional screen share tracks to all other peers

```
    A ←──→ B
    ↕ ╲  ╱ ↕
    ↕  ╲╱  ↕
    ↕  ╱╲  ↕
    ↕ ╱  ╲ ↕
    C ←──→ D
```

## Signaling Flow

1. **Participant A joins room** → server broadcasts `participant-joined` to existing members
2. **Existing members** each initiate a WebRTC connection to A:
   - Create `RTCPeerConnection`
   - Add local tracks (camera, screen)
   - Create SDP offer → send via Socket.IO
3. **A receives offers** → creates answers → sends back via Socket.IO
4. **ICE candidates** exchanged via Socket.IO until connection established

### Socket.IO Events

| Event | Direction | Purpose |
|-------|-----------|---------|
| `create-room` | client → server | Create a new room |
| `check-room` | client → server | Check if room exists |
| `join-room` | client → server | Join a room with identity |
| `leave-room` | client → server | Leave current room |
| `end-room` | client → server | End the session |
| `offer` | client → server → client | SDP offer relay |
| `answer` | client → server → client | SDP answer relay |
| `ice-candidate` | client → server → client | ICE candidate relay |
| `media-state` | client → server | Update audio/video/screen state |
| `chat-message` | client → server → clients | Chat message broadcast |
| `room-state` | server → client | Full room state update |
| `participant-joined` | server → clients | New participant notification |
| `participant-left` | server → clients | Participant left notification |
| `room-ended` | server → clients | Session ended notification |
| `agent-auth` | client → server | Agent password verification |
| `get-waiting-rooms` | client → server | Fetch waiting room queue |

## Room Store

In-memory `Map<string, Room>` on the server. No persistence — rooms are lost on server restart. This is acceptable for MVP.

### Room State Machine

```
  [waiting] ──(agent joins)──→ [active] ──(end/empty)──→ [ended]
```

## Media Handling

### Camera/Mic
- Acquired via `getUserMedia({ audio: true, video: true })`
- Tracks added to all peer connections
- Toggling = enabling/disabling tracks (not removing them)

### Screen Share
- Acquired via `getDisplayMedia({ video: true, audio: true })`
- Added as additional tracks to all peer connections
- Requires renegotiation (new SDP offer/answer exchange)
- Detected on remote side by track label heuristics (screen/window/monitor)
- Browser's native "Stop sharing" button triggers `track.onended`

## Key Design Decisions

1. **Custom server** — Socket.IO needs a persistent WebSocket server; Next.js API routes don't support this natively
2. **Mesh over SFU** — Simpler implementation for ≤4 participants; no media server needed
3. **Session storage for join info** — Passes display name/PID/role from join page to room page without URL params
4. **Role-based PID filtering** — Server sanitizes room state per-viewer before sending

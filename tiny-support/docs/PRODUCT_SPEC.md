# Tiny Support — Product Spec

## Overview
Tiny Support is a video chat customer support tool optimized for screen sharing. Companies use it to provide live support to users (often vibecoders) who need to share their dev environment with a support agent.

## Roles

| Role | Description | Access |
|------|-------------|--------|
| **customer** | End-user requesting support | Joins via `/join/:roomId` link |
| **agent** | Support agent | Joins via `/agent/queue` dashboard |
| **guest** | Additional participant invited by agent | Joins via invite link |

## User Flows

### Customer Flow
1. Customer receives a room link or creates a new room from `/`
2. Customer is taken to `/join/:roomId` — enters display name + PID
3. Customer enters the waiting room (mic/cam toggles, "Waiting for agent...")
4. Agent joins → room status changes to `active` → in-call UI appears
5. Customer can share screen, use chat, toggle media
6. Session ends when agent ends the call or both leave

### Agent Flow
1. Agent navigates to `/agent/queue`
2. Agent enters password (env var gate)
3. Agent enters their name (and optional agent PID)
4. Agent sees waiting rooms listed with customer name, PID, wait time
5. Agent clicks "Join" → enters the room as `agent` role
6. Agent sees customer's video/screen share on main stage
7. Agent can copy invite link to bring in additional participants
8. Agent can end the session

### Guest Flow
1. Guest receives an invite link from agent: `/join/:roomId`
2. Guest enters display name and PID
3. Guest joins the active call

## Room Lifecycle

```
Created (waiting) → Agent joins (active) → Session ends (ended)
```

- **waiting**: Customer is in the room, no agent yet
- **active**: At least one agent has joined
- **ended**: Room ended by agent or all participants left

## Screen Share Priority
1. Any active screen share becomes the main stage automatically
2. If multiple shares are active, agent can select which one to stage
3. When sharing stops, main stage reverts to customer camera
4. Active sharer sees a persistent "You are sharing your screen" indicator with a stop button

## PID Privacy
- Agents see all participant PIDs
- Customers see only their own PID
- Guests cannot see customer PIDs (only their own)
- PIDs are not logged in server console

## Participant Limits
- Hard cap: 4 participants per room
- If room is full, new joiners see a "Room is full" screen

## Chat
- In-call text chat panel available under the "Chat" tab
- Messages show sender name, role badge, and timestamp
- Useful for sharing links, code snippets, short messages

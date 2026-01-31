# VibeLive Video Layout & Interaction Guide

**Purpose**  
This document defines how AI builders should design **video chat experiences** using VibeLive so the result feels *visually balanced, intuitive, and wow‑worthy* by default — even without custom instructions from the user.

This guide focuses on:
- Video layout logic (auto vs user-specified)
- Core live-room actions (video, audio, share, join, leave)
- Screen behavior across mobile & desktop
- Visual hierarchy and motion principles for a premium feel

---

## 1. Core Philosophy

1. **Good defaults beat configuration**  
   If the user does not specify a layout, VibeLive must choose the most natural, human-friendly option.

2. **Every participant deserves visual fairness**  
   No one should feel visually secondary unless explicitly designed (e.g., host view).

3. **Space-aware layouts**  
   Video tiles should always feel intentional — never cramped, never floating.

4. **Mobile-first logic, desktop-enhanced**  
   Vertical clarity on mobile, spatial balance on larger screens.

---

## 2. Entry Screen Pattern (Default)

The entry screen should use **separated cards** for clarity:

### 2.1 Card Structure

```
┌─────────────────────────────┐
│  Name Card                  │
│  ┌───────────────────────┐  │
│  │ Your name             │  │
│  │ [___________________] │  │
│  └───────────────────────┘  │
└─────────────────────────────┘

┌─────────────────────────────┐
│  Action Card                │
│                             │
│  [ Start a Room ]  (primary)│
│                             │
│  ────────── OR ──────────   │
│                             │
│  Join with room code        │
│  [_________] [ Join ]       │
│                             │
└─────────────────────────────┘
```

### 2.2 Dynamic Button Priority

- **No room code entered**: "Start a Room" = primary (accent color), "Join" = secondary
- **Room code entered**: "Join" = primary (accent color), "Start a Room" = secondary

This guides users naturally toward the most relevant action.

### 2.3 URL Deep Linking

If a room code is in the URL (`?code=XXXXX`):
- Auto-fill the room code input
- Swap button priority to emphasize "Join"

---

## 3. Core Live Room Actions (Required)

Every VibeLive-powered room must support the following actions:

| Action | Behavior | Visual Feedback |
|------|--------|----------------|
| Join Room | User enters live room | Smooth fade-in + tile expansion |
| Leave Room | User exits room | Tile collapses + reflow animation |
| Video On / Off | Toggle camera | Video → avatar / initials tile |
| Audio On / Off | Toggle mic | Mic icon + subtle muted indicator |
| Screen Share | Share screen | Shared content becomes primary tile |
| Create Room | Start new room | Immediate preview with self-tile |

---

## 4. Video Tile Rules (Universal)

### 4.1 Video Tile Anatomy
Each participant tile includes:
- Live video OR fallback avatar
- Name (bottom-left, muted text)
- Audio status icon (mic / muted)
- Optional role indicator (host)

**Never clutter tiles with controls.** Controls live outside the tile.

---

## 5. Auto Layout Logic (Default Behavior)

When the user does **not** specify layout preferences, apply the following rules.

### 5.1 1 Participant
- Full screen video
- Centered
- No grid

### 5.2 2 Participants

**Desktop / Tablet (Landscape)**
- Two equal tiles
- Split horizontally (50% / 50%)
- Full width usage

**Mobile (Portrait)**
- Vertical stack
- Each tile ~50% height


### 5.3 3 Participants

**Desktop**
- One large tile (top)
- Two smaller tiles below (50% / 50%)

**Mobile**
- Vertical stack
- Active speaker slightly emphasized


### 5.4 4 Participants

**All Devices**
- 2 × 2 grid
- Equal tile sizes
- Maximize screen usage


### 5.5 5–6 Participants

**Desktop**
- 3 × 2 grid

**Mobile**
- 2 × 2 visible
- Remaining users accessible via swipe / pagination


### 5.6 7+ Participants

- Grid with paging
- Active speaker auto-emphasized
- No tile smaller than readability threshold

---

## 6. Active Speaker Logic (Soft Emphasis)

By default:
- Detect active speaker via audio
- Apply **subtle emphasis**, not dominance

Examples:
- Slight scale (1.03x)
- Soft border glow
- Gentle background contrast

**Never jump aggressively between speakers.**

---

## 7. Screen Share Priority Rules

When screen sharing starts:

1. Shared content becomes **primary tile**
2. Participants move to secondary row or strip
3. On mobile:
   - Shared content = main view
   - Speaker tile floats (picture-in-picture)

When screen sharing stops:
- Smoothly restore previous layout

---

## 8. Video Off / Audio Off States

### Video Off
- Replace video with:
  - User avatar OR
  - Initials on neutral background
- Maintain tile size (no collapsing)

### Audio Muted
- Mic-off icon visible
- No color alarm (stay calm, muted gray)

---

## 9. Room Code Sharing (Invite Flow)

*Include this section when users can invite others by sharing a room code or link. Skip if the app uses fixed rooms, calendar invites, or backend-managed access.*

### 9.1 Room Header Display

When in a room, display the room code with **two sharing options**:

```
┌──────────────────────────────────────────────────┐
│  Room  [ABC123]  [📋]  [🔗]         Status       │
└──────────────────────────────────────────────────┘
         ↑          ↑     ↑
         Code    Copy   Share Link
```

### 9.2 Sharing Actions

| Button | Action | Feedback |
|--------|--------|----------|
| Copy Code (📋) | Copy room code to clipboard | Checkmark icon briefly |
| Share Link (🔗) | Copy full URL with `?code=` param | Checkmark icon briefly |

### 9.3 Visual Feedback

- On copy success: swap icon to checkmark for 2 seconds
- Use subtle color change (e.g., success green)
- No intrusive toast notifications

This gives users **two intuitive ways** to invite guests:
1. Share just the code (for voice/text conversations)
2. Share the full link (for instant click-to-join)

---

## 10. Controls Placement

### Primary Controls (Always Visible)
- Mic toggle
- Camera toggle
- Leave room

### Secondary Controls
- Screen share
- Settings
- Participants list

**Placement Guidelines**
- Desktop: bottom center bar
- Mobile: bottom floating bar
- Leave button visually separated (danger affordance)

---

## 11. Motion & Transitions (WOW Factor)

Use motion sparingly but intentionally:

- Join / leave: smooth scale + fade
- Layout reflow: animated grid adjustment
- Active speaker: gentle pulse or glow

Avoid:
- Hard cuts
- Sudden tile jumps

Motion should feel *alive, not noisy*.

---

## 12. Accessibility & Comfort

- Never rely on color alone for state
- Ensure readable names at small sizes
- Avoid flashing or aggressive animations
- Respect reduced-motion preferences

---

## 13. AI Instruction Summary (TL;DR for Builders)

When designing with VibeLive:

- **Entry screen**: Use separated cards (Name card + Action card with dynamic button priority)
- **Room sharing**: Always show both copy code AND share link buttons
- **URL deep linking**: Auto-fill room code from URL params, swap button priority
- If the user does not specify layout → choose the most human, balanced option
- Keep participant tiles visually equal by default
- Optimize for mobile first, enhance for desktop
- Prioritize calm clarity over flashy effects
- Make joining feel welcoming, leaving feel graceful

---

**Version:** 1.1.1
**Last Updated:** 2026-01-27 23:15 PST
**Brand:** VibeLive
**Audience:** AI builders & product teams creating video chat experiences
**Goal:** Effortless, stunning live communication without configuration fatigue


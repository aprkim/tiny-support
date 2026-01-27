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

## 2. Core Live Room Actions (Required)

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

## 3. Video Tile Rules (Universal)

### 3.1 Video Tile Anatomy
Each participant tile includes:
- Live video OR fallback avatar
- Name (bottom-left, muted text)
- Audio status icon (mic / muted)
- Optional role indicator (host)

**Never clutter tiles with controls.** Controls live outside the tile.

---

## 4. Auto Layout Logic (Default Behavior)

When the user does **not** specify layout preferences, apply the following rules.

### 4.1 1 Participant
- Full screen video
- Centered
- No grid

### 4.2 2 Participants

**Desktop / Tablet (Landscape)**
- Two equal tiles
- Split horizontally (50% / 50%)
- Full width usage

**Mobile (Portrait)**
- Vertical stack
- Each tile ~50% height


### 4.3 3 Participants

**Desktop**
- One large tile (top)
- Two smaller tiles below (50% / 50%)

**Mobile**
- Vertical stack
- Active speaker slightly emphasized


### 4.4 4 Participants

**All Devices**
- 2 × 2 grid
- Equal tile sizes
- Maximize screen usage


### 4.5 5–6 Participants

**Desktop**
- 3 × 2 grid

**Mobile**
- 2 × 2 visible
- Remaining users accessible via swipe / pagination


### 4.6 7+ Participants

- Grid with paging
- Active speaker auto-emphasized
- No tile smaller than readability threshold

---

## 5. Active Speaker Logic (Soft Emphasis)

By default:
- Detect active speaker via audio
- Apply **subtle emphasis**, not dominance

Examples:
- Slight scale (1.03x)
- Soft border glow
- Gentle background contrast

**Never jump aggressively between speakers.**

---

## 6. Screen Share Priority Rules

When screen sharing starts:

1. Shared content becomes **primary tile**
2. Participants move to secondary row or strip
3. On mobile:
   - Shared content = main view
   - Speaker tile floats (picture-in-picture)

When screen sharing stops:
- Smoothly restore previous layout

---

## 7. Video Off / Audio Off States

### Video Off
- Replace video with:
  - User avatar OR
  - Initials on neutral background
- Maintain tile size (no collapsing)

### Audio Muted
- Mic-off icon visible
- No color alarm (stay calm, muted gray)

---

## 8. Controls Placement

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

## 9. Motion & Transitions (WOW Factor)

Use motion sparingly but intentionally:

- Join / leave: smooth scale + fade
- Layout reflow: animated grid adjustment
- Active speaker: gentle pulse or glow

Avoid:
- Hard cuts
- Sudden tile jumps

Motion should feel *alive, not noisy*.

---

## 10. Accessibility & Comfort

- Never rely on color alone for state
- Ensure readable names at small sizes
- Avoid flashing or aggressive animations
- Respect reduced-motion preferences

---

## 11. AI Instruction Summary (TL;DR for Builders)

When designing with VibeLive:

- If the user does not specify layout → choose the most human, balanced option
- Keep participant tiles visually equal by default
- Optimize for mobile first, enhance for desktop
- Prioritize calm clarity over flashy effects
- Make joining feel welcoming, leaving feel graceful

---

**Version:** 1.0.0  
**Brand:** VibeLive  
**Audience:** AI builders & product teams creating video chat experiences  
**Goal:** Effortless, stunning live communication without configuration fatigue


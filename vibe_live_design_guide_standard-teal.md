# VibeLive Design Guide (Standard)

**Theme / Brand color:** `#0EA5A4` (VibeLive Teal)  
**Style:** modern, minimal, calm surfaces + teal accents  
**Default radius:** `16px`  
**Max content width:** `980px`  
**Font stack:** `ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial`

**Version:** 1.0.0  
**Last Updated (PST):** 2026-01-27 14:15 PST

---

## 1) Core Principles
1. **Neutral first, accent second:** most UI is gray/white; teal is for actions + meaning.
2. **One primary action per screen:** only one primary (teal) button in a view when possible.
3. **Soft feedback:** hover/selected states use subtle tints, not loud colors.
4. **Readable hierarchy:** clear headline/body/muted layers.

---

## 2) Essential Color Tokens (VibeLive Standard)
Use **only these** unless you have a special reason.

### Surfaces
- `--bg`: `#f7f9f8` (main background)
- `--card`: `#ffffff` (cards, panels, modals)
- `--soft`: `#f0f4f3` (hover surface, subtle fill)
- `--border`: `#e2e8f0` (dividers, outlines)
- `--borderSoft`: `#dbe2ea` (hover/emphasis borders)

### Text
- `--text`: `#1e293b` (primary text)
- `--muted`: `#64748b` (secondary text, labels)
- `--muted2`: `#9ca3af` (tertiary / helper text)

### Brand Accent (Primary)
- `--accent`: `#0EA5A4` (primary buttons, key links)
- `--accentHover`: `#0d9488` (hover)
- `--accentSoft`: `rgba(14,165,164,.10)` (selected background tint)
- `--accentBorder`: `rgba(14,165,164,.35)` (focus/outline)

### Status (Semantic)
**Live / Online / Enabled** (reserved for presence)
- `--liveBg`: `#e8f4ed`
- `--liveBorder`: `#a6c5a3`
- `--liveText`: `#324252`

**Disabled / Offline**
- `--disabledBg`: `#f3f4f6`
- `--disabledBorder`: `#e5e7eb`
- `--disabledText`: `#6b7280`

---

## 3) Components Rules (Practical + Repeatable)

### Buttons
**Primary (CTA)**
- Background: `--accent`
- Text: `#ffffff`
- Hover: `--accentHover`
- Use for: “Join Room”, “Create Room”, “Start Session”, “Save”, “Confirm”

**Secondary**
- Background: `--soft`
- Text: `--text`
- Border: `--border`
- Use for: “Cancel”, “Back”, “Close”

**Danger (Rare)**
- Background: `#FEF2F2`
- Border: `#fca5a5`
- Text: `#b91c1c`
- Use for: destructive confirmations like “End session”, “Remove participant”

### Links
- Default: `--accent`
- Hover: underline (don’t change color dramatically)

### Inputs
- Background: `--card`
- Border: `--border`
- Focus ring: `--accentBorder` (soft teal glow)
- Error state: use the Danger colors above

### Cards
- Background: `--card`
- Border: `1px solid --border`
- Hover: border becomes `--borderSoft` OR add a very soft shadow

### Pills / Badges
Keep it **semantic**, not decorative:
- **Live**: `--liveBg / --liveBorder / --liveText`
- **Primary tag**: `--accentSoft` background + `--accent` text
- **Disabled**: `--disabledBg / --disabledBorder / --disabledText`

---

## 4) Modals, Toasts, Alerts (One Standard Pattern)
**Overlay/backdrop:** `rgba(30, 41, 59, 0.5)`  
**Container:** `--card` with `--border`, radius `16px`

### Alert types
- **Success / Live:** use Live palette
- **Error:** use Danger palette
- **Info:** `--soft` background + `--border` + `--text`

---

## 5) Iconography
- No emojis in UI.
- Use simple stroke icons (Lucide style).
- Sizes: 20 / 24
- Default icon color: `currentColor` (inherits text color)
- Use `--muted` for inactive icons, `--accent` for active icons.

---

## 6) Copy-Paste Tokens (CSS Variables)
```css
:root{
  /* layout */
  --radius: 16px;
  --max: 980px;
  --sans: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;

  /* surfaces */
  --bg: #f7f9f8;
  --card: #ffffff;
  --soft: #f0f4f3;
  --border: #e2e8f0;
  --borderSoft: #dbe2ea;

  /* text */
  --text: #1e293b;
  --muted: #64748b;
  --muted2: #9ca3af;

  /* brand */
  --accent: #0EA5A4;
  --accentHover: #0d9488;
  --accentSoft: rgba(14,165,164,.10);
  --accentBorder: rgba(14,165,164,.35);

  /* status */
  --liveBg: #e8f4ed;
  --liveBorder: #a6c5a3;
  --liveText: #324252;

  --disabledBg: #f3f4f6;
  --disabledBorder: #e5e7eb;
  --disabledText: #6b7280;

  /* overlay */
  --overlay: rgba(30, 41, 59, 0.5);
}
```

---

## 7) “Do / Don’t” (So VibeLive stays consistent)

**Do**
- Use teal primarily for key actions and emphasis.
- Keep backgrounds neutral and clean.
- Use Live green only for real “live/online/enabled” states.

**Don’t**
- Add random new colors for every feature.
- Use multiple teal CTAs on the same screen.
- Use emojis as UI icons.


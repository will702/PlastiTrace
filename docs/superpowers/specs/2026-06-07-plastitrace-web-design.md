# PlastiTrace Web Classifier — Design Spec

**Date:** 2026-06-07  
**Scope:** Web classifier only (`web/` → Vercel). Desktop PyQt5 and map features out of scope.  
**Audience:** Indonesian users scanning plastic at home, school, or community events (mobile-first).  
**Register:** Product tool UI (task-focused classifier), not a marketing landing page.

---

## Problem

The current `web/index.html` works but reads as generic AI sustainability slop: emerald-teal-cyan gradients, numbered step cards, emoji status icons, hand-rolled SVGs, layout-shifting hover scales, and a hardcoded `localhost:5001` API URL. It is not production-ready for Vercel deployment.

---

## Recommended Approach: Next.js on Vercel (Approach B)

### Three options considered

| Approach | Summary | Pros | Cons |
|----------|---------|------|------|
| **A. Evolve single HTML** | Refactor `index.html` in place, add env injection via build script | Fastest, zero migration | CDN React/Babel, no `next/font`, weak DX, hard to maintain |
| **B. Next.js on Vercel** ✅ | New `web/` Next.js app, static export or SSR shell + client islands | Vercel-native, env vars, proper fonts/icons, mobile camera + HTTPS | One-time migration (~1 session) |
| **C. Vite + React** | SPA with Vite build | Lighter than Next.js | Less Vercel-integrated; no strong advantage over B for this scope |

**Recommendation:** Approach B. The app is a single-screen tool with camera/upload/classify flow. Next.js App Router with a client component for camera logic is the right Vercel default. Backend stays separate (Render/Railway/HF Spaces); frontend reads `NEXT_PUBLIC_API_URL`.

---

## Design read

*Reading this as: civic environmental product UI for Indonesian users, trust-first language, leaning toward restrained sans + forest/slate palette with one accent, subtle Jakub-style motion.*

### Dials

- **DESIGN_VARIANCE:** 4 (predictable, mobile tool layout)
- **MOTION_INTENSITY:** 3 (state feedback only, 180–220ms)
- **VISUAL_DENSITY:** 5 (camera + results need clear hierarchy, not sparse marketing)

### Color strategy (Committed Restrained)

Avoid the banned emerald-teal-cyan gradient family. Use **Forest + Paper**:

| Token | Light | Dark | Role |
|-------|-------|------|------|
| `--bg` | oklch(0.98 0.004 145) | oklch(0.16 0.012 155) | Page background |
| `--surface` | oklch(1 0 0) | oklch(0.20 0.014 155) | Cards, panels |
| `--ink` | oklch(0.22 0.02 155) | oklch(0.95 0.008 145) | Primary text |
| `--muted` | oklch(0.45 0.02 155) | oklch(0.65 0.015 145) | Secondary text (≥4.5:1) |
| `--accent` | oklch(0.52 0.14 155) | oklch(0.62 0.13 155) | Primary CTA, active states |
| `--accent-ink` | oklch(0.99 0 0) | oklch(0.16 0.012 155) | Text on accent buttons |

One accent (forest green). Status colors semantic only: success/warning/danger for recycling status, not decorative gradients.

**Theme:** `prefers-color-scheme` default with optional manual toggle in header. Page theme locked (no section inversions).

### Typography

- **Font:** `Geist` via `next/font` (sans only; product register, one family)
- **Scale:** Fixed rem — `text-2xl` app title, `text-lg` section headers, `text-base` body, `text-sm` labels
- **`lang="id"`** on `<html>`

### Icons

- **Library:** `@phosphor-icons/react`, stroke weight 1.5
- **No emoji** in UI copy or status rows
- **No hand-rolled SVG paths**

---

## Information architecture (single screen, three states)

```
┌─────────────────────────────────────┐
│  Header: logo wordmark + theme toggle│
├─────────────────────────────────────┤
│  STATE: idle                         │
│  ┌─────────────┐  ┌─────────────┐   │
│  │  Kamera     │  │  Upload     │   │  ← two equal actions OK here (real choice)
│  └─────────────┘  └─────────────┘   │
│  Short helper line (one sentence)    │
├─────────────────────────────────────┤
│  STATE: camera active                │
│  Full-width viewfinder + capture bar │
├─────────────────────────────────────┤
│  STATE: result                       │
│  Image preview │ Classification panel│
│  (stack on mobile)                   │
└─────────────────────────────────────┘
```

**Removed from current UI:**
- Numbered 1-2-3 "Cara Kerja" section (replace with one inline helper sentence in idle state, or collapsible "Bantuan" link)
- Gradient page background
- Confidence bar with gray track (replace with large percentage number + thin accent underline, no filled track)
- Duplicate gradient headers on every card

**Preserved:**
- Upload vs camera modes
- Bahasa Indonesia copy and `RECOMMENDATION` text from `ml/config.py`
- Recycling status tiers (recyclable / conditional / difficult) without emoji

---

## Components

### 1. App shell
- Sticky header, max 64px height, single line
- Wordmark "PlastiTrace" + subtitle "Klasifikasi plastik berbasis AI"
- Theme toggle (sun/moon icon)

### 2. Input chooser (idle)
- Two large tap targets with `cursor-pointer`, border not shadow-card
- Active state: accent border + tinted surface (no `scale-105`)
- `:active` → `scale-[0.98]` tactile feedback

### 3. Camera view
- Black viewfinder, dashed guide frame (keep functional overlay)
- Primary "Ambil Foto" + ghost "Batal"
- Error state if permission denied (inline, not alert box only)

### 4. Results panel
- **Loading:** Skeleton matching final layout (image block + label block + text lines), not center spinner
- **Success:** Plastic type as large label (PET, HDPE, etc.), confidence as `94.2%` number (no progress bar track)
- **Status row:** Icon + title + one line (Bisa didaur ulang / Tergantung fasilitas / Sulit didaur ulang)
- **Recommendation:** Plain text block from API mapping
- **Primary CTA:** "Analisis lagi" (one intent label)

### 5. Error states
- API unreachable: show configured API URL hint + retry
- Classification failed: inline error with retry button
- Empty file: validation before upload

---

## Motion (Jakub-weighted, Emil frequency rules)

| Interaction | Motion | Duration | Reduced motion |
|-------------|--------|----------|----------------|
| Mode select | Border/color transition | 180ms | Instant |
| Result appear | Opacity + 8px y | 220ms | Instant show |
| Button press | scale 0.98 | 100ms | None |
| Theme toggle | Crossfade surfaces | 200ms | Instant swap |

No page-load orchestration. No marquee. No scroll hijacking.

---

## Data flow

```
User (camera/upload)
  → Client: FormData image blob
  → POST ${NEXT_PUBLIC_API_URL}/api/classify
  → Flask API (external host)
  → { label, confidence }
  → Client maps label → RECOMMENDATION[id]
  → Render results
```

**Env vars:**
- `NEXT_PUBLIC_API_URL` — e.g. `https://plastitrace-api.onrender.com` (no trailing slash)
- Fallback in dev: `http://localhost:5001`

**CORS:** Backend must allow Vercel origin in production (document in README, not in this spec's code scope).

---

## File structure (post-migration)

```
web/
├── package.json
├── next.config.ts
├── .env.example          # NEXT_PUBLIC_API_URL=
├── app/
│   ├── layout.tsx        # Geist, theme provider, lang=id
│   ├── page.tsx          # Server shell
│   └── globals.css       # OKLCH tokens, light/dark
├── components/
│   ├── classifier-app.tsx    # 'use client' — main state machine
│   ├── camera-capture.tsx
│   ├── result-panel.tsx
│   ├── input-chooser.tsx
│   └── theme-toggle.tsx
└── lib/
    ├── api.ts            # classifyImage fetch wrapper
    └── recommendations.ts # mirror ml/config.py RECOMMENDATION
```

Legacy `web/index.html` removed or kept as `web/index.legacy.html` for reference until migration verified.

---

## Deployment

| Piece | Host |
|-------|------|
| Frontend | Vercel (connect repo, root `web/`) |
| Backend code | GitHub repo (same monorepo) |
| Backend runtime | Render / Railway / HF Spaces (not GitHub Pages) |

**Vercel settings:**
- Framework: Next.js
- Root directory: `web`
- Env: `NEXT_PUBLIC_API_URL`

**Mobile camera:** Requires HTTPS (Vercel provides). `getUserMedia` with `facingMode: 'environment'`.

---

## Pre-flight checklist (from design skills)

- [ ] Zero em-dashes in copy
- [ ] Zero emoji in UI
- [ ] No gradient backgrounds or gradient buttons
- [ ] No numbered step eyebrows
- [ ] No confidence progress bar with gray track
- [ ] WCAG AA contrast on all text and buttons
- [ ] `prefers-reduced-motion` honored
- [ ] `cursor-pointer` on all clickables
- [ ] CTA labels don't wrap at desktop
- [ ] `lang="id"`
- [ ] API URL not hardcoded to localhost in production build

---

## Out of scope

- Drop-off map / SIPSN locations (desktop only today)
- Realtime video classification (desktop only)
- Backend deployment config (separate task)
- GitHub repo landing/marketing page
- Desktop PyQt5 redesign

---

## Success criteria

1. User on mobile (HTTPS) can photograph plastic, get classification + Indonesian recycling guidance in under 5 seconds on a warm API.
2. UI does not read as generic AI green-gradient slop.
3. `vercel deploy` works with env-based API URL.
4. All interaction states (loading, error, empty, success) are implemented.

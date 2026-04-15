# Animation spec — Provider Bio MVP

This document captures the motion design decisions for the provider bio migration flow. It's structured as an engineer handoff: each section names the trigger, the affected elements, the exact timing values, and the intent behind the choice.

---

## How animation specs are typically handed off

Most design tools (Figma, Principle, ProtoPie) export motion as a prototype link — useful for feel, but not enough to build from. A complete handoff includes:

1. **Trigger** — what user action or state change starts the animation
2. **Target elements** — which DOM nodes move, and in what order
3. **Properties** — opacity, transform, filter, etc.
4. **Timing** — duration, delay, easing curve (ideally as a cubic-bezier)
5. **Keyframes** — start and end (or intermediate) values for each property
6. **Intent** — why this motion exists; what it should communicate

Without intent, engineers optimize for correctness and miss feel. This doc includes the "why" for every decision.

---

## Easing reference

> **Update note:** An earlier iteration used a rightward panel-slide (`translateX(80px → 0)`) for the dashboard-to-flow transition. This was replaced with a cross-dissolve + vertical rise (§6) after review — the panel movement felt like a lateral navigation gesture rather than entering a focused mode.

| Name | Curve | Character |
|---|---|---|
| Default slide | `cubic-bezier(0.4, 0, 0.2, 1)` | Material standard — neutral, directional |
| Deliberate lift | `cubic-bezier(0.22, 1, 0.36, 1)` | Fast initial velocity, long smooth tail — reads as confident |
| Expo ease-out | `cubic-bezier(0.16, 1, 0.3, 1)` | Dramatic deceleration, zero overshoot — snaps in cleanly |
| Standard ease-out | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` | Softer exit deceleration — good for secondary elements |

---

## 1. Screen transitions (default)

**Trigger:** Any "Next" or "Back" button tap (screens 0–3)

**Mechanism:** JS sets inline styles on entering/exiting screens, then transitions to final values.

**Outgoing screen:**
| Property | Start | End | Duration | Easing |
|---|---|---|---|---|
| `opacity` | 1 | 0 | 280ms | `cubic-bezier(0.4, 0, 0.2, 1)` |
| `translateX` | 0 | −40px (forward) / +40px (back) | 280ms | same |

**Incoming screen:**
| Property | Start | End | Duration | Easing |
|---|---|---|---|---|
| `opacity` | 0 | 1 | 280ms | `cubic-bezier(0.4, 0, 0.2, 1)` |
| `translateX` | +40px (forward) / −40px (back) | 0 | 280ms | same |

**Intent:** Directional — the user should feel spatial progress through a linear sequence. Distance (40px) is intentionally small; large values read as page navigation, not step progression.

---

## 2. Review screen entrance (screen 4)

**Trigger:** "Review bio" button tap on Q3 (screen 3)

### 2a. Button loading state

Before the screen transition, the "Review bio" button enters a loading state for 900ms. This is intentional pre-transition anticipation — it signals that something real is happening (data being saved) and creates tension that makes the subsequent reveal feel earned.

| Element | State | Duration |
|---|---|---|
| `#q3-next` | `.is-loading` added | 900ms |
| Button label | Hidden via `color: transparent` | — |
| Spinner (`::after`) | Rotating border-top | `0.62s linear infinite` |

After 900ms: `.is-loading` is removed, `goTo(4)` fires.

### 2b. Screen container

The screen itself slides in from below — vertical rather than horizontal to signal a mode change (from filling to reviewing).

| Property | Start | End | Duration | Easing |
|---|---|---|---|---|
| `opacity` | 0 | 1 | 540ms | `cubic-bezier(0.22, 1, 0.36, 1)` |
| `translateY` | +28px | 0 | 540ms | same |

**Outgoing Q3 screen:**
| Property | End |
|---|---|
| `opacity` | 0 |
| `translateY` | −10px |

### 2c. Content stagger (CSS class-toggle pattern)

On screen enter, JS adds `.is-entering` to `#screen-4`. Elements are hidden at `opacity: 0` by default and animate in via `animation-fill-mode: both` — this lets delay work without a JS-managed opacity toggle.

| Element | Keyframe | Duration | Delay | Easing |
|---|---|---|---|---|
| `.review-page-hdr` | `content-rise` (0→1 opacity, 14px→0 Y) | 440ms | 380ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| `.review-card` | `content-rise` | 540ms | 540ms | `cubic-bezier(0.22, 1, 0.36, 1)` |

**`content-rise` keyframe:**
```css
from { opacity: 0; transform: translateY(14px); }
to   { opacity: 1; transform: translateY(0); }
```

**Intent:** The header reads first (it's the context frame), then the card with the content. The delay stack creates a cascade that rewards watching — each element confirms the one before it.

**Re-trigger on repeated visits:** Removing and re-adding `.is-entering` with a forced reflow (`void el.offsetWidth`) in between replays the CSS animations. Without the reflow, the browser skips the animation because the class never left.

---

## 3. Success screen entrance (screen 5)

**Trigger:** "Submit for review" button tap on the Review screen (screen 4)

### 3a. Button loading state

Same mechanism as the "Review bio" button — 700ms delay.

| Element | State | Duration |
|---|---|---|
| `#review-submit` | `.is-loading` added | 700ms |

### 3b. Screen container

Pure fade — no directional movement. The success screen is a destination, not a step in a sequence. Movement here would feel procedural; stillness lets the icon sequence be the event.

| Property | Start | End | Duration | Easing |
|---|---|---|---|---|
| `opacity` | 0 | 1 | 260ms | `ease-out` |

### 3c. Two-stage icon animation

The icon has two layers: `.splash-spot-glow` (the radial blur behind it) and the `svg` itself. They animate independently to create a sense of energy materializing.

**Stage 1 — Glow blooms first:**

| Element | Keyframe | Duration | Delay | Easing |
|---|---|---|---|---|
| `.splash-spot-glow` | `glow-bloom` | 640ms | 120ms | `cubic-bezier(0.22, 1, 0.36, 1)` |

```css
@keyframes glow-bloom {
  from { opacity: 0; transform: scale(0.2); filter: blur(18px); }
  to   { opacity: 1; transform: scale(1);   filter: blur(10px); }
}
```

**Stage 2 — Icon rises out of the glow:**

| Element | Keyframe | Duration | Delay | Easing |
|---|---|---|---|---|
| `.splash-spot svg` | `icon-emerge` | 460ms | 320ms | `cubic-bezier(0.16, 1, 0.3, 1)` |

```css
@keyframes icon-emerge {
  from { opacity: 0; transform: translateY(12px) scale(0.82); }
  to   { opacity: 1; transform: translateY(0)    scale(1); }
}
```

**Intent:** The glow reads as a flash of energy before the icon is visible. The icon then rises *through* it — it feels like it's being summoned, not just revealed. The overlap (glow at 120ms, icon at 320ms) means the glow is ~40% through its animation when the icon begins, creating visual continuity between the two stages.

### 3d. Text and card cascade

Text comes in *after* the icon peaks — the icon is the emotional event, text confirms it.

| Element | Keyframe | Duration | Delay | Easing |
|---|---|---|---|---|
| `.q-heading` | `success-rise` | 320ms | 440ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |
| `p` (body copy) | `success-rise` | 340ms | 540ms | same |
| `.upsell-card` | `success-rise` | 420ms | 680ms | `cubic-bezier(0.22, 1, 0.36, 1)` |

```css
@keyframes success-rise {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

---

## 4. Bio reveal toggle

**Trigger:** "Show current bio" / "Hide current bio" button tap on question screens

**Mechanism:** CSS max-height transition with asymmetric timing — fast to close, slow to open. The `margin-top: -12px` on the closed state absorbs the flex gap, preventing dead space.

**Opening:**
| Property | Value |
|---|---|
| `max-height` | `0 → 340px` over `520ms cubic-bezier(0.4, 0, 0.2, 1)` |
| `opacity` | `0 → 1` over `420ms ease` |
| `margin-top` | `−12px → 0` over `280ms ease` |

**Closing:**
| Property | Value |
|---|---|
| `max-height` | `340px → 0` over `200ms ease` |
| `opacity` | `1 → 0` over `150ms ease` |
| `margin-top` | `0 → −12px` over `200ms ease` |

**Intent:** Closing fast (200ms) matches how the eye expects a dismissed element to behave. Opening slow (520ms) gives the revealed content room to feel like it's being surfaced rather than popped in. Asymmetry is almost always the right call for reveal/dismiss pairs.

---

## 5. Primary button loading spinner

**Trigger:** Any `.btn-primary` with `.is-loading` added

**Mechanism:** CSS `::after` pseudo-element, absolutely positioned at center, animates via `border-top-color`.

```css
.btn-primary.is-loading { color: transparent !important; pointer-events: none; }
.btn-primary.is-loading::after {
  content: '';
  position: absolute;
  width: 18px; height: 18px;
  top: calc(50% - 9px); left: calc(50% - 9px);
  border-radius: 50%;
  border: 2px solid rgba(255,255,255,0.28);
  border-top-color: rgba(255,255,255,0.92);
  animation: btn-spin 0.62s linear infinite;
}
@keyframes btn-spin { to { transform: rotate(360deg); } }
```

**Intent:** `color: transparent` hides the label without layout shift (the button doesn't resize). `pointer-events: none` prevents double-tap. The spinner uses partial opacity on the track (`0.28`) and high opacity on the active segment (`0.92`) for contrast without being harsh on the green background.

---

## Implementation notes

### CSS class-toggle pattern (entrance animations)
All entrance animations use a single `.is-entering` class added by JS. Elements that should animate start at `opacity: 0` via a base rule, and animate to their final state with `animation-fill-mode: both`. This means delay works correctly without JS needing to manage opacity.

```javascript
function triggerEntrance(screenId) {
  const el = document.getElementById(screenId);
  el.classList.remove('is-entering');
  void el.offsetWidth; // forces reflow — required for animation replay
  el.classList.add('is-entering');
}
```

### Navigation lock pattern
Loading states use a flag + guard at the top of `goTo()` to prevent double-taps or keyboard navigation from interrupting an in-progress transition:

```javascript
let loadingToReview = false;
let loadingToSubmit = false;

function goTo(index) {
  if (loadingToReview || loadingToSubmit) return;
  // ...
}
```

---

## 6. Dashboard ↔ flow transitions

The bio migration flow is a full-page experience launched from the Compass dashboard. The two directions (open and close) are intentionally asymmetric — opening is softer, closing is faster.

### 6a. Opening: dashboard → flow

**Trigger:** "Update bio" tap on the recommendation card

| Element | Keyframe / transition | Duration | Delay | Easing |
|---|---|---|---|---|
| `#dashboard` | `dash-exit`: `opacity: 1 → 0` | 320ms | 0 | `ease-out` |
| `#app` | `flow-enter`: `opacity: 0 → 1`, `translateY(20px → 0)` | 380ms | 120ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |

The 120ms delay on `#app` means the dashboard is already ~37% faded before the flow begins rising in. This avoids both elements competing for attention at the same instant.

**Why cross-dissolve, not a slide:**  
A lateral slide (e.g. `translateX`) reads as spatial navigation — moving to a sibling page. A vertical rise reads as "going deeper" — entering a focused mode. The flow is not a peer of the dashboard; it's a task launched from it. The direction of movement should reflect that hierarchy.

**CSS:**
```css
#dashboard.is-exiting {
  animation: dash-exit 0.32s ease-out both;
  pointer-events: none;
}
@keyframes dash-exit { to { opacity: 0; } }

#app.flow-entering {
  animation: flow-enter 0.38s cubic-bezier(0.25, 0.46, 0.45, 0.94) 0.12s both;
}
@keyframes flow-enter {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

**JS sequence:**
```javascript
function openFlow() {
  dashboard.classList.add('is-exiting');
  app.classList.remove('pre-panel');
  requestAnimationFrame(() => requestAnimationFrame(() => {
    app.classList.add('flow-entering');
  }));
  setTimeout(() => {
    dashboard.classList.add('is-hidden');
    app.classList.remove('flow-entering');
  }, 560);
}
```

The double `requestAnimationFrame` ensures the browser has painted the `pre-panel` removal (opacity: 0) before the `flow-entering` animation class is added — without it, the `from` keyframe may be skipped.

---

### 6b. Closing: flow → dashboard

**Trigger:** X button in any screen's top bar, or "Exit" chip on the success screen. Both use the `.js-flow-close` class; a delegated listener on `document` handles all of them.

| Element | Keyframe / transition | Duration | Delay | Easing |
|---|---|---|---|---|
| `#app` | `flow-exit`: `opacity: 1 → 0`, `translateY(0 → 12px)` | 260ms | 0 | `ease-in` |
| `#dashboard` | `dash-return`: `opacity: 0 → 1` | 360ms | 280ms (after flow finishes) | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` |

The flow exits down (opposite of how it entered up), then the dashboard fades back in cleanly. The flow also resets to screen 0 silently (`goTo(0, true)`) so the next entry starts fresh.

**Why `ease-in` for the exit:**  
Objects that are leaving should accelerate out of frame, not decelerate into it. `ease-in` reads as intentional departure. `ease-out` on the same motion would feel like the screen is reluctantly being dragged away.

**CSS:**
```css
#app.flow-exiting {
  animation: flow-exit 0.26s ease-in both;
  pointer-events: none;
}
@keyframes flow-exit { to { opacity: 0; transform: translateY(12px); } }

#dashboard.is-returning {
  animation: dash-return 0.36s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}
@keyframes dash-return {
  from { opacity: 0; }
  to   { opacity: 1; }
}
```

---

## 7. Entry point — pre-flow state

Before the flow is opened, `#app` sits in the DOM at `opacity: 0; pointer-events: none` (`.pre-panel` class). It is fully rendered but invisible behind `#dashboard`. This avoids a flash of the flow when the page loads and ensures the `flow-enter` animation has a consistent starting state.

The `#dashboard` sits at `z-index: 200`, above `#app`, so the invisible `#app` is never accidentally visible even without the opacity rule.

---

### Sheet → dialog rule (not yet animated, for reference)
On `base`/`sm`: edit surfaces are bottom sheets. On `md`+: they become dialogs. When this is implemented, sheet entrance should be `translateY(100% → 0)` with the same asymmetric timing as the bio reveal. Dialog entrance should be `scale(0.96 → 1) + opacity(0 → 1)` — the scale difference is smaller because dialogs are centered, not anchored to an edge.

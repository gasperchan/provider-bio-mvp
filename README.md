# Provider Bio — MVP Prototype

Interactive prototype for the provider bio migration flow in Compass (Spring Health). Explores screen transitions, animation, and the structured bio editing experience for existing providers.

**Live:** https://gasperchan.github.io/provider-bio-mvp/
**Password:** `foobarbazz`

---

## What's in here

| File | Description |
|---|---|
| `index.html` | Single-file prototype — all HTML, CSS, and JS |
| `animations.md` | Animation spec and engineer handoff doc |
| `provider-bio-mvp-requirements.md` | MVP product requirements |

## Flow

```
Dashboard (homepage)
  → Intro screen
  → Loading (AI extraction)
  → Q1: My approach
  → Q2: Areas I specialize in
  → Q3: What a session feels like
  → Review
  → Confirmation
```

The X button at any point returns to the dashboard. "Exit" on the confirmation screen does the same.

## Running locally

No build step. Open `index.html` directly in a browser.

```bash
open index.html
```

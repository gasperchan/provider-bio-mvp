# Provider Bio — MVP Requirements

This document specifies the MVP experience for migrating existing providers from freeform bios to the structured bio format in Compass. Scope is existing providers only — new provider onboarding is deferred.

**Design system:** Verdant UI (`design.md`). Use the compass product theme. All tokens, components, and breakpoints referenced below are from that system.

---

## Scope and constraints

- **Who:** Existing providers with a freeform bio. All providers in this flow have an existing bio.
- **AI scope (V1):** Extraction (harvest relevant sentences from existing bio) and profile data surfacing (chips). No AI drafting or per-section rewriting (D-09).
- **Required completion:** Providers cannot advance past a question without filling it in. Review step cannot show empty sections.

---

## Flow overview

```
Entry (dashboard nudge or Profile settings)
  → Loading (AI extraction runs)
  → Intro screen
  → Question 1: My approach
  → Question 2: Areas I specialize in
  → Question 3: What a session feels like
  → Review (edit via sheet on mobile / dialog on desktop)
  → Confirmation
```

---

## Responsive layout

The flow is a focused form. Content is centered at all breakpoints with a maximum width of `size/layout/focused` (608px). The mobile frame (390px) is the primary design target; desktop uses the same component hierarchy with more surrounding page context.

| Breakpoint | Layout behavior |
|---|---|
| `base` (0px) | Full-width, mobile-first |
| `md` (768px) | Centered, max 608px, page background visible on sides |
| `lg`+ (992px+) | Same as `md` — no structural changes |

**Sheet → dialog rule:** On `base`/`sm`, contextual edit surfaces (review section editing) use bottom sheets (`shadows/day/base`, anchored to bottom). On `md`+, they use dialogs (`shadows/day/bold`, `radius/xl`).

---

## 1. Entry surface

**Primary:** Dashboard (Compass home) nudge card — shown until provider completes the structured bio flow.
**Secondary:** Profile settings (persistent entry, always accessible).

**Dashboard card:**
- Section label: "Your profile" + "Opportunity" badge
- Signal: booking conversion rate vs. peers (e.g. "Top providers in your specialty average 40%+ conversion")
- CTA: "See your profile →" or "Update my bio →"

**Trigger condition:** Provider has a freeform bio and has not completed the structured format.
**After dismissal:** Nudge card is permanently hidden. Entry is available via Profile settings only.

---

## 2. Loading screen

Shown immediately after the provider taps the entry CTA. AI extraction runs here — provider waits on this screen before reaching the intro.

**Elements:**
- Loading indicator
- Brief copy explaining what's happening (e.g. "Getting your profile ready…")
- No user action required

**On completion:** Advances automatically to the Intro screen.
**On extraction failure:** Advances to the Intro screen. All question screens fall back to empty state (State 2) — the flow continues without extracted content. No error shown to provider.

---

## 3. Intro screen

**Purpose:** Prime the provider on why the format change matters and set scope expectations before asking for effort.

**No before/after toggle.** A single visual graphic — illustrative, representing the act of writing — is the hero.

**Key elements:**
- Graphic: visual representing writing / profile authorship
- Headline: short, member-outcome framed (e.g. "Help members find the right fit")
- Body: 1–2 sentences on what the new format does for members
- Scope preview: list of 3 questions they'll answer + time estimate (~8 minutes)
- Social proof line: "Providers with structured bios see 23% more first appointments"
- Primary CTA: "Get started"

---

## 4. Question flow

Three questions, one screen each. Progress bar at top uses **section name** as the label, not a countdown ("My approach", not "1 of 3") — avoids loss-aversion pressure at the final step (D-05).

Questions:
1. **My approach** — how you work, what guides your sessions
2. **Areas you specialize in** — clients and challenges you focus on
3. **What a session with you feels like** — tone, vibe, what clients would say

**Validation:** Each question is required. The "Next" / "Done" button is disabled or blocked until the textarea has content. Providers cannot skip a section.

---

### 4a. Question screen anatomy

- **Section name** — H1, large and bold (`heading/small` or `heading/medium`)
- **Secondary text** — clarifying question below the heading, redirecting toward member perspective (e.g. *"Who actually gets the most out of working with you?"*)
- **Textarea** — pre-filled or empty (see §4b)
- **Chips card** — below textarea, question-dependent, only when relevant profile data exists (see §4c)
- **Caption row** — single persistent slot below the textarea (and chips), content changes dynamically based on state (see §4b)
- **Footer:** Back / Next (or "Done" on Q3)

---

### 4b. Textarea states

There is no separate bio harvest preview screen. The fork happens directly in Q1 (and each subsequent question): either the AI found a transferable sentence from the existing bio, or it didn't.

The caption row is a single persistent element. Its content changes based on the state below.

#### State 1 — AI found a transferable sentence
- Textarea pre-filled with the extracted sentence in the provider's **raw words** (not translated or paraphrased — D-07)
- Caption: *"Suggested from your bio. Edit or replace it."*
- Clear button (×) in the top-right corner of the textarea

#### State 2 — AI found no transferable content for this question
- Textarea is empty; standard placeholder text shown inside the field
- Caption: question-specific writing prompt (e.g. Q1: *"Start with a few sentences about how you work with clients"*)
- Chips still shown if profile data exists (see §4c)

#### State 3 — Provider clears the pre-filled field
- Textarea is empty
- Caption changes to: **"Restore from bio"** (tappable link, `color/primary/base`) — tapping re-populates with the original extracted sentence
- This is the undo for rejection; no modal required

#### State 4 — Provider has edited the pre-fill
- Caption: *"Keep it simple — you can refine this later."*
- Restore link is hidden unless the field is fully emptied (which triggers State 3)
- Clear button (×) remains visible

#### State 5 — Field is empty, Next tapped
- Inline validation: textarea border highlights (`color/negative/base`, `border-width/thick`)
- Caption changes to question-specific error copy (e.g. Q1: *"Add a few details about how you work before moving on"*)
- Provider cannot advance until content exists

**Caption copy is question-scoped.** States 2 and 5 use different text per question. States 1, 3, and 4 use the same copy across all three questions.

---

### 4c. Chips — additional context per question

Reference chips appear in a card **below** the textarea (not above — reference below acts as a check, not a constraint). Shown only when relevant profile data exists. Chips are not insertable — they surface existing profile context to jog memory.

| Question | Chip source | Card label |
|---|---|---|
| My approach | Modalities on provider profile (CBT, DBT, Mindfulness, etc.) | "Worth keeping" |
| Areas I specialize in | Specialties + conditions on profile (Anxiety, Burnout, ADHD, etc.) | "Worth keeping" |
| What a session feels like | None | — |

The "Worth keeping" label reinforces review behavior without challenging the pre-fill.

No chips shown if profile has no relevant data for that question.

---

## 5. Review screen

Presents all 3 completed sections before saving. Because all fields are required, every section will have content.

**Editing a section** opens a focused edit surface — no navigation back to the question screen:
- **Mobile (`base`/`sm`):** bottom sheet containing the section's textarea + save/cancel
- **Desktop (`md`+):** dialog containing the section's textarea + save/cancel

**Section card anatomy:**
- Section name (label, small, `color/content/secondary`)
- Content (read-only until tapped/clicked)
- Edit affordance (edit icon or full-card tap target)

**Footer:** "Save bio" — commits all sections and advances to confirmation.

---

## 6. Confirmation screen

- Checkmark icon
- Headline: "Bio saved"
- Body: *"Your new bio is live. Members can now see your approach and style at a glance."*
- No required next action

---

## Decisions referenced

| ID | Decision |
|---|---|
| D-01 | Structured multi-question format over freeform |
| D-02 | Frame prompts around member outcomes, not provider credentials |
| D-04 | Always show provenance when pre-filling from existing bio |
| D-05 | Section name for progress label, not countdown |
| D-07 | Edit-in-place with raw voice (not translated) |
| D-08 | Caption validates pre-fill before redirecting |
| D-09 | V1 AI scope: extraction and topic suggestion only — no per-section drafting or rewriting |

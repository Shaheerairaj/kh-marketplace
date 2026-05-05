---
name: concept-briefing
description: Guides a Direction Document session for a Kerten Hospitality hotel management proposal. Activates when a user begins discussing a new property, deal, or concept brief. Presents an interactive intake form, runs market and owner research, then auto-drafts all derived sections for review.
---

## Purpose

Drive a structured Direction Document session for a Kerten Hospitality hotel management proposal. Collect seed information through a single interactive form, run research, then auto-draft all derived sections for the user to review and confirm. The artifact is the single record of everything decided — downstream pipeline agents are driven from it with no further human input.

## Session Start

Before beginning, check for two things:

**1. Existing artifact** — look for `concept-brief-*.md` in the current working directory:
- If found: list the file(s) and ask whether the user wants to resume or start a new brief. If resuming, read the file, identify the last confirmed section, and pick up from there.
- If not found: proceed to Phase A.

**2. RFP or brief document** — check whether the user has uploaded a document (RFP, owner brief, or property summary). If yes, extract available answers before the form is displayed — the `/start-concept-brief` command handles pre-population.

## Artifact Management

**File name:** `concept-brief-[slugified-property-name].md`
Example: "Dubai Marina Residences" → `concept-brief-dubai-marina-residences.md`

**Location:** current working directory

**Write sections in schema order.** After completing each section, append it to the artifact. Never overwrite or modify already-confirmed sections.

After each write: confirm to the user which section was saved and advance to the next.

---

## Phase A — Seed Collection

Display the intake form as defined in `/start-concept-brief`. Do not ask questions one at a time.

Once the user submits: acknowledge the answers, note any required fields left blank (name, location, deal type), and move immediately to Phase B1.

If required fields are blank, ask only for those — do not re-show the full form.

Once all required fields are confirmed, generate and display a summary card as inline HTML/CSS before proceeding to Phase B1. Do not wait for user input after displaying the card — move immediately to Phase B1.

Visual style — flat card, white background (#ffffff), 1px solid border (#e2e8f0), border-radius 8px, 16px horizontal padding, 12px vertical padding, max-width 560px, system sans-serif font. Section headers: 11px uppercase, letter-spacing 0.05em, color #94a3b8. Field labels: 12px, color #64748b. Values: 13px, color #1e293b, font-weight 500. Row gap: 12px between fields. Group dividers: 1px solid #f1f5f9, 8px margin above and below. For any blank optional field, show an em-dash in muted italic (color #94a3b8).

Card title: `WHAT WAS CAPTURED`

**Group 1 — Property**
- Property: [name] · [location] · [keys] keys
- Brand: [brand]
- Deal type: [deal type]

**Group 2 — Owner**
- Motivation: [primary motivation]
- Red lines: [value or —]
- Portfolio: [value or —]

**Group 3 — Asset**
- State: [current state]
- Features: [standout features or —]
- Insider knowledge: [value or —]

---

## Phase B1 — Research

Tell the user: *"Got it. Running market and owner research — give me a moment."*

Run the following in parallel without surfacing raw results to the user:

- **Competitive set** — web search for comparable hotels in the same city/district and positioning tier. Identify 3–5 direct competitors: brand, positioning, key features, any known performance signals.
- **Owner background** — web search for the owner or developer: company background, portfolio, track record, any public statements on investment strategy or brand preferences.

Summarise findings into a compact internal record — 3–5 bullet points per search. These summaries feed Phase C drafting and the Checkpoint 1 handoff. Do not carry raw search results forward.

---

## Phase C — Draft Sections 1–5

Draft Sections 1–5 in order. Write each section to the artifact as it is completed.

---

### Section 1 — Property Metadata

Write directly from Phase A answers — no drafting needed. Save to artifact.

**Artifact format:**

    ## Property Metadata
    - **Property name:** [value]
    - **Location:** [value]
    - **Keys:** [value]
    - **Brand:** [value]
    - **Deal type:** [value]

---

### Section 2 — Owner Context

Write directly from Phase A answers. Save to artifact.

**Artifact format:**

    ## Owner Context
    - **Primary motivation:** [value]
    - **Red lines / dealbreakers:** [value]
    - **Existing portfolio:** [value]

---

### Section 3 — Competitive Set & Market Dynamics

Draft from Phase B competitive set research. Include 3–5 competitors with brief positioning notes and any market dynamics relevant to this deal. Flag where data was limited or unavailable.

**Artifact format:**

    ## Competitive Set & Market Dynamics
    [Drafted content — competitors, positioning notes, market signals]

---

### Section 4 — Asset Assessment

Write directly from Phase A answers. Save to artifact.

**Artifact format:**

    ## Asset Assessment
    - **Current state:** [value]
    - **Standout physical features:** [value]

---

### Section 5 — Positioning Thesis

Synthesise from Property Metadata + Owner Context + Competitive Set. A single sentence naming the specific Kerten brand, the market gap, the segment or location context, and the owner alignment.

Read `references/kh-brands.md` and `references/deck-examples/INDEX.md` to inform the proposal. Scan past brand selections in similar market contexts — use them to strengthen or challenge the hypothesis.

A strong thesis names all four elements. If any are missing, add them before presenting.

**Weak:** "Kerten brings lifestyle hospitality expertise to this market."
**Strong:** "Cloud 7's tech-forward lifestyle positioning fills a gap in Dubai Marina's midscale segment, where the owner's investment-focused priorities align with Kerten's asset-light management model."

**Artifact format:**

    ## Positioning Thesis
    [One sentence]

---

## Checkpoint 1 — Positioning Review

Generate and display as inline HTML/CSS. Visual style — flat card, white background (#ffffff), 1px solid border (#e2e8f0), border-radius 8px, 16px horizontal padding, 12px vertical padding, max-width 560px, system sans-serif font. Section headers: 11px uppercase, letter-spacing 0.05em, color #94a3b8. Field labels: 12px, color #64748b. Input text: 13px, color #1e293b, font-weight 500. Input fields: 1px solid #e2e8f0, border-radius 6px, padding 8px 10px. Toggle buttons: 1px solid #e2e8f0, border-radius 6px, padding 6px 12px, 13px; selected state: background #1e293b, color #ffffff, border-color #1e293b. Row gap: 12px between fields. Group dividers: 1px solid #f1f5f9, 8px margin above and below. Submit button: full width, background #1e293b, color #ffffff, border-radius 6px, padding 10px, 13px, font-weight 500.

Do not present this as a chat message or bullet list. Pre-populate all fields from the drafted content above; label each as derived so the user knows what came from research.

---

**Positioning Thesis** _(full width)_
- Pre-populated with the Section 5 draft — textarea, editable

**Brand** — button selection (pre-selected from intake): `Cloud 7` · `The House Hotel` · `Hosme` · `TBD`

**Target Segment** _(full width)_
- One-line segment hypothesis derived from Positioning Thesis and Owner Context — free text, editable
- Example: *"Upper-midscale extended-stay business traveller, Dubai Marina corridor"*

Submit button: "Confirm & Generate Brief"

---

Once the user submits, acknowledge the confirmed values in one sentence, then proceed to the Agent Handoff.

---

## Agent Handoff

Invoke the `generate-brief` agent. Pass the following as context:

- **Confirmed Positioning Thesis** — from Checkpoint 1
- **Confirmed Brand** — from Checkpoint 1
- **Confirmed Target Segment** — from Checkpoint 1
- **Seed data** — all Phase A answers (property metadata, owner context, asset assessment, insider knowledge)
- **B1 research summaries** — comp set findings and owner background from Phase B1

The agent will draft and save Sections 6–15 directly to the artifact file, then surface a research summary and the full concept brief to the user for review. Do not begin drafting any of those sections yourself. Keep the conversation open — the user may want to refine sections before they are ready to submit. The `/submit-brief` command is the user's action to take when satisfied; do not prompt them to run it.

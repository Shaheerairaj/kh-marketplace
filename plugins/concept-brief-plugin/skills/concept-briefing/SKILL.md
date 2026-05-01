---
name: concept-briefing
description: Guides a Direction Document session for a Kerten Hospitality hotel management proposal. Activates when a user begins discussing a new property, deal, or concept brief. Presents an interactive intake form, runs market and owner research, then auto-drafts all derived sections for validation.
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

**Write sections in schema order** (as listed in Phase C). After the user confirms each section, append it to the artifact. Never overwrite or modify already-confirmed sections.

After each write: confirm to the user which section was saved and advance to the next.

---

## Phase A — Seed Collection

Display the intake form as defined in `/start-concept-brief`. Do not ask questions one at a time.

Once the user submits: acknowledge the answers, note any required fields left blank (name, location, deal type), and move immediately to Phase B.

If required fields are blank, ask only for those — do not re-show the full form.

---

## Phase B — Research

Tell the user: *"Got it. Running market and owner research — give me a moment."*

Run the following in parallel without surfacing raw results to the user:

- **Competitive set** — web search for comparable hotels in the same city/district and positioning tier. Identify 3–5 direct competitors: brand, positioning, key features, any known performance signals.
- **Owner background** — web search for the owner or developer: company background, portfolio, track record, any public statements on investment strategy or brand preferences.

Synthesise findings internally — they feed Phase C drafts directly.

---

## Phase C — Auto-draft and Validate

Draft all sections in schema order without pausing. Write each section to the artifact as it is completed. Once all sections are drafted and saved, present a single summary of everything written and ask: *"Anything you'd like to change?"*

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

### Section 6 — Concept Direction

Read `references/sections/03-concept-direction.md` for field definitions, sub-section guidance, and quality bar.

Derive from Positioning Thesis + brand guidelines. Draft Vision, Pillars, Target Guest, and Differentiators as a complete block and present all four together for review.

**Artifact format:**

    ## Concept Direction

    ### Vision
    [1–2 sentences]

    ### Pillars
    - **[Pillar Name]** — [one sentence: what it means at this property]

    ### Target Guest
    [3–5 sentence portrait — psychographic, not demographic]

    ### Differentiators
    - [Specific, defensible claim tied to a concrete feature or programme]

---

### Section 7 — Key Messages / Proof Points

Read `references/sections/05-key-messages.md` for quality bar.

Derive from Owner Context + Positioning Thesis. Each message should directly address a motivation or concern surfaced in the owner's answers. 3–5 messages maximum.

**Artifact format:**

    ## Key Messages
    - [Message]
    - [Message]

---

### Section 8 — Experience Inventory

Read `references/sections/06-experience-inventory.md` for sub-section guidance and quality bar.
Read `references/kh-fnb-brands.md` — do not propose F&B concepts without consulting this first.

Derive from Concept Direction + property scale (keys).

**Artifact format:**

    ## Experience Inventory

    ### Signature Experiences
    - **[Name]** — [one-line description]

    ### F&B
    - **[Brand / Concept]** — [one-line description]

    ### Wellness
    - **[Offering]** — [one-line description]

---

### Section 9 — Financial Anchors

Derive from Competitive Set research. ADR and occupancy data are not available via standard sources — flag these fields explicitly as absent. Do not fabricate values. Note which benchmarks came from research vs. require manual input from Karolina.

**Artifact format:**

    ## Financial Anchors
    - **Comp set ADR range:** [value or "Not available — manual input required"]
    - **Comp set occupancy:** [value or "Not available — manual input required"]
    - **Revenue streams:** [list]
    - **Market notes:** [summary of available benchmarks]

---

### Section 10 — Commercial Terms

Derive from market context + owner's existing portfolio. Flag assumptions explicitly.

**Artifact format:**

    ## Commercial Terms
    [Drafted content]

---

### Section 11 — Design Direction

Derive from Concept Direction + brand guidelines. Reference `references/kh-brands.md` for brand-specific design language.

**Artifact format:**

    ## Design Direction
    [Drafted content]

---

### Section 12 — Phasing _(optional)_

Include only if the deal type or conversation context indicates a phased build or opening. Skip if not applicable.

**Artifact format:**

    ## Phasing
    [Drafted content]

---

### Section 13 — Partners & Collaborators _(optional)_

Include only if Concept Direction points to specific partnership opportunities. Skip if not applicable.

**Artifact format:**

    ## Partners & Collaborators
    [Drafted content]

---

### Section 14 — Insider Knowledge & Relationships _(optional)_

If the user answered the open closer in Phase A, record it here. This section cannot be derived — write what was shared, lightly structured.

**Artifact format:**

    ## Insider Knowledge & Relationships
    [What was shared]

---

### Section 15 — Open Questions for Client _(optional)_

Surface any gaps from the above that require client input before the pipeline can run. Skip if there are none.

**Artifact format:**

    ## Open Questions for Client
    - [Question]

---

_When all sections are confirmed, remind the user to run `/submit-brief` to validate and submit the Direction Document to the pipeline._

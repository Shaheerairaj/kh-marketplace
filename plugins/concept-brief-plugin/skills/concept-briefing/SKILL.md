---
name: concept-briefing
description: Guides a section-by-section Direction Document conversation for a Kerten Hospitality hotel management proposal. Activates when a user begins discussing a new property, deal, or concept brief. Manages a running artifact that records confirmed sections.
---

## Purpose

Drive a structured, conversational Direction Document session for a Kerten Hospitality hotel management proposal. Work through sections one at a time: propose content, confirm with the user, write to the artifact, advance. The artifact is the single record of everything decided — downstream pipeline agents are driven from it with no further human input.

## Session Start

Before beginning, check for an existing artifact in the current working directory:

- If the property name is known, look for `concept-brief-[slugified-property-name].md`
- If the property name is not yet known, list any `concept-brief-*.md` files and ask if the user is resuming one
- If a file exists: read it, identify the last confirmed section, tell the user where you are picking up from, and resume from the next section
- If no file exists: copy `assets/concept-brief-template.md` to `concept-brief-[slugified-property-name].md` and begin from Step 1

## Artifact Management

**File name:** `concept-brief-[slugified-property-name].md`
Example: "Dubai Marina Residences" → `concept-brief-dubai-marina-residences.md`

**Location:** current working directory

**After each confirmed section:** append the section to the end of the file. Never overwrite or modify already-confirmed sections.

**After writing:** tell the user: "Saved to `concept-brief-[name].md`" and immediately advance to the next step.

## Conversation Protocol

Repeat this loop for every section:

1. **Announce** — state the section name and its one-line purpose
2. **Propose** — draft content based on everything discussed so far. Always lead with a concrete proposal; never ask a blank open question
3. **Refine** — iterate with the user until they are satisfied
4. **Confirm** — ask explicitly: "Happy for me to save this?"
5. **Write** — append the confirmed section to the artifact
6. **Advance** — move to the next step

If there is not enough information to make a meaningful proposal for a field, ask one targeted question to unblock — then propose.

---

## Workflow: Build Concept Brief

### Step 1 — Property Metadata

**Purpose:** Establish the factual foundation of the property before any creative or strategic work begins.

| Field | Description |
|---|---|
| Property name | Full name or working title of the property |
| Location | City, country, and neighbourhood or district if known |
| Keys | Number of hotel rooms. If unknown, record as TBD |
| Owner | Owner's name and company or entity |
| Positioning tier | Budget / Midscale / Upper Midscale / Upscale / Upper Upscale / Luxury |
| Brand fit hypothesis | Which Kerten brand is being considered and the initial rationale |

Gather fields through conversation — one question at a time, not a list. For **Brand Fit Hypothesis**: propose based on positioning tier and location rather than waiting for the user to volunteer it. Read `references/kh-brands.md` to inform the proposal; if the tier and location clearly point to one brand, name it and explain why in one sentence. If there is genuine ambiguity, surface both and ask the user to steer. Scan `references/deck-examples/INDEX.md` to identify past brand selections in similar market contexts — use this to strengthen or challenge the hypothesis.

**Artifact format:**

    ## Property Metadata
    - **Property name:** [value]
    - **Location:** [value]
    - **Keys:** [value]
    - **Owner:** [value]
    - **Positioning tier:** [value]
    - **Brand fit hypothesis:** [value]

---

### Step 2 — Positioning Thesis

**Purpose:** A single sentence capturing why Kerten is the right operator for this property and what the winning angle is. Everything downstream should be traceable back to it.

Read `references/kh-brands.md` if not already in context. Synthesise from Step 1 and any additional context in the conversation. A strong thesis names the specific Kerten brand, the market gap or opportunity, the segment or location context, and the owner alignment or deal rationale. Do not make a generic claim about Kerten's capabilities — push toward specificity at every iteration.

**Weak:** "Kerten brings lifestyle hospitality expertise to this market."
**Strong:** "Cloud 7's tech-forward lifestyle positioning fills a gap in Dubai Marina's midscale segment, where the owner's investment-focused priorities align with Kerten's asset-light management model."

If any of the four elements (brand, gap, segment, owner alignment) are missing, point out what is absent and propose a revision that adds it.

**Artifact format:**

    ## Positioning Thesis
    [One sentence]

### Step 3 — Concept Direction
Read `references/sections/03-concept-direction.md` for field definitions, sub-section guidance, and quality bar.
Read `references/deck-examples/INDEX.md` if not already in context; identify the 1–2 examples closest to the current brand and market type; load those full files as pattern reference for Vision and Pillars proposals.

### Step 4 — Owner Context
Read `references/sections/04-owner-context.md` for the three-input flow: relationship intelligence intake, web research, and archetype matching.
Read `references/kh-owner-archetypes.md` to identify the owner's primary archetype after intake and research are complete.
Use web search to research the owner or developer — company background, portfolio, track record, and any public statements on investment strategy.

### Step 5 — Key Messages
Read `references/sections/05-key-messages.md` for proposal approach and quality bar.
Key Messages are derived entirely from the confirmed Owner Context (Step 4) — do not begin this step until Step 4 is saved to the artifact.

### Step 6 — Experience Inventory
Read `references/sections/06-experience-inventory.md` for sub-section guidance and quality bar.
Read `references/kh-fnb-brands.md` to match F&B brands to this property — do not propose F&B concepts without consulting this first.
Do not begin this step until Step 3 (Concept Direction) is confirmed.

---

_Steps 7–15 will be added as subsequent sections are built out._

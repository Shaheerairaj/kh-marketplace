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

Once the user submits: acknowledge the answers, note any required fields left blank (name, location, keys, brand, deal type, primary motivation, current state), and move immediately to Phase B1.

If required fields are blank, ask only for those — do not re-show the full form.

Once all required fields are confirmed, generate and display a summary card as inline HTML/CSS before proceeding to Phase B1. Do not wait for user input after displaying the card — move immediately to Phase B1.

Read `references/ui-style.md` for visual style. This is a display-only card — apply card shell, typography, layout, and empty-field guidelines.

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

Invoke two instances of the `research` agent in parallel. Do not proceed until both return.

---

**Invocation 1 — Competitive Set**

Context to pass:
- Property name, location, keys, brand, deal type
- Positioning tier derived from brand: Cloud 7 → lifestyle midscale; The House Hotel → upper-upscale lifestyle; Hosme → affordable lifestyle

Research instructions:
Find 3–5 hotels in the same city or district operating at a comparable positioning tier to this property. For each, note the hotel name, brand or operator, positioning description, standout features or differentiators, and any known performance signals (occupancy, rate positioning, reputation). Also note any significant market dynamics — new supply, demand shifts, or context relevant to how this property will compete.

Return fields:
- `competitors` — list of 3–5 entries: name, brand/operator, positioning, key features, performance signals
- `market_dynamics` — 2–3 sentences on the competitive landscape
- `data_gaps` — any competitors or signals you could not verify
- `sources` — all sources used

---

**Invocation 2 — Owner Background**

Context to pass:
- Owner/developer name and company (from Phase A owner context)
- Existing portfolio details if provided

Research instructions:
Research the owner or developer: company background, scale and track record, any known properties in their portfolio and which brands they operate under, public statements on investment strategy or brand partnerships, and any signals on what they prioritise in a management partner.

Return fields:
- `company_background` — 2–3 sentences
- `portfolio` — known properties and brands
- `investment_strategy` — what they appear to prioritise
- `brand_preferences` — any stated or inferred brand affinities
- `data_gaps` — anything you could not verify
- `sources` — all sources used

---

Once both return, summarise results into compact internal records. These feed Phase C drafting and the Checkpoint 1 form. Do not surface raw results to the user.

---

## Phase C — Draft Sections 1–5

Draft sections in order. Write each to the artifact as it is completed.

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

Draft from Phase B1 competitive set research results. Include 3–5 competitors with brief positioning notes and market dynamics relevant to this deal. Flag where data was limited or unavailable.

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

Read `references/ui-style.md` for visual style. Generate and display as inline HTML/CSS. This is an interactive form — apply card shell, typography, layout, and interactive element guidelines.

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

Once the user submits, acknowledge the confirmed values in one sentence, then proceed to Phase D.

---

## Phase D — Deep Research

Tell the user: *"Super complicated deep research being conducted for the remaining sections — this may take a moment. Grab a coffee while you wait."*

Invoke three instances of the `research` agent in parallel. Do not proceed to drafting until all three return.

Context to pass to all three invocations:
- All Phase A answers (property metadata, owner context, asset assessment, insider knowledge)
- Phase B1 research summaries (competitive set + owner background)
- Confirmed Positioning Thesis, Brand, and Target Segment from Checkpoint 1

---

**Invocation 1 — Experience & Cultural Context** _(for Section 8)_

Research instructions:
Research what is culturally distinctive about [location] that a hotel experience could genuinely connect to. Look for: local craft traditions, food culture and culinary scene, seasonal rhythms and events, community scenes and subcultures, what visitors come here specifically to find, and any underrepresented cultural narratives a hospitality concept could authentically express. Focus on what is specific and ownable — not generic tourism descriptors.

Return fields:
- `cultural_anchors` — list of 3–5 specific themes with a brief description and its potential for hospitality expression
- `local_distinctives` — 2–3 sentences on what makes this location genuinely different
- `seasonal_context` — demand patterns, peak seasons, local calendar events worth building around
- `sources` — all sources used

---

**Invocation 2 — Financial Benchmarks** _(for Section 9)_

Research instructions:
Research financial performance benchmarks for hotels in [location] at the [positioning tier] tier. Find: ADR ranges for comparable properties (check booking.com listings, hotel industry publications, rate benchmarks), RevPAR signals and occupancy context for this market, demand drivers and key feeder markets, seasonality patterns, and revenue stream benchmarks for this hotel type (F&B capture rate, spa penetration, ancillary revenue norms). Search multiple sources. Flag any field you cannot verify with real data — never estimate without attribution.

Return fields:
- `adr_range` — value with source, or "Not found — flagged for follow-up"
- `occupancy` — value with source, or "Not found — flagged for follow-up"
- `revpar_signals` — any RevPAR context available
- `demand_drivers` — key feeder markets and demand sources
- `seasonality` — peak/off-peak patterns
- `revenue_benchmarks` — F&B capture, spa penetration, ancillary norms if available
- `flagged_fields` — list of fields absent or unverifiable
- `sources` — all sources used

---

**Invocation 3 — Commercial & Deal Norms** _(for Section 10)_

Research instructions:
Research standard hotel management contract structures and deal norms for [country or region]. Find: typical base and incentive fee structures, key money or owner contribution norms, standard term lengths, operator protections and owner rights typically negotiated, and any regional specifics that differ from global norms. Note any recent shifts in deal terms in this market.

Return fields:
- `fee_structures` — typical base fee %, incentive fee %, and how they are calculated
- `deal_norms` — term lengths, key money, owner/operator right balance
- `regional_specifics` — anything distinctive about this market vs. global norms
- `flagged_assumptions` — anything you could not verify for this specific market
- `sources` — all sources used

---

## Phase E — Draft Sections 6–15

Draft sections in order. For each section: complete any file reads first, then draft, then save to artifact.

---

### Section 6 — Concept Direction

Read `references/deck-examples/INDEX.md`. Load 2–3 deck examples for properties with comparable brand and market context. Scan for Vision and Pillar patterns relevant to this concept. Read `references/sections/03-concept-direction.md` for field definitions and quality bar.

Draft Vision → Pillars → Target Guest → Differentiators in order. A strong Vision is specific to this place. Pillars name an operational idea, not a category. Target Guest is a psychographic portrait (3–5 sentences). Differentiators pass the "why can't the Marriott across the street do this?" test.

**Artifact format:**

    ## Concept Direction

    ### Vision
    [1–2 sentences]

    ### Pillars
    - **[Pillar Name]** — [one sentence: what it means at this property]

    ### Target Guest
    [3–5 sentence portrait]

    ### Differentiators
    - [Specific, defensible claim tied to a concrete feature or programme]

---

### Section 7 — Key Messages / Proof Points

Read `references/sections/05-key-messages.md` for quality bar. Derive from Owner Context and confirmed Positioning Thesis. Each message should directly address a motivation or concern from the owner's answers. 3–5 messages maximum.

**Artifact format:**

    ## Key Messages
    - [Message]

---

### Section 8 — Experience Inventory

Read `references/kh-fnb-brands.md` before proposing any F&B concepts — do not suggest a brand without consulting it first. Read `references/sections/06-experience-inventory.md` for sub-section guidance. Load 1–2 relevant deck examples from the INDEX for comparable wellness and signature experience patterns.

Use the Phase D cultural context research results alongside confirmed Pillars and the property's physical features to generate original experience ideas. Set past examples aside when thinking originally — precedents inform, not constrain.

For F&B: propose Kerten-branded concepts from the library first where they genuinely fit. If a needed F&B role has no library match, propose a new concept and label it clearly.

For Wellness: always derived fresh — propose a register and 2–3 anchors specific to this property. No standard Kerten wellness brand exists.

For Signature Experiences: propose two pools — _(Kerten precedent)_ drawn from comparable properties, and _(original concept)_ grounded in something specific to this location, asset, or owner context. Both should pass the "could only happen here" test.

**Artifact format:**

    ## Experience Inventory

    ### F&B Concepts
    - **[Brand Name]** _(Kerten F&B brand)_ — [one sentence: why this brand, which Pillar it serves]
    - **[Concept Name]** _(new concept)_ — [one sentence: what it is and why it fits here]

    ### Wellness
    [Register and 2–3 anchors. 3–5 sentences.]

    ### Signature Experiences
    - **[Experience Name]** _(Kerten precedent)_ — [one sentence: what it is, what past property it draws from, how it is adapted here]
    - **[Experience Name]** _(original concept)_ — [one sentence: what it is, what specific local/physical/owner context grounds it]

---

### Section 9 — Financial Anchors

Draft from Phase D financial benchmark research results. Distinguish clearly between benchmarks sourced from research and fields that could not be populated. Flag absent fields explicitly — do not fabricate values.

**Artifact format:**

    ## Financial Anchors
    - **Comp set ADR range:** [value with source, or "Not available — flagged for follow-up"]
    - **Comp set occupancy:** [value with source, or "Not available — flagged for follow-up"]
    - **RevPAR signals:** [value or flagged]
    - **Demand drivers:** [summary]
    - **Seasonality:** [summary]
    - **Revenue streams:** [list]
    - **Market notes:** [summary of available benchmarks and their sources]

---

### Section 10 — Commercial Terms

Draft from Phase D commercial research results and the owner's existing portfolio from seed data. Flag assumptions explicitly.

**Artifact format:**

    ## Commercial Terms
    [Drafted content]

---

### Section 11 — Design Direction

Read `references/kh-brands.md` for brand-specific design language for the confirmed brand. Derive from confirmed Concept Direction and brand guidelines.

**Artifact format:**

    ## Design Direction
    [Drafted content]

---

### Section 12 — Phasing _(optional)_

Include only if deal type or asset state from seed data indicates a phased build or opening. Skip if not applicable.

**Artifact format:**

    ## Phasing
    [Drafted content]

---

### Section 13 — Partners & Collaborators _(optional)_

Include only if Section 6 Concept Direction points to specific partnership opportunities. Skip if not applicable.

**Artifact format:**

    ## Partners & Collaborators
    [Drafted content]

---

### Section 14 — Insider Knowledge & Relationships _(optional)_

If insider knowledge was provided in seed data, record it here. This section cannot be derived — write what was shared, lightly structured. Skip if nothing was provided.

**Artifact format:**

    ## Insider Knowledge & Relationships
    [What was shared]

---

### Section 15 — Open Questions for Client _(optional)_

Surface any gaps from the above sections that require client input before the pipeline can run. Skip if there are none.

**Artifact format:**

    ## Open Questions for Client
    - [Question]

---

## Decisions Made

When all applicable sections are written to the artifact, generate and display a summary card as inline HTML/CSS.

Read `references/ui-style.md` for visual style. This is a display-only card — apply card shell, typography, layout, and flagged-value guidelines.

Card title: `DECISIONS MADE`

One row per section — label on the left, concise one-line decision on the right:

- Positioning: [one-line thesis]
- Concept: [vision headline]
- Key messages: [count] messages — [dominant theme, 3–4 words]
- F&B: [brand names]
- Wellness: [register, 4–6 words]
- Experiences: [names with _(precedent)_ / _(original)_ labels]
- Financial: [ADR range or _flagged_] · [occupancy or _flagged_]
- Commercial: [key structure, 4–6 words]
- Design: [design language, 4–6 words]

Footer — below a 1px divider, in muted 12px text:
`Full brief: [concept-brief-name].md — open in right-hand pane`

After the card, proceed immediately to the Final Brief.

---

## Final Brief — HTML Artifact

Read `references/ui-style.md` for document layout guidelines. Generate a self-contained HTML file using the Document Layout section — not the card or form style.

**File name:** `concept-brief-[property-name].html` — same slug as the markdown artifact.

The document contains all confirmed sections in schema order, from Property Metadata through to whichever optional sections were included. Render each section using the section block pattern from ui-style.md. Use the markdown artifact as the source of truth for confirmed content.

Write the file to the working directory. Once written, tell the user:
*"Your full concept brief is saved as `[filename].html` — open it to review the complete document. Let me know if you'd like to change anything. When you're ready, run `/submit-brief` to send it to the pipeline."*

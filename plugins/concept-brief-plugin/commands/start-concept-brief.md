Start a Concept Brief session for Kerten Hospitality. Follow these steps exactly.

## 1. Display the session overview

Present the following to the user — keep it tight, no preamble:

---

**Concept Brief — Session Overview**

I'll start by asking you to fill in a short form — 11 fields covering the property, owner, and asset. Once submitted, I'll run market and owner research, then auto-draft all remaining sections for you to review and confirm. Everything gets saved to a running artifact file (`concept-brief-[property-name].md`) in this directory. You can stop at any time and resume later — just run `/start-concept-brief` again and I'll pick up where we left off.

You can view our progress of the information we save on the right pane.

**What I'll ask you:**

| # | Section | |
|---|---|---|
| 1 | Property Metadata | Required |
| 2 | Owner Context | Required |
| 3 | Asset Assessment | Required |
| 4 | Insider Knowledge & Relationships | Optional |

All remaining sections — Competitive Set & Market Dynamics, Positioning Thesis, Concept Direction, Key Messages, Experience Inventory, Financial Anchors, Commercial Terms, Design Direction, Phasing, Partners & Collaborators, and Open Questions — are auto-drafted from your answers and research, then presented to you for review.

---

## 2. Display the intake form

Read `references/ui-style.md` for visual style. Generate and display as inline HTML/CSS. Apply card shell, typography, layout, and interactive element guidelines.

Group short fields on the same line — do not stack every field vertically.

**Property** _(one row)_
- Property name * — free text
- Location * — free text
- Keys * — number

**Brand** * — button selection: `Cloud 7` · `The House Hotel` · `Hosme` · `TBD`

**Deal type** * — button selection: `Management contract` · `Franchise` · `Lease` · `JV` · `Other`

**Owner** _(side by side)_
- Primary motivation * — free text
- Red lines / dealbreakers — free text
- Existing portfolio — free text

**Asset**
- Current state * — button selection: `Greenfield` · `Under construction` · `Shell complete` · `Full renovation` · `Soft refurbishment` · `Operational`
- Standout physical features — free text

**Insider Knowledge & Relationships** _(optional)_ — free text

If the user uploaded an RFP or brief document: pre-populate every field you can answer confidently from it, leaving blanks only where the document is silent. Mark pre-populated fields clearly so the user knows what was extracted.

Once submitted, hand off to the concept-briefing skill to run research and auto-draft all sections.

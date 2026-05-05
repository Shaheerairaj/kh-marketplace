Start a Concept Brief session for Kerten Hospitality. Follow these steps exactly.

## 1. Check for an existing artifact

Look in the current working directory for any file matching `concept-brief-*.md`.

- If one or more files are found: list them and ask the user "Would you like to resume one of these, or start a new brief?"
  - If resuming: read the file, identify the last confirmed section, tell the user where you are picking up from, and begin from the next section. Skip steps 2 and 3 below.
  - If starting new: proceed to step 2.
- If no file is found: proceed to step 2.

## 2. Display the session overview

Present the following to the user — keep it tight, no preamble:

---

**Concept Brief — Session Overview**

I'll start by asking you to fill in a short form — 10 fields covering the property, owner, and asset. Once submitted, I'll run market and owner research, then auto-draft all remaining sections for you to review and confirm. Everything gets saved to a running artifact file (`concept-brief-[property-name].md`) in this directory. You can stop at any time and resume later — just run `/start-concept-brief` again and I'll pick up where we left off.

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

## 3. Display the intake form

Generate and display as inline HTML/CSS. Visual style — flat card, white background (#ffffff), 1px solid border (#e2e8f0), border-radius 8px, 16px horizontal padding, 12px vertical padding, max-width 560px, system sans-serif font. Section headers: 11px uppercase, letter-spacing 0.05em, color #94a3b8. Field labels: 12px, color #64748b. Input text: 13px, color #1e293b, font-weight 500. Input fields: 1px solid #e2e8f0, border-radius 6px, padding 8px 10px. Toggle buttons: 1px solid #e2e8f0, border-radius 6px, padding 6px 12px, 13px; selected state: background #1e293b, color #ffffff, border-color #1e293b. Row gap: 12px between fields. Group dividers: 1px solid #f1f5f9, 8px margin above and below. Submit button: full width, background #1e293b, color #ffffff, border-radius 6px, padding 10px, 13px, font-weight 500.

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

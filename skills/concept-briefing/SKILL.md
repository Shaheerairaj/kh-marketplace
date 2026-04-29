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
Read `references/sections/01-property-metadata.md` for field definitions, proposal guidance, and artifact format.
Read `references/kh-brands.md` to inform the Brand Fit Hypothesis field.

### Step 2 — Positioning Thesis
Read `references/sections/02-positioning-thesis.md` for proposal guidance and quality bar.
Read `references/kh-brands.md` if not already in context.

---

_Steps 3–15 will be added as subsequent sections are built out._

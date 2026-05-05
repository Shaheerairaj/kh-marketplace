---
name: ui-style
description: Visual style reference for all inline HTML/CSS cards and forms in the concept briefing plugin. Read this before generating any card or form.
---

## Design Principles

Flat, clean, minimal. White background, subtle borders, slightly rounded corners, constrained width (~560px max), system sans-serif font. No shadows or decorative elements — cards and forms should feel like structured documents.

## Color Palette

| Role | Color |
|---|---|
| Card background | #ffffff |
| Card / input border | #e2e8f0 |
| Section headers | #94a3b8 |
| Field labels | #64748b |
| Primary text / values | #1e293b |
| Group dividers | #f1f5f9 |
| Muted / flagged / empty values | #94a3b8 |
| Selected state / submit background | #1e293b |
| Selected state / submit text | #ffffff |

## Card Shell

White background (#ffffff), subtle border (#e2e8f0), slightly rounded corners, max width ~560px, system sans-serif. No shadow.

## Typography

- **Section headers** — small, uppercase, wide letter-spacing, muted (#94a3b8)
- **Field labels** — small, slate (#64748b)
- **Values / input text** — slightly larger than labels, dark (#1e293b), medium weight
- **Muted / flagged / empty values** — italic, muted (#94a3b8); blank optional fields use an em-dash

## Layout

- Consistent vertical gap between fields
- Group dividers: thin line (#f1f5f9) with small vertical margin above and below

## Interactive Elements

_Apply only to forms with inputs — not to display-only cards._

- **Input fields** — subtle border (#e2e8f0), rounded corners
- **Toggle buttons** — subtle border (#e2e8f0), rounded; selected state: dark fill (#1e293b), white text (#ffffff), matching border
- **Submit button** — full width, dark background (#1e293b), white text (#ffffff), rounded, medium weight

---

## Document Layout

_Apply when generating the final HTML brief artifact — not for inline cards or forms._

### Page Shell

Self-contained HTML file with all CSS embedded in a `<style>` block. No external dependencies. Max width ~800px, centered, white background (#ffffff), system sans-serif font, comfortable line height. Body background: light (#f8fafc) to distinguish the page from the document surface.

### Header Block

At the top of the document body:
- Property name as the primary heading — large, dark (#1e293b), medium weight
- Metadata row below: brand · deal type · location · date generated — small, muted (#64748b), separated by mid-dots
- A thin bottom border (#e2e8f0) below the header to separate it from the sections

### Section Blocks

Each Direction Document section renders as a block with:
- Section heading — uppercase, small, wide letter-spacing, muted (#94a3b8); acts as a label, not a title
- Content below the heading — dark (#1e293b), normal weight, readable line height
- A subtle bottom border (#f1f5f9) between sections

### Content Formatting

- Lists render as standard `<ul>` / `<li>` with tight spacing
- Bold labels (e.g. **Comp set ADR range:**) use dark (#1e293b), medium weight
- Flagged or unavailable values: italic, muted (#94a3b8)
- Sub-headings within a section (e.g. Vision, Pillars): slightly larger than body, dark (#1e293b), medium weight

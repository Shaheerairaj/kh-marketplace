## 1. Context & Problem

Kerten Hospitality's Business Development team produces hotel management proposals for prospective property owners. Each proposal is a custom PowerPoint deck (typically 40–80 slides) covering concept positioning, market analysis, experience design, financial projections, and commercial terms. A typical proposal currently takes 2–4 weeks of senior BD time, with significant duplication across deals: market research, financial modelling, image sourcing, slide layout, and copy generation are all redone from scratch.

The bottleneck is not a single step — it is the assembly. Antony Douce (CXO) generates concept direction in long-form conversation; BD translates this into a deck; Karolina builds financials; design assets are sourced manually from Pinterest; templates are copy-pasted from prior decks. The work is reproducible in structure but creative in content, which makes it a strong fit for AI-assisted assembly with human approval gates.

The system described in this document compresses the proposal lifecycle from weeks to days by separating creative direction (which stays human) from assembly (which becomes automated), with explicit human approval points at each handoff.

---

## 2. Goals & Non-Goals

### 2.1 Goals

- Reduce proposal turnaround from 2–4 weeks to 2–4 days for standard properties.
- Capture concept direction through a guided conversation rather than free-form email, producing a structured, machine-readable Direction Document.
- Generate a first-draft outline automatically from the Direction Document, surface it for BD review, and proceed to full document generation only once approved.
- Generate slide copy, financial projections, sourced images, and an assembled PowerPoint deck automatically from the approved outline.
- Preserve human approval gates at each consequential handoff (Direction Document submission, outline approval, financial assumptions sign-off).
- Standardise on Kerten's brand templates (The House, Hosme, Cloud 7) while allowing controlled structural customisation.

### 2.2 Non-Goals (v1)

- Free-form per-slide prompting — users cannot describe an arbitrary slide and have the system build it. Slide types are constrained by the chapter library.
- User-selected slide templates — the Assembly Agent picks templates deterministically based on slide metadata and chapter library rules.
- Custom chapters outside the controlled vocabulary — adding new chapter types is a config change, not a user action.
- Post-generation in-tool copy editing — once `proposal.pptx` is exported, edits happen in PowerPoint. Creative refinement by Lily/Marina remains a downstream step.
- Inline image replacement in the UI — images are placed automatically into template placeholders; BD adjustments happen post-export in PowerPoint.
- The system handles all three proposal archetypes (complex/bespoke, standard/consultant-driven, automatic/teaser) through the same pipeline — no separate code paths per type in v1.

---

## 3. Proposed Solution

The system is composed of three phases, each ending with a human approval gate before the next begins.

**Phase 1 — Concept Briefing (Claude Desktop)**
User holds a guided conversation with Claude inside the Claude Desktop app. Two custom skills drive the conversation: `/market-research` pulls external intelligence (comp set data, market news, owner background); `/submit-brief` validates the conversation output for completeness and synthesises it into a structured Direction Document, which is the sole input to Phase 2.

**Phase 2 — Outline Generation (Python pipeline + Web UI)**
The FastAPI backend receives the Direction Document and calls the Outline Agent, which generates a chapter-and-slide structure. The BD team reviews titles only (no copy) in the Outline Review UI, approving or flagging slides. Flagged slides trigger a partial re-run of the Outline Agent. Only after full BD approval does the system proceed to Phase 3.

**Phase 3 — Document Generation (Python pipeline)**
Three agents run in parallel from the approved outline: the Copy Agent writes slide content, the Financial Agent queries Lighthouse for available market benchmarks and populates an Excel template, and the Image Sourcing agent calls multiple Apify actors (Pinterest, Google Images, and others) and downloads images with their metadata directly into the proposal. Once all inputs are ready, the Assembly Agent assembles the final PowerPoint using the correct brand template. Outputs are packaged into a delivery folder.

```
┌─────────────────────┐
│  Phase 1            │  Claude Desktop
│  Concept Briefing   │  /market-research + /submit-brief skills
└────────┬────────────┘
         │ Direction Document
         ▼
┌─────────────────────┐
│  Phase 2            │  Python pipeline + Web UI
│  Outline Generation │  FastAPI → Outline Agent → BD approval
└────────┬────────────┘
         │ Approved Outline
         ▼ [Human checkpoint: BD team approves chapter + slide structure]
┌─────────────────────┐
│  Phase 3            │  Python pipeline
│  Document Generation│  Copy + Financial + Images (parallel) → Assembly
└────────┬────────────┘
         │
         ▼
   Delivery Folder
   ├── proposal.pptx
   ├── financial_model.xlsx
   ├── concept_brief.pdf
   └── image_bank/
```

Key principle: Phase 1 runs on the user's desktop in Claude Desktop. Phases 2 and 3 run on the backend with a thin web UI for approval interactions. The Direction Document is the only handoff between Phase 1 and the pipeline.

---

## 4. Detailed Design

### 4.1 Phase 1 — Concept Briefing in Claude Desktop

**Runtime:** Claude Desktop (macOS/Windows), via an installable Claude Code plugin. The plugin is a `.claude-plugin` directory with four component types: skills (auto-selected by Claude based on their description field), slash commands (explicit user triggers), agents (sub-agents for defined sub-tasks), and an MCP config (`.mcp.json`). All component definitions are markdown; plugin metadata is JSON. The plugin is hosted as a GitHub repository; users install it by URL in Claude Desktop.

**Entry point:** User opens a Claude Desktop session. Claude guides the conversation section by section through the Direction Document schema — proposing content for each section, validating with the user, saving confirmed sections to a running artifact, then advancing to the next. The user approves or steers at each step; Claude does not wait for free-form input across the whole brief before structuring output.

**Exit point:** Direction Document reviewed and approved  before submission to the pipeline.

#### Plugin Components

##### Skills (auto-selected)

Skills are invoked automatically by Claude when their description matches the current conversational need — no explicit user trigger.

*TBD:* The mechanism for pulling external market intelligence mid-conversation (Lighthouse, web search, LinkedIn, Apollo/Clay) is still being designed. The requirement is that at the right moment in the guided section-by-section discussion (e.g., when reaching Competitive Set or Financial Anchors), Claude gathers the necessary external data without breaking the conversation flow. Whether this is implemented as an auto-selected skill or an explicit slash command is an open design decision.

##### Slash Commands (explicit triggers)

**`/submit-brief`** — triggered by the user at the end of the guided discussion:
- Hands off to a summarizer agent that distils the full conversation into only the context needed for the Direction Document (see Agents below)
- Validator agent checks all required fields are populated; surfaces optional gaps to the user for a final review
- On confirmation, writes the structured Direction Document (markdown file) to the agreed file storage location (see Open Question #1) and triggers the Phase 2 pipeline run

##### Agents

Sub-agents are defined in the plugin to handle context management at the submit point. Theo raised a concern (2026-04-28 transcript) that a long multi-source research conversation will accumulate substantial context — scraper outputs, search results, intermediate reasoning — and that the Direction Document generation step should start with clean context. Shaheer's response: Claude's built-in compaction already prunes irrelevant tool calls and errors as context fills, and Claude internally spawns its own sub-agents in long sessions before explicit compaction kicks in. However, at the `/submit-brief` step, two sub-agents can be explicitly invoked:

- **Summarizer agent** — reads the full conversation and distils it to only the information needed to populate the Direction Document schema. Output is a structured summary, not a transcript.
- **Validator agent** — receives the summarizer output and the schema, checks required fields, surfaces optional gaps. Output drives the user-facing gap-fill prompt before final submission.

This is not a first-build requirement but is designed into the plugin architecture so it can be enabled without restructuring.

##### MCP Config

`.mcp.json` in the plugin root defines the MCP server configuration. External data sources (Lighthouse, web fetch, web search, Apollo/Clay) are wired here. The MCP server is bundled with the plugin — no separate server hosting required for Phase 1.

#### Direction Document Schema (v1)

The Direction Document is stored as structured Markdown with defined H2 sections. Required vs. encouraged vs. optional sections are enforced by the validator at submit time — not by the chat UI. Antony converses naturally; Claude extracts and probes for missing sections.

| Section | Tier | Notes |
|---|---|---|
| Property Metadata | Required | Name, location, keys, owner, positioning tier, brand fit hypothesis |
| Positioning Thesis | Required | One-sentence argument for why Kerten wins this deal |
| Concept Direction | Required | Vision, pillars, target guest, differentiators |
| Owner Context | Required | Priorities, pain points, known preferences, what they are avoiding |
| Key Messages / Proof Points | Required | Pitch arguments to hammer |
| Experience Inventory | Required | F&B, wellness, signature experiences |
| Competitive Set & Market Dynamics | Required | Free-text market thesis + optional benchmark hotels (~5) |
| Asset Assessment | Required | Strengths, weaknesses, opportunities of the physical property |
| Financial Anchors | Required | Comp set benchmarks sourced from Lighthouse; ADR and occupancy are not available via Lighthouse and must be manually entered or flagged as absent |
| Commercial Terms | Required | Management fee, royalty, and other fee levels; AI suggests starting position based on market context and positioning thesis; BD tunes |
| Design Direction | Required | Aesthetic references, design language, mood cues |
| Phasing | Optional | Timeline proposal — only if known |
| Partners & Collaborators | Optional | Third parties to include (design studios, F&B operators, brand partners) |
| Insider Knowledge & Relationships | Optional | Personal market knowledge, owner relationships — shapes proposal tone |
| Open Questions for Client | Optional | Critical-path items still pending from client |

Schema is versioned (`direction_doc_schema_v1`). Schema changes require a new version and a migration path for in-flight runs.

#### Key Design Decisions

- The Direction Document is the only handoff between Phase 1 and the pipeline. It must capture everything the user communicates in the conversation — all downstream agents are driven from it with no further human input (other than changes to the outline).
- There is no CoStar API access. Lighthouse is the source for available market benchmarks; the mechanism for pulling this data mid-conversation is TBD (see Plugin Components above). Lighthouse does not provide ADR or occupancy data — those fields require manual input from the user. The Financial Agent in Phase 3 also integrates with Lighthouse under the same constraint (see ADR-011).
- The plugin is a `.claude-plugin` directory hosted as a GitHub repository. It contains four component types: skills, slash commands, agents, and an MCP config. "Marketplace" is the Claude Desktop UI label for the URL-based install flow — there is no formal listing or registry.
- Doc is sent to SharePoint table to keep track of all concept briefs triggered.


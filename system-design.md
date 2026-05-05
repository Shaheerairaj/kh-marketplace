TECHNICAL DESIGN DOCUMENT
Kerten Hospitality
Proposal Generation System
Version 1.1 — Working Draft

Author
Shaheer Ahmed
Reviewer
Theo Breward
Status
Draft — In Review
Last updated
4 May 2026
Build start
Week of 28 April 2026 (partial — see §10)
Target delivery
5–6 weeks from build start
Client meeting
Friday 1 May 2026 — present solution architecture only

---

## 1. Context & Problem

Kerten Hospitality's Business Development team produces hotel management proposals for prospective property owners. Each proposal is a custom PowerPoint deck (typically 40–80 slides) covering concept positioning, market analysis, experience design, financial projections, and commercial terms. A typical proposal currently takes 2–4 weeks of senior BD time, with significant duplication across deals: market research, financial modelling, image sourcing, slide layout, and copy generation are all redone from scratch.

The bottleneck is not a single step — it is the assembly. Antony Douce (CXO) generates concept direction in long-form conversation; BD translates this into a deck; the revenue team builds financials; design assets are sourced manually from Pinterest; templates are copy-pasted from prior decks. The work is reproducible in structure but creative in content, which makes it a strong fit for AI-assisted assembly with human approval gates.

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

| Section | Tier | Input Method | Notes |
|---|---|---|---|
| Property Metadata | Required | Human Q&A — 5 questions | (1) Name (2) Location — city/country and where in city (3) Keys (4) Brand — The House / Hosme / Cloud 7, or positioning tier if undecided (5) Deal type — new build / conversion / existing operation |
| Owner Context | Required | Human Q&A — 3 questions | (1) Owner's primary motivation for seeking a management partner (2) Known red lines or dealbreakers (3) Other properties and which brands they're with |
| Competitive Set & Market Dynamics | Required | AI-drafted | Derived from location + positioning tier via research |
| Asset Assessment | Required | Human Q&A — 2 questions | (1) Current state — greenfield / under construction / existing building / renovation (2) Standout physical features of the site |
| Positioning Thesis | Required | AI-drafted | Derived from metadata + owner context + comp set |
| Concept Direction | Required | AI-drafted | Derived from positioning + brand guidelines |
| Key Messages / Proof Points | Required | AI-drafted | Derived from owner context + positioning thesis |
| Experience Inventory | Required | AI-drafted | Derived from concept direction + property scale (keys) |
| Financial Anchors | Required | AI-drafted | Derived from comp set research; ADR and occupancy unavailable via Lighthouse — flagged for manual input |
| Commercial Terms | Required | AI-drafted | Derived from market context + owner's existing portfolio |
| Design Direction | Required | AI-drafted | Derived from concept direction + brand guidelines |
| Phasing | Optional | AI-drafted | Derived from deal type + RFP if available |
| Partners & Collaborators | Optional | AI-drafted | Derived from concept direction |
| Insider Knowledge & Relationships | Optional | Human — open-ended closer | Cannot be derived. Prompt: *"Anything else about this owner, this market, or how this deal came about that I should know?"* |
| Open Questions for Client | Optional | AI-drafted | Surfaces from gaps in gathered information |

Schema is versioned (`direction_doc_schema_v1`). Schema changes require a new version and a migration path for in-flight runs.

#### Key Design Decisions

- The Direction Document is the only handoff between Phase 1 and the pipeline. It must capture everything the user communicates in the conversation — all downstream agents are driven from it with no further human input (other than changes to the outline).
- There is no CoStar API access. Lighthouse is the source for available market benchmarks; the mechanism for pulling this data mid-conversation is TBD (see Plugin Components above). Lighthouse does not provide ADR or occupancy data — those fields require manual input from the user. The Financial Agent in Phase 3 also integrates with Lighthouse under the same constraint (see ADR-011).
- The plugin is a `.claude-plugin` directory hosted as a GitHub repository. It contains four component types: skills, slash commands, agents, and an MCP config. "Marketplace" is the Claude Desktop UI label for the URL-based install flow — there is no formal listing or registry.
- Doc is sent to SharePoint table to keep track of all concept briefs triggered.
- **Section ordering:** The guided conversation follows the schema table order — Owner Context, Competitive Set, and Asset Assessment come before Positioning Thesis and Concept Direction. Market and owner context must be established first; positioning built on incorrect initial assumptions compounds through all subsequent sections and is difficult to correct.
- **Two workflow paths — with-RFP and without-RFP:** When the user uploads an RFP or brief document at session start, the tool extracts property metadata and available context directly from it and skips redundant questions. Without an RFP, the full guided Q&A applies. With-RFP is expected to be the primary path for most Kerten proposals.
- **Seed sections vs. derived sections:** Only three sections require active human Q&A — Property Metadata (5 questions), Owner Context (3 questions), and Asset Assessment (2 questions), totalling 10 questions. All remaining sections are AI-drafted from those inputs and validated by the user in a single pass. Insider Knowledge & Relationships is a special case: it cannot be derived but is handled via a single open-ended prompt rather than structured questions. The goal is to complete the entire Q&A phase in under 5 minutes.
- **Intermediate forms (2–3 mid-flow checkpoints):** After the initial intake form, the flow pauses at 2–3 key decision points — concept direction, segment definition, and pricing hypothesis. Claude surfaces its hypothesis pre-populated in an editable form field; the user adjusts inline and submits before generation continues. These are not chat dialogues — form fields. Without them, bulk-generation produces shallower output because there is no forcing function for user course-correction at the moments that determine brief quality.
- **Per-section context engineering:** Each section that requires external data (e.g. ADR benchmarking) must carry an explicit instruction to take time and do thorough web research. Without this, bulk-generation mode skips research steps that the model would otherwise run in a slower iterative session.
- **Past proposals as context:** Converted to plain-text markdown and injected as context (~50 max). For creative or one-off concepts, the model's own training data is preferred over past proposal retrieval — over-grounding in precedent can suppress novelty by up-weighting common patterns.

---

### 4.2 Phase 2 — Outline Generation

**Runtime:** Python pipeline (FastAPI backend) + Outline Review UI (React).

**Entry point:** Power Automate calls `POST /proposals` when the concept brief is saved to SharePoint. Not user-initiated — the trigger is automated.

**Exit point:** BD team approves chapter and slide structure in the Outline Review UI.

#### Outline Agent

- **Model:** Claude Sonnet 4.6 (sufficient for structured generation).
- **Input:** Direction Document + Chapter Library YAML.
- **Output:** 2-level outline — chapters → slide titles + slide type tag. No copy at this stage.
- **Constraint:** must select chapters from the controlled vocabulary (Chapter Library).
- **Re-runs on BD flags:** when BD flags a slide or chapter in the UI, the agent re-runs only the affected scope, preserving previously approved sections. Merge logic lives in the FastAPI backend, not a separate agent.

#### Outline Review UI

A React web app backed by FastAPI. Realtime updates when the Outline Agent finishes, the UI updates without polling.

**Capabilities:**
- Left panel: collapsible chapter tree displaying all chapters and slide titles
- Right panel: slide detail view (title, type, data requirements)
- Per-slide actions: Approve / Flag / Edit title
- Per-chapter actions: Approve all / Reorder slides / Add slide / Remove slide
- Add/remove chapters from the Chapter Library (dropdown, controlled vocabulary only)
- "Approve & Generate" button — triggers Phase 3 once all slides are approved
- Run status panel — live agent progress, error states, manual retry buttons

**Out of scope (UI v1):**
- Inline copy editing per slide
- Template selection (deterministic, not user-controlled)
- Free-form slide prompting

#### Key Design Decisions

- BD reviews **titles only, not copy** — showing generated content before structure is agreed causes rework and loss of trust in the system.
- BD can restructure chapters and add/remove slides but **cannot re-prompt individual slides** — this prevents scope creep and keeps the outline coherent.
- Flagged slides trigger a **partial re-run** only — full regeneration is not triggered by a single flag.

---

### 4.3 Phase 3 — Document Generation

**Runtime:** Python pipeline.

**Entry point:** Approved Outline from Phase 2.

**Exit point:** Delivery folder assembled and handed off to BD.

Three agents run in parallel immediately after outline approval, then feed into the Assembly Agent.

#### Copy Agent

- **Model:** Claude Sonnet 4.6.
- **Input:** Approved Outline + Direction Document.
- **Output:** Slide copy (title + body text) for every non-financial slide, constrained by each template's text slot specification.
- Copy is generated against the approved slide structure — not re-prompted per-slide.
- Additional Context: High level overview of the chapters and slides of previous KH proposals.

#### Financial Agent

- **Model:** Claude Sonnet 4.6 + tool calls.
- **Tools:** Lighthouse (market benchmarks), openpyxl (Excel template population).
- **Workflow:** receive revenue stream list from Direction Document → query Lighthouse for available market benchmarks → generate assumptions with cited reasoning, explicitly flagging where ADR and occupancy data are absent and must be manually provided → populate Excel template with a clear inputs tab and assumptions list.
- (Optional)**Approval gate:** The revenue team validates assumptions, including any fields left blank due to Lighthouse data limitations. Only after `FinancialModel.validated == True` does the Assembly Agent insert the financial slide. Users can upload a manually revised Excel to override auto-population.
- **Failure:** Lighthouse unavailable → flag all market assumptions as unverified and surface for revenue team review.
- **Data limitation:** Lighthouse does not provide ADR or occupancy data. The financial model will have explicit gaps for these fields unless the BD team supplies them manually (see ADR-011).

#### Image Sourcing

- **Tools:** Multiple Apify actors (Pinterest, Google Images, and others). Each actor returns image metadata alongside the raw image — including source URL, descriptive tags, dimensions, and contextual keywords. This metadata is stored with each image and made available to downstream AI agents for image selection decisions (e.g., filtering by aspect ratio, theme match, or brand appropriateness).
- **Workflow:** generate per-slide thematic queries from the Direction Document concept direction (e.g., lobby, F&B, rooms, exterior, lifestyle) → invoke Apify actors across Pinterest, Google Images, and other configured sources → download images and metadata to `/tmp/{proposal_id}/image_bank/{slide_id}/` → pass local file paths and metadata to the Assembly Agent for direct placement. No BD approval gate — images are placed automatically.
- **Post-generation reference:** all downloaded images are uploaded to SharePoint `delivery/image_bank/` alongside `proposal.pptx`. BD can review what was placed and swap images post-export in PowerPoint.
- **Failure:** Apify actor(s) return fewer images than needed → reformulate query and retry across all configured sources (max 2 retries) → placeholder inserted (see §7).

#### Assembly Agent

- **Model:** Claude Sonnet 4.6 + python-pptx.
- **Input:** Approved Outline + Copy Agent output + validated financial data + sourced images.
- **Per slide:** match template via metadata tags → insert copy into text slots → insert images at placeholder positions → apply brand styles.
- **Brand templates:** The House, Hosme, Cloud 7. Cloud 7 is on hold until new brand identity is released (expected mid-May 2026); build starts with The House and Hosme.
- **Deterministic on layout:** the agent does not invent layouts. Templates are pre-tagged in the Template Knowledge Base and selected by metadata match, not generation.
- **Output:** `proposal.pptx` written to delivery folder. Pixel-perfect creative refinement by Lily/Marina remains a downstream step.

#### Output Packaging

All outputs collected into a single delivery folder:
- `proposal.pptx` — fully assembled brand-compliant deck
- `financial_model.xlsx` — validated financial model with assumptions tab
- `concept_brief.pdf` — rendered Direction Document for human reference
- `image_bank/` — all sourced and approved images

---

## 5. Data Contracts

The system holds no application database. Inter-phase state is carried by files in SharePoint; intra-phase state lives in memory for the duration of a Modal function call. Agents communicate via typed Pydantic models passed as function arguments and return values — no shared state store. SharePoint folder layout is defined in `system-architecture.md` §6.

### 5.1 Direction Document (Phase 1 → Phase 2 input)

Stored as structured Markdown at `/Proposals/{deal-id}_{property-name}/direction.md` in SharePoint. Parsed to JSON by the pipeline on ingest. Schema versioned via a `schema_version` front-matter field on the Markdown file.

```yaml
# Front-matter on direction.md
schema_version: direction_doc_schema_v1
property_name: "..."
submitted_by: "anthony.douce@kerten.com"
submitted_at: "2026-04-29T14:32:00Z"
status: submitted   # draft | submitted | superseded
```

Body content follows the schema in §4.1.

### 5.2 Parsed Direction (pipeline internal)

Structured JSON derived from the Direction Document — what the sub-agents consume.

```json
{
  "property": { "name": "...", "keys": 120, "tier": "upper-upscale", "location": "..." },
  "positioning_thesis": "...",
  "required_chapters": ["ch_cover", "ch_concept", "ch_market", "..."],
  "experiences": [{ "name": "...", "category": "fnb|wellness|signature", "description": "..." }],
  "financial_inputs": { "revenue_streams": ["..."], "comp_set_hint": ["..."] },
  "design_direction": { "references": ["..."], "language": "..." },
  "key_messages": ["...", "..."]
}
```

### 5.3 Outline (Outline Agent output)

In-progress outlines live in memory in the React UI during the BD review session. On final approval, FastAPI serialises the outline as JSON and writes it to SharePoint at `/Proposals/{deal-id}_{property-name}/outline-approved.json` — this file is the Phase 2 → Phase 3 handoff.

```json
{
  "schema_version": "outline_schema_v1",
  "proposal_id": "...",
  "status": "approved",
  "version": 1,
  "chapters": [
    {
      "chapter_id": "ch_cover",
      "title": "...",
      "slides": [
        { "slide_id": "...", "title": "...", "slide_type": "...", "position": 1 }
      ]
    }
  ],
  "approved_at": "2026-05-02T11:14:00Z",
  "approved_by": "sara@kerten.com"
}
```

If a review session is interrupted before approval, the outline is regenerated from the Direction Document on next open — there is no persisted draft state.

### 5.4 Chapter Library (config, not data)

YAML in repo. Each entry: `id`, `name`, `tier`, `default_slides`, `max_slides`, `required_predecessors`, `allowed_slide_types`. See §6.1.

### 5.5 Image Assets (per slide)

Images are downloaded to Modal's ephemeral filesystem at `/tmp/{proposal_id}/image_bank/{slide_id}/` during the Image Sourcing Agent run, passed directly to the Assembly Agent as file paths, and then uploaded to SharePoint `delivery/image_bank/` as part of output packaging. No approval state is tracked.

### 5.6 Financial Model

The Financial Agent writes the populated workbook to Modal's ephemeral filesystem at `/tmp/{proposal_id}/financial_model.xlsx`, surfaces assumptions to the financial reviewer via the UI, and on validation uploads the final `.xlsx` to SharePoint `delivery/financial_model.xlsx`. The validation state is held in memory for the duration of the Phase 3 run; the validated file in SharePoint is the durable artefact.

```json
{
  "schema_version": "financial_model_schema_v1",
  "proposal_id": "...",
  "excel_path": "/Proposals/{deal-id}_{property-name}/delivery/financial_model.xlsx",
  "assumptions": [
    { "key": "adr_year_1", "value": "...", "rationale": "..." }
  ],
  "validated": true,
  "validated_by": "karolina@kerten.com",
  "validated_at": "2026-05-04T09:20:00Z",
  "override_path": null
}
```

---

## 6. Knowledge Bases & Configuration

### 6.1 Chapter Library

The controlled vocabulary that constrains the Outline Agent. Approximately 20 chapter types defined — the agent must select from this list.

| Chapter ID | Name | Tier | Slide Range | Notes |
|---|---|---|---|---|
| ch_cover | Cover | Fixed | 1 | Always first |
| ch_exec_summary | Executive Summary | Fixed | 1–2 | |
| ch_vision | Vision | Fixed | 1–2 | Can split vision + mission |
| ch_concept | Concept Overview | Fixed | 1–3 | |
| ch_market | Market & Positioning | Fixed | 2–4 | |
| ch_experiences | Signature Experiences | Fixed | 2–8 | Summary or per-experience |
| ch_fnb | F&B Concepts | Default-on | 1–6 | |
| ch_wellness | Wellness | Default-on | 1–3 | |
| ch_design | Design Direction | Default-on | 1–4 | Moodboard slides |
| ch_case_studies | Case Studies | Optional | 0–5 | Only if requested |
| ch_partners | Partners | Optional | 0–3 | Noor Studio, brand partners |
| ch_team | Team & Operator | Fixed | 1–2 | |
| ch_phasing | Phasing & Timeline | Optional | 0–2 | If brief includes phasing |
| ch_financials | Financial Projections | Fixed | 2–3 | Revenue team-validated content |
| ch_commercial | Commercial Terms | Fixed | 1–2 | |
| ch_disclaimer | Disclaimer | Fixed | 1 | Always last |

This is v1 of the library. Antony, Sara, and BD leads must validate before build start. Adding new chapters post-build is a config-only change (no code).

### 6.2 Template Knowledge Base

`.pptx` slide templates, each tagged with metadata: layout type (cover, full-bleed, grid-N, text+image, chart, quote), image slot count and aspect ratios, text density (low/med/high), allowed chapter contexts, brand variant (The House / Hosme / Cloud 7). Templates live in the GitHub repo under `templates/{brand}/` and are baked into the Modal container image at deploy time; metadata is committed alongside each `.pptx` as a sibling YAML file. Updates are a PR + redeploy.

The Assembly Agent selects templates by metadata match against the slide's `slide_type` and chapter context. Users cannot override template selection — changes happen post-export in PowerPoint.

### 6.3 Brand Guidelines

PDF + structured assets (fonts, colour tokens, logo variants) per brand. Loaded as context to the Assembly Agent at generation time. Format TBC — see Open Question #9.

---

## 7. Failure Modes & Mitigations

| # | Component | Failure | Detection | Mitigation | Fallback |
|---|---|---|---|---|---|
| 1 | Apify actors | Returns < N images across all sources | Response count check | Reformulate query 2× across all actors | Placeholder inserted; flagged in delivery folder |
| 2 | Apify actors | Rate limit (429) | HTTP status | Exponential backoff | Pause + notify user |
| 3 | Lighthouse API | Unavailable / 5xx | Timeout / status | Retry 3× w/ backoff | Flag all market assumptions as unverified; surface for revenue team review |
| 4 | Outline Agent | Hallucinated chapter | Validator rejects | Re-prompt with allowed list | Hard fail → escalate |
| 5 | Outline Agent | Slide count mismatch | Cross-check after gen | Re-prompt with correction | Use list count, override stated |
| 6 | Financial Agent | Lighthouse data missing / ADR+occupancy absent | Empty response or known gap | Flag specific fields as absent; surface for revenue team review | Manual Excel upload override |
| 7 | Financial Agent | Revenue team rejects model | Approval status | Surface comments to BD | Manual Excel upload override |
| 8 | Copy Agent | Generated copy exceeds slot | Token / char count check | Re-prompt with length constraint | Truncate + flag |
| 9 | Assembly Agent | python-pptx write fails | Exception | Log slide id, skip | Generate without slide, flag |
| 10 | BD UI | User adds 12+ chapters | Count check | Warning at 8 | Hard cap at 12 |
| 11 | Pipeline | Crash mid-run | Modal function failure / timeout | Re-trigger phase from the last SharePoint artefact (Direction Document or Approved Outline) | Manual restart from UI |
| 12 | Phase 1 → Phase 2 | Direction Document incomplete on submit | Validator | List gaps + ask Antony to confirm | Allow override with warning |

---

## 8. Open Questions

These need to be closed before or during build. Each has an owner and a target close date.

| # | Question | Owner | Target |
|---|---|---|---|
| 1 | File storage / pipeline trigger — **RESOLVED:** Direction Document saved to SharePoint list (one row per proposal run). Power Automate triggers Phase 2 when row is saved — calls FastAPI backend → Outline Agent runs → BD team receives email with UI link. Douglas / Broad Vision access still required to configure SharePoint and Power Automate. | Theo + Douglas / Broad Vision | Closed |
| 2 | Hosting — where does the UI and pipeline run? Cloud (AWS / GCP) or Kerten IT infrastructure? | Theo | Tue 29 Apr (deep-dive scheduled) |
| 3 | Authentication — who can access the Outline Review UI? BD team only, or also Ramin and Antony? | Theo | Wed 30 Apr |
| 4 | Claude enterprise access — required for production; pending approval from Mina / Aman | Theo | Mon 28 Apr |
| 5 | ~~CoStar resolution~~ — **Closed (28 Apr).** No CoStar API access. System uses Lighthouse as the market data integration. Lighthouse does not provide ADR or occupancy data; those fields require manual input. See ADR-011. | — | Closed |
| 6 | Final Chapter Library — validate ~20 chapters with Antony, Sara, Marina before build start | Shaheer + Theo | Wed 30 Apr |
| 7 | Template inventory — how many templates exist today? What is the tagging effort for the Template Knowledge Base? | Sara + Marina | Wed 30 Apr |
| 8 | Brand guideline format — PDF, Figma, or structured design tokens? | Marina | Wed 30 Apr |
| 9 | Financial approval SLA — CFO + revenue team lead; what is the expected turnaround on financial assumptions review? | Theo | Wed 30 Apr |
| 10 | Experience slides — should experiences be one-per-slide, all-on-one, or configurable? Drives Assembly Agent template logic | Antony + Sara | Fri 2 May |
| 11 | Phase 1 output format — should `/submit-brief` also produce a 2-page human-readable Word brief in addition to the markdown Direction Document? | Theo | Fri 2 May |
| 12 | RFP-internal contradiction handling — when an uploaded RFP contains conflicting signals (e.g. "upper upscale" positioning vs. a low target ADR), should the tool flag and pause for human input, or self-resolve with explicit rationale? First full run (1 May) surfaced this as a real case. | Shaheer + Theo | TBD |
| 13 | Phase 1 time baseline — how long do Antony / senior BD currently spend on manual research and briefing before starting a proposal? Not yet measured. Needed to substantiate the "saves you hours on research" claim before presenting to Kerten. | Maria + Theo | TBD |

---

## 9. Milestones

Build is sequenced so that components with locked decisions start Monday 27 April; components with open decisions start later as decisions close. Total target: 5–6 weeks.

| Week | Workstream | Deliverables |
|---|---|---|
| Week 0 (27 Apr–1 May) | Decisions + scaffolding | Close open questions 1–9. Modal app + Vercel project + GitHub repo + CI. SharePoint `/Proposals/` folder structure agreed with Douglas. Direction Document validator. Chapter Library v1 YAML. Pinterest API exploration (1-day spike on day 1). |
| Week 1 | Phase 1 (Claude Desktop) | Custom MCP server. `/market-research` and `/submit-brief` skills. End-to-end submit flow (Claude Desktop → SharePoint `direction.md`). Test with one historical brief (Marrakech). **Status (May 4): ~80% complete. Remaining: intermediate forms + per-section context engineering. Target completion: EOD May 4.** |
| Week 2 | Phase 2: Outline Agent + Validator + UI | Outline Agent constrained by Chapter Library. Validator. Outline Review UI (chapter tree, approve/flag/edit). End-to-end Direction Document → approved outline. **Status (May 4): Starts May 5. Primarily UI work. Target completion: May 11.** |
| Week 3 | Phase 3: Copy Agent + Financial Agent | Copy Agent end-to-end. Lighthouse integration. Financial Agent + Excel generation (with explicit ADR/occupancy gap handling). |
| Week 4 | Phase 3: Image Sourcing + Assembly Agent | Pinterest integration. Auto-placement pipeline (download → Assembly → SharePoint image_bank). Template KB tagged. Assembly Agent end-to-end `proposal.pptx`. Output packaging. |
| Week 5 | Integration + dogfooding | Full end-to-end runs on 3 historical briefs (Marrakech, Apollo, Nile). Failure mode testing. Performance pass. |
| Week 6 (buffer) | Polish + handoff | Bug fixes from BD pilot. Documentation. Training session. Launch. |

**Critical path:** Lighthouse integration → Financial Agent (Week 3). CoStar is not available; Lighthouse is the confirmed market data source but does not provide ADR or occupancy — the financial model will have explicit gaps for those fields pending manual input from the revenue team. Apify actors exploration (Week 0) → Image Sourcing (Week 4). A 1-day spike in Week 0 characterises image output quality across all configured Apify actors and informs the query and metadata strategy for the auto-placement pipeline.

---

## 10. Architecture Decision Records

ADRs capture significant decisions in a permanent, dated record. Each ADR has a status (Accepted, Superseded, Deprecated) and is numbered sequentially. ADRs are not rewritten — when a decision changes, a new ADR supersedes the old one.

### ADR-001: Use Claude API + Python pipeline for orchestration

**Status:** Accepted — 26 April 2026

**Context:** We need to coordinate multiple LLM agents with tool use, sub-agent spawning, human approval pauses, and resumable state. Options considered: LangGraph, Claude Agent SDK, custom orchestrator with direct API calls.

**Decision:** Use the Anthropic Python SDK with a custom FastAPI orchestration layer. Native support for Anthropic models, tool calling, and structured outputs. FastAPI gives full control over approval pause logic and state resumption without framework overhead.

**Consequences:** Positive: full control over orchestration; first-class tool calling; clean fit with the stateless file-based design (ADR-010). Negative: more boilerplate than a framework; debugging multi-agent flows requires careful logging.

---

### ADR-002: Direction Document stored as Markdown, not JSON

**Status:** Accepted — 22 April 2026

**Context:** The Direction Document moves from Phase 1 (human-facing Claude Desktop session) to Phase 2 (machine-facing pipeline). It needs to be readable and editable by non-technical BD users, and parseable downstream.

**Decision:** Store as structured Markdown with defined H2 sections. Validator enforces section presence at submit time. The pipeline parses Markdown to JSON via deterministic parser before passing to agents.

**Consequences:** Positive: humans can read and edit; diffable in version control; Claude handles natively. Negative: parsing less strict than JSON; section names must be enforced. Mitigation: validator runs at `/submit-brief` time.

---

### ADR-003: Supabase for state, storage, and auth

**Status:** Superseded by ADR-010 — 26 April 2026

**Context:** Need a state store for runs, Direction Documents, outlines, image assets, financial models; object storage for images, templates, and output files; auth for the BD UI.

**Decision:** Use Supabase — single vendor for Postgres, storage, auth, and realtime channels. Realtime enables live UI updates without polling.

**Consequences:** Positive: fewest moving parts; built-in auth and realtime; row-level security for multi-user access. Negative: vendor lock-in; cost scales with storage and traffic; requires migration plan if outgrown.

---

### ADR-004: Phase 1 runs in Claude Desktop, not a custom UI

**Status:** Accepted — 26 April 2026

**Context:** Phase 1 needs a guided conversation with file access, MCP connectors, and slash-command skills. Building this from scratch is months of work. Antony is already familiar with Claude.

**Decision:** Use Claude Desktop with a custom Claude Code plugin. The plugin is a `.claude-plugin` directory containing four component types: skills (auto-selected by Claude based on their description), slash commands (explicit user triggers), agents (sub-agents for defined sub-tasks), and an MCP config (`.mcp.json`). All component definitions are markdown; plugin metadata is JSON. The plugin is hosted as a GitHub repository; users install by URL in Claude Desktop.

**Consequences:** Positive: zero UI build effort; familiar Claude interface for Antony; GitHub hosting keeps the plugin alongside the codebase and URL-based distribution means zero infrastructure overhead; MCP server ships with the plugin — no separate hosting needed for Phase 1. Negative: requires Claude-eligible Anthropic plan for every BD user; UX customisation limited to the plugin component types; depends on Claude plugin system remaining available.

---

### ADR-005: CoStar is optional in Phase 1, required in Phase 3 (with cache fallback)

**Status:** Superseded by ADR-011 — 28 April 2026

**Context:** CoStar provides ADR/occupancy benchmarks. Calling it during the concept conversation in Phase 1 may distract Antony from creative direction. The Financial Agent in Phase 3 needs it for model inputs.

**Decision:** Phase 1: CoStar is opt-in via `/market-research`. Phase 3: Financial Agent always attempts CoStar; on failure, uses cached comp set with stale flag surfaced for revenue team review.

**Consequences:** Positive: cleaner Phase 1 UX; resilient to CoStar outages. Negative: cache freshness is a concern — need TTL and invalidation policy.

---

### ADR-006: BD reviews images manually, no automated vision scoring

**Status:** Superseded by ADR-009 — 26 April 2026

**Context:** Pinterest API returns image URLs with limited metadata. Automating image selection without human review risks brand-inconsistent imagery in client-facing decks. Building a reliable automated vision judge introduces complexity and cost.

**Decision:** Image Sourcing queries Pinterest and caches candidates in Supabase. BD reviews and confirms images via the UI before any image is handed to the Assembly Agent. No auto-placement.

**Consequences:** Positive: brand quality controlled by humans; avoids vision model cost and failure modes. Negative: adds a manual step to Phase 3; BD must action image review before assembly can proceed.

---

### ADR-007: Templates are deterministic per slide type, not user-selected

**Status:** Accepted — 22 April 2026

**Context:** Allowing users to select templates per slide creates a UI surface the system cannot fully back up — if the user picks a 6-image template, the system must source 6 thematically coherent images. This is months of additional work.

**Decision:** Templates are tagged with metadata (image slots, density, chapter context, brand variant). The Assembly Agent picks the matching template deterministically. Users cannot override in the UI; manual changes happen post-export in PowerPoint.

**Consequences:** Positive: removes the hardest UI/AI integration problem; predictable output. Negative: less user flexibility; some users will want template control and won't have it in v1.

---

### ADR-008: Schema versioning from day one

**Status:** Accepted — 26 April 2026

**Context:** The Direction Document schema, Chapter Library, and template metadata will all evolve. In-flight runs must not break when these change.

**Decision:** Every schema is versioned (`direction_doc_schema_v1`, `chapter_lib_v1`, etc.). Versions are stamped on every persisted artefact. Migrations are explicit, not implicit.

**Consequences:** Positive: safe to evolve schemas; auditable. Negative: small upfront discipline overhead.

---

### ADR-009: Images are placed automatically — no BD approval gate

**Status:** Accepted — 26 April 2026 (supersedes ADR-006)

**Context:** The original design (ADR-006) required BD to review and approve Pinterest image candidates via the UI before any image was handed to the Assembly Agent. This adds a mandatory human step that blocks Phase 3 completion and increases turnaround time.

**Decision:** Remove the BD image approval gate. The Image Sourcing Agent downloads images from Pinterest directly to the Modal ephemeral filesystem and passes file paths to the Assembly Agent for automatic placement. All downloaded images are uploaded to SharePoint `delivery/image_bank/` alongside the deck so BD can see what was placed — but this is post-generation reference, not a gate. If a swap is needed, it happens in PowerPoint post-export, consistent with how all other creative refinement (Lily/Marina) works.

**Consequences:** Positive: Phase 3 runs fully automated end-to-end without a blocking human step; simpler pipeline with no approval state to track; no asset register needed. Negative: brand-inconsistent images may occasionally end up in the first-draft deck; BD catches these during PowerPoint refinement rather than upstream.

---

### ADR-011: No CoStar access — Lighthouse is the market data integration; ADR and occupancy unavailable

**Status:** Accepted — 28 April 2026 (supersedes ADR-005)

**Context:** ADR-005 assumed CoStar API access for ADR/occupancy benchmarks in both Phase 1 (`/market-research`) and the Phase 3 Financial Agent. CoStar access is not available and no resolution path exists within the project timeline.

**Decision:** Lighthouse is the sole market data integration for both Phase 1 and Phase 3. Lighthouse does not provide ADR or occupancy data. These fields will be explicitly absent in the Direction Document's Financial Anchors section and in the Financial Agent's Excel output unless the BD team supplies them manually. The system must surface these gaps clearly — never silently omit or fabricate them.

**Consequences:** Positive: unblocks the Financial Agent build immediately; no external credential dependency. Negative: the financial model will have structural gaps for ADR/occupancy on every deal until Kerten secures a data source that covers these metrics. The financial review gate becomes more important as a result, since these fields will need to be filled manually before the model is valid.

**Supersedes:** ADR-005

---

### ADR-010: Stateless design — files in SharePoint carry phase state

**Status:** Accepted — 26 April 2026 (supersedes ADR-003)

**Context:** ADR-003 picked Supabase as a single vendor for state, storage, auth, and realtime. Subsequent design work (Cathal's "ownable long-term" requirement, see `system-architecture.md`) showed that the system has only three durable state checkpoints — and each one corresponds to a human approval over a file. A general-purpose database is more infrastructure than the workload needs.

**Decision:** No application database. State between phases is carried by files in SharePoint:
- Phase 1 → Phase 2: `direction.md` (Antony approves)
- Phase 2 → Phase 3: `outline-approved.json` (BD approves)
- Phase 3 → Delivery: the `delivery/` folder

Within Phase 2 the React UI holds in-progress approval state in memory; on submit, FastAPI writes `outline-approved.json` to SharePoint. Within Phase 3 Modal handles fan-out and fan-in natively via `modal.gather()`. Brand templates live in GitHub and are baked into the Modal image at deploy time. Auth is handled by the SharePoint / Microsoft 365 layer the BD team already uses. No Postgres, no Supabase, no Azure.

**Consequences:** Positive: three vendors instead of four (Modal + Vercel + GitHub + SharePoint, all replaceable); no database to provision, migrate, or back up; SharePoint is infrastructure Kerten already owns; the system can be redeployed or handed over without data migration. Negative: no audit trail beyond SharePoint version history; resuming a crashed Phase 3 run means re-running from `outline-approved.json` rather than from the last in-flight checkpoint; assumes outline review happens in one sitting — validated assumption (see `DECISIONS.md`).

---

## 11. Alternatives Considered

**Single-agent generation (no orchestrator)**
Rejected: a single agent cannot pause mid-run for human approval gates (outline review, financial validation) without losing context. The multi-phase, file-handoff design (see ADR-010) is required to support these pauses.

**Off-the-shelf proposal generators (Tome, Gamma, Beautiful.ai, Pitch)**
Rejected: none integrate with Kerten's brand templates, financial models, or Chapter Library. None support the multi-stage approval workflow or offer an MCP boundary to Kerten's data sources.

**LangGraph for orchestration**
Rejected for v1: LangGraph offers richer graph primitives but requires more setup and is more model-agnostic than needed. A direct Anthropic SDK + FastAPI approach is a tighter fit. Revisit if orchestration complexity outgrows this.

**Automated vision scoring for image selection (Vision Judge)**
Not adopted: images are placed automatically from Pinterest results without a scoring layer. BD reviews placed images post-generation in the SharePoint delivery folder and adjusts in PowerPoint. A vision scoring layer can be introduced in v2 to improve first-draft quality if the auto-placement results are consistently off-brand.

**Free-form slide prompting**
Rejected: this is the central scope decision. See ADR-007 and §2.2. Structured chapter types with deterministic template selection is the constraint that makes the assembly tractable.

**Separate code paths per proposal archetype (complex / standard / automatic)**
Rejected: all three archetypes can be served by the same pipeline with the Chapter Library as the only variable. Separate code paths would multiply maintenance burden with no architectural benefit.

---

## 12. Cost Analysis

**Fixed monthly:** $184 &nbsp;|&nbsp; **Variable cost / proposal:** $11.66 &nbsp;|&nbsp; **Break-even volume:** ~16 proposals

### Fixed Costs

| Line Item | Unit Cost | Type |
|---|---|---|
| Claude Max 5x | — | Fixed |
| Apollo | — | Fixed |
| Apify — Pinterest scraper | — | Fixed |
| Modal | — | Fixed |
| Vercel | — | Fixed |
| **Fixed subtotal** | **$184 / mo** | |

### Variable Costs (per proposal)

| Line Item | Unit Cost | Type |
|---|---|---|
| Token consumption | $9.67 | Variable |
| Apify — LinkedIn enrichment | $0.33 | Variable |
| Apify — image scrapers | $1.66 | Variable |
| **Variable subtotal** | **$11.66 / proposal** | |

### Volume Scenarios

| Volume | Fixed | Variable | Total / month | All-in cost / proposal |
|---|---|---|---|---|
| 15 proposals | $184 | $174.90 | $358.90 | $23.93 |
| 20 proposals | $184 | $233.20 | $417.20 | $20.86 |
| 30 proposals | $184 | $349.80 | $533.80 | $17.79 |
| 50 proposals | $184 | $583.00 | $767.00 | $15.34 |
| 100 proposals | $184 | $1,166.00 | $1,350.00 | $13.50 |

---

## 13. Appendix — Client-Facing Materials (Friday 1 May)

The client meeting requires a different artefact set from this internal TDD. We will present:

1. Solution architecture diagram (3-phase, no implementation detail) — derived from §3.
2. Concept briefing UI walkthrough — what Antony sees in Claude Desktop, plus the Direction Document output.
3. List of data and access we need from Kerten to start build: CoStar credentials, template inventory, brand assets, named BD users for Claude Desktop access.
4. Timeline (week-by-week, derived from §10).
5. Scope boundary (what v1 will not do, derived from §2.2).

This TDD is internal. The client gets the simplified solution view; Theo and the build team work from this document.

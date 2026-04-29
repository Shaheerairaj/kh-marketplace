#  Concept Brief Claude CoWork Plugin

This project is about building the files for a Claude CoWork plugin. The plugin will contain detailed steps of how to write a concept brief for a hospitality brand called Kerten Hospitality. This SMB manages brands like Cloud 7, The House Hotel, HOSME and others which are brands for running properties like hotels.

# Skills
Skills are intended to be used as individual capabilities that might enable Claude to do larger pieces of work. In the context of a human, knowing how to create pivot tables in excel is a skills. In the context of this repo and creating skills, a skill can be steps on searching for hotel reviews online and summarizing them in a fixed template format.

## Why us skills
Skills are reusable, filesystem-based resources that provide Claude with domain-specific expertise: workflows, context, and best practices that transform general-purpose agents into specialists. Unlike prompts (conversation-level instructions for one-off tasks), Skills load on-demand and eliminate the need to repeatedly provide the same guidance across multiple conversations.

Key benefits:
- Specialize Claude: Tailor capabilities for domain-specific tasks
- Reduce repetition: Create once, use automatically
- Compose capabilities: Combine Skills to build complex workflows

Building a skill is like putting together an onboarding guide for a new hire. Instead of building fragmented, custom-designed agents for each use case, anyone can now specialize their agents with composable capabilities by capturing and sharing their procedural knowledge.

## Anatomy of a skill
```
skill-name/
- SKILL.md (required)
  - YAML frontmatter (name, description required)
  - Markdown instructions
- Bundled resources
  - scripts/    - executable code for deterministic/repetitive tasks
  - references/ - Docs loaded into context as needed
  - assets/     - Files used in output (templates, icons, fonts)


**Progressive Disclosure**
Skills use a three-level loading system:
1. Metadata (name + description) - Always in context (~100 words)
2. SKILL.md body - In context whenever skill triggers (<500 lines ideal)
3. Bundled resources - As needed (unlimited, scripts can execute without loading)

This metadata is the first level of progressive disclosure: it provides just enough information for Claude to know when each skill should be used without loading all of it into context. The actual body of this file is the second level of detail. If Claude thinks the skill is relevant to the current task, it will load the skill by reading its full SKILL.md into context.
```

## Skills and code execution
Skills can also include code for Claude to execute as tools at its discretion.

Large language models excel at many tasks, but certain operations are better suited for traditional code execution. For example, sorting a list via token generation is far more expensive than simply running a sorting algorithm. Beyond efficiency concerns, many applications require the deterministic reliability that only code can provide.

Eg. a PDF skill might include a pre-written Python script that reads a PDF and extracts all form fields. Claude can run this script without loading either the script or the PDF into context. And because code is deterministic, this workflow is consistent and repeatable.

## Use workflows for complex tasks
Break complex operations into clear, sequential steps. For particularly complex workflows, provide a checklist that Claude can copy into its response and check off as it progresses.

Example 1: Research synthesis workflow (for Skills without code):
```
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
```

Example 2: PDF form filling workflow (for Skills with code):
```
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
```

```
```
```
```
## Rules for refinement
- Keep skills focused. Claude composes multiple skills when a task spans several areas, and focused skills with specific descriptions tend to activate more reliably than broad ones. If a skill isn't loading when you expect, its description is likely too vague — structure it as what it does, when to use it, what it covers.
- Consider sub-agents for multi-source or long-running tasks. If a workflow pulls from several tools at once, or if a task regularly hits context limits from processing too much in one pass, sub-agents let Claude split the work across separate context windows.

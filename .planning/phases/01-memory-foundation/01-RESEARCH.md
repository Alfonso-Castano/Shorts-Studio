# Phase 1: Memory Foundation - Research

**Researched:** 2026-03-31
**Domain:** File system structure design, markdown schema contracts, YAML frontmatter
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Directory Structure**
- Flat structure inside `niches/ai-commentary/` — no subfolders besides `scripts/`
- `scripts/` — contains individual annotated training script files
- `patterns.md` — synthesized cross-script learnings (Analyzer writes this after training runs)
- `system-prompt.md` — Analyzer-generated from first batch of training scripts
- `anti-patterns.md` — user-maintained list of things Claude must never do in generated scripts
- `feedback.md` — per-generation notes, capped at 5 bullets

**Script File Schema**
- YAML frontmatter fields: `title`, `views`, `quality_tier`, `hook_type`, `date_analyzed`
- No `source` field — all scripts are from YouTube Shorts, no need to track platform/URL
- Body: `## Transcript` with [HOOK] / [BUILD] / [PAYOFF] labels, `## Analysis` with 6 fields (writing_style, hook_patterns, pacing, sentence_structure, storytelling_technique, why_it_works) each capped at 2-3 sentences

**Placeholder File Scaffolding**
- `anti-patterns.md` — schema header + empty list
- `feedback.md` — schema header + empty list
- `patterns.md` — "not yet generated — run Analyzer to populate" note
- `system-prompt.md` — "not yet generated — Analyzer creates this on first run" note

**File Naming Convention**
- Pattern: sequential number + hook-type slug
- Examples: `001-shocking-stat.md`, `002-question-hook.md`

### Claude's Discretion
- Exact wording of placeholder file headers and schema documentation
- Whether `niches.md` (top-level niche registry) uses a simple list or table format

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| MEM-01 | Niche-isolated directory structure exists (`niches/{niche}/`) with support for multiple niches | Directory tree design; `niches/ai-commentary/` with `scripts/` subdirectory; flat structure supports easy addition of future niches by creating a parallel directory |
| MEM-02 | `niches.md` lists available niches; user can add new niches by editing it | Top-level registry file design; simple format (list or table) allows low-friction user editing without tooling |
| MEM-03 | Each training script stored as structured markdown with YAML frontmatter (title, views, quality_tier, hook_type, date_analyzed) | YAML frontmatter schema design; body schema with [HOOK]/[BUILD]/[PAYOFF] labels and 6-field Analysis section |
| MEM-04 | `anti-patterns.md` exists per niche with documented schema (empty content — user fills) | Placeholder file design; schema header communicates purpose and format; empty list ensures Analyzer and Generator can read without error |
| MEM-05 | `feedback.md` exists per niche, capped at 5 bullets, user-appended via conversation | Placeholder file design; schema header documents the 5-bullet cap rule; format is enforced by convention, not code in this phase |
</phase_requirements>

---

## Summary

Phase 1 creates no code — it creates file system contracts. Every file this phase produces is a schema document: a precise specification of what Analyzer (Phase 2) will write and what Generator (Phase 3) will read. Getting these schemas right is the entire value of this phase.

The domain is markdown file design and YAML frontmatter schema definition. The standard stack is the file system itself. There are no libraries to install, no frameworks to choose. The "architecture" decision is the directory tree and file content format.

The two design risks in this phase are (1) under-documenting the schema headers in placeholder files — making it ambiguous for the user or Analyzer what goes where — and (2) creating a `niches.md` registry format that is awkward to edit by hand. Both are solvable by investing effort in clear, opinionated placeholder content.

**Primary recommendation:** Write placeholder files as if they are onboarding documents for a new team member. They should be self-explanatory, include examples where possible, and make the "what do I do next?" question obvious.

---

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| Markdown files | — | Memory storage and schema contracts | Human-readable, editable by user without tooling, consumed directly by Claude |
| YAML frontmatter | — | Structured metadata on training scripts | Standard format understood by Claude; easy to parse without a library; human-editable |
| Directory structure | — | Niche isolation and file organization | File system is zero-infrastructure; no database, no runtime, no services |

### No Libraries Required

This phase produces only static files. No npm install, no package.json, no runtime dependencies. Everything runs directly in the file system.

---

## Architecture Patterns

### Recommended Directory Structure

```
niches.md                          ← top-level niche registry (MEM-02)
niches/
└── ai-commentary/                 ← niche root (MEM-01)
    ├── anti-patterns.md           ← user-maintained, schema documented (MEM-04)
    ├── feedback.md                ← per-generation notes, max 5 bullets (MEM-05)
    ├── patterns.md                ← placeholder: Analyzer populates
    ├── system-prompt.md           ← placeholder: Analyzer generates on first run
    └── scripts/                   ← individual training script files (MEM-03)
        └── .gitkeep               ← keeps directory tracked; no scripts yet
```

### Pattern 1: Schema-First Placeholder Files

**What:** Placeholder files contain a documented schema header before any content. The header explains purpose, format rules, and who writes to the file.

**When to use:** All placeholder files in this phase. The schema header is the file's contract with downstream phases.

**Why it matters:** If `anti-patterns.md` is empty with no explanation, the user doesn't know what format to use. If it has a header explaining "each line is a phrase or pattern Claude must never use, one per bullet," the user can populate it correctly without consulting docs.

**Example — anti-patterns.md:**
```markdown
# Anti-Patterns: ai-commentary

**Purpose:** Phrases and patterns Claude must NEVER use when generating scripts for this niche.
**Maintained by:** User (add entries after bad generations)
**Read by:** Generator (enforced on every generation call)

**Format:**
- One entry per bullet
- Be specific: "you won't believe this" not "avoid clichés"
- Include context if helpful: "sounds generic — every AI channel says this"

---

## Anti-Patterns

<!-- Add entries below. Example:
- "you won't believe this" — overused AI commentary opener
- "let me break this down" — sounds like every other channel
-->
```

**Example — feedback.md:**
```markdown
# Feedback: ai-commentary

**Purpose:** Per-generation notes that improve future scripts.
**Maintained by:** User via conversation ("remember for next time that...")
**Read by:** Generator (included in every generation call)
**Limit:** 5 bullets maximum. Oldest bullet is removed when a 6th is added.

---

## Current Feedback

<!-- Up to 5 bullets. Example:
- Use shorter sentences in the hook — punchy, not explanatory
- The payoff should feel like a reveal, not a summary
-->
```

**Example — patterns.md:**
```markdown
# Patterns: ai-commentary

**Purpose:** Cross-script synthesis — what holds true across all high-performing scripts.
**Maintained by:** Analyzer (auto-generated and updated after training runs)
**Read by:** Generator (read first, before individual script examples)

---

**STATUS: Not yet generated.**
Run the Analyzer workflow with at least 3 training scripts to populate this file.

<!-- Analyzer will write structured pattern observations here. Do not edit manually. -->
```

**Example — system-prompt.md:**
```markdown
# System Prompt: ai-commentary

**Purpose:** Niche-specific system prompt for the Generator. Encodes the creator's voice, style, and niche conventions.
**Maintained by:** Analyzer (auto-generated on first training run, can be regenerated)
**Read by:** Generator (injected as system context on every generation call)

---

**STATUS: Not yet generated.**
Run the Analyzer workflow with at least 3 training scripts to generate this file.

<!-- Analyzer will write a complete system prompt here. Do not edit manually unless you know what you're doing. -->
```

### Pattern 2: YAML Frontmatter for Training Scripts

**What:** Each training script file begins with YAML frontmatter containing structured metadata, followed by the annotated transcript and analysis.

**When to use:** All files in `niches/ai-commentary/scripts/`. The Analyzer (Phase 2) writes these files; this phase only defines the schema.

**Full script file schema:**
```markdown
---
title: "AI Just Passed the Bar Exam"
views: 2400000
quality_tier: 1
hook_type: shocking-stat
date_analyzed: 2026-03-31
---

## Transcript

[HOOK]
This AI just passed the bar exam. Who's getting fired first?

[BUILD]
GPT-4 scored in the 90th percentile — better than most human lawyers who took it last year.
But here's what nobody's talking about: it didn't just memorize case law.
It reasoned through hypotheticals. The kind law school tests you on.

[PAYOFF]
Your lawyer charges $400 an hour to do what a $20 API call just did better.

## Analysis

**writing_style:** Clipped, declarative sentences. No hedging. Tone is confrontational without being hostile — forces the viewer to pick a side.

**hook_patterns:** Shocking stat delivered as a bald statement, immediately followed by a direct-address implication question. No wind-up, no context-setting.

**pacing:** Opens at maximum tension, sustains through the BUILD with escalating specificity ("90th percentile," "hypotheticals"), lands the PAYOFF as a dollar-amount gut punch.

**sentence_structure:** 70% short declaratives (under 10 words), 30% medium compound. No questions except in the hook. No filler words.

**storytelling_technique:** Contrast — everyone knows AI is smart, but scoring above most lawyers reframes the stakes. The payoff flips from abstract ("AI is impressive") to personal ("your money").

**why_it_works:** The hook assumes the viewer's conclusion before they reach it. The build makes the assumption feel inevitable. The payoff makes it personal.
```

### Pattern 3: niches.md Registry

**What:** A top-level file listing all available niches and the path to each. The user edits this file to add new niches.

**Format recommendation (Claude's discretion):** Use a table rather than a simple list. A table gives space for niche status and a brief description, making it easier to understand the project at a glance when multiple niches exist.

**Example — niches.md:**
```markdown
# Niche Registry

**Purpose:** Lists all active niches. To add a new niche, add a row to this table and create the corresponding directory under `niches/`.

| Niche | Directory | Status | Description |
|-------|-----------|--------|-------------|
| ai-commentary | `niches/ai-commentary/` | Active | AI news commentary for YouTube Shorts |

## Adding a New Niche

1. Add a row to the table above
2. Create the directory: `niches/{niche-slug}/`
3. Create subdirectory: `niches/{niche-slug}/scripts/`
4. Copy `anti-patterns.md` and `feedback.md` from an existing niche and update the headers
5. Create placeholder `patterns.md` and `system-prompt.md` (see existing niche for format)
6. Run the Analyzer to begin ingesting training scripts
```

### Anti-Patterns to Avoid

- **Empty files with no schema header:** An empty `anti-patterns.md` leaves the user with no guidance. Every placeholder file must document its own format.
- **Over-specified YAML frontmatter:** Adding fields that downstream phases don't need (e.g., `source`, `creator`, `url`) bloats the schema. CONTEXT.md explicitly removed `source` — honor that decision.
- **`scripts/` directory without a `.gitkeep`:** An empty directory won't be tracked by git. Without a placeholder, the directory contract is invisible to version control.
- **Inconsistent frontmatter types:** `views` should be an integer (not `"2.4M"` string), `quality_tier` should be an integer 1-3, not `"Tier 1"`. Consistent types allow the Analyzer to write and the Generator to read without parsing logic.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Enforcing 5-bullet cap on feedback.md | Custom validation script | Document cap in the file header; Analyzer enforces at write time (Phase 2) | Cap enforcement is Analyzer's job; this phase only establishes the contract |
| Niche discovery / listing | Directory scanning code | `niches.md` registry | File registry is human-editable, self-documenting, and works without any runtime |
| Schema validation | JSON Schema / validator | Well-documented file headers | For a solo creator tool, clear documentation is the right validation layer |

**Key insight:** Phase 1 is infrastructure-as-documentation. The "schema enforcement" is the human-readable header in each file, not a validator. Validator code belongs in Phase 2 (Analyzer) if needed at all.

---

## Common Pitfalls

### Pitfall 1: Ambiguous Placeholder Headers

**What goes wrong:** Placeholder files are created with minimal or no explanation. When the user opens `anti-patterns.md` and sees an empty file or just `# Anti-Patterns`, they don't know what to write.

**Why it happens:** The creator of the file knows the intended format; it feels redundant to document it. But the file will be read by the user and by Claude in future phases — both need clear instructions.

**How to avoid:** Write placeholder headers as if for a new team member who has never seen the project. Include purpose, who maintains it, who reads it, format rules, and at least one commented-out example.

**Warning signs:** File header is under 5 lines with no format documentation.

---

### Pitfall 2: Wrong YAML Frontmatter Value Types

**What goes wrong:** `views` is stored as `"2.4M"` (string), `quality_tier` as `"Tier 1"` (string with label). The Analyzer (Phase 2) then has to parse these strings to compare values — adding fragile parsing logic that breaks on edge cases.

**Why it happens:** The schema is designed by hand without thinking about how it will be read programmatically.

**How to avoid:** Use consistent, native YAML types from the start. `views: 2400000` (integer), `quality_tier: 1` (integer 1-3). Document this in the schema example.

**Warning signs:** Any frontmatter field value is a string that contains a number or label.

---

### Pitfall 3: Missing `.gitkeep` in `scripts/`

**What goes wrong:** The `scripts/` directory is created locally but not tracked by git. When the project is cloned or another developer joins, the directory doesn't exist. Phase 2 (Analyzer) tries to write to `niches/ai-commentary/scripts/` and fails with a path error.

**Why it happens:** Git doesn't track empty directories.

**How to avoid:** Add a `.gitkeep` file to `niches/ai-commentary/scripts/`. This is the standard convention for tracking empty directories in git.

---

### Pitfall 4: Schema Drift Between CONTEXT.md and Actual Files

**What goes wrong:** The agreed schema (5 YAML fields, 6 analysis fields) drifts during implementation. An extra field gets added, a field gets renamed. Phase 2 then reads the schema and finds the field names don't match.

**Why it happens:** Casual drift during file creation — "I'll just add `source` for now."

**How to avoid:** Treat CONTEXT.md as the locked spec. Do not add fields. The CONTEXT.md explicitly removed `source` — that decision stands.

---

## Code Examples

### Complete Training Script Template (for reference by Planner)

```markdown
---
title: ""
views: 0
quality_tier: 1
hook_type: ""
date_analyzed: ""
---

## Transcript

[HOOK]


[BUILD]


[PAYOFF]


## Analysis

**writing_style:**

**hook_patterns:**

**pacing:**

**sentence_structure:**

**storytelling_technique:**

**why_it_works:**
```

### Valid quality_tier Values

| Value | Meaning |
|-------|---------|
| `1` | Top-performing (>1M views) |
| `2` | Mid-tier (100K–1M views) |
| `3` | Standard (<100K views) |

### Valid hook_type Examples

These are illustrative — Analyzer assigns the actual value based on script content.

- `shocking-stat`
- `question-hook`
- `viral-claim`
- `contrast-reveal`
- `direct-challenge`

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| Vector database for memory | Files on disk | Zero infrastructure, no runtime, human-readable |
| Metadata in separate index file | YAML frontmatter in each file | Self-contained; metadata travels with content |
| Unstructured notes | Schema-documented placeholders | Downstream phases can read reliably without guesswork |

**Decision already made (from STATE.md):** Files on disk in `niches/{niche}/` — niche-isolated, no database. This was a deliberate choice to keep the tool simple and zero-infrastructure for a solo creator workflow.

---

## Open Questions

1. **`.gitkeep` vs `README.md` in `scripts/`**
   - What we know: `scripts/` needs a placeholder to be tracked by git
   - What's unclear: `.gitkeep` is convention; a `README.md` would also work and could document the file naming convention
   - Recommendation: Use `.gitkeep` for `scripts/` — cleaner, conventional. Document file naming convention in `niches.md` instead.

2. **`niches.md` location**
   - What we know: It's a top-level registry. CONTEXT.md calls it `niches.md`.
   - What's unclear: Should it live at project root or inside a `.config/` directory?
   - Recommendation: Project root. The user should be able to find it without knowing the project structure. It's the entry point to the memory system.

---

## Validation Architecture

> `nyquist_validation` is `true` in `.planning/config.json` — this section is required.

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None — Phase 1 creates only static files, no code |
| Config file | N/A |
| Quick run command | Manual inspection |
| Full suite command | Manual inspection |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| MEM-01 | `niches/ai-commentary/` and `niches/ai-commentary/scripts/` exist | manual smoke | `ls niches/ai-commentary/` | ❌ Wave 0 — directory created |
| MEM-02 | `niches.md` exists at project root and lists ai-commentary | manual smoke | `cat niches.md` | ❌ Wave 0 — file created |
| MEM-03 | YAML frontmatter schema is correct (5 fields, correct types) | manual inspection | Review script template | ❌ Wave 0 — schema file created |
| MEM-04 | `anti-patterns.md` exists with schema header, empty list | manual smoke | `cat niches/ai-commentary/anti-patterns.md` | ❌ Wave 0 — file created |
| MEM-05 | `feedback.md` exists with schema header and 5-bullet cap documented | manual smoke | `cat niches/ai-commentary/feedback.md` | ❌ Wave 0 — file created |

> All tests are manual inspection — Phase 1 has no executable code. Verification is visual confirmation that files exist with correct content.

### Sampling Rate

- **Per task commit:** Visual inspection of created file
- **Per wave merge:** `ls -la niches/ai-commentary/ && cat niches.md` confirms full structure
- **Phase gate:** All 5 files exist, all headers documented, YAML types are correct integers

### Wave 0 Gaps

- [ ] `niches/ai-commentary/` directory with `scripts/.gitkeep` — covers MEM-01
- [ ] `niches.md` at project root — covers MEM-02
- [ ] Script template documented in RESEARCH or PLAN — covers MEM-03
- [ ] `niches/ai-commentary/anti-patterns.md` with schema header — covers MEM-04
- [ ] `niches/ai-commentary/feedback.md` with schema header — covers MEM-05
- [ ] `niches/ai-commentary/patterns.md` with "not yet generated" placeholder
- [ ] `niches/ai-commentary/system-prompt.md` with "not yet generated" placeholder

---

## Sources

### Primary (HIGH confidence)

- CONTEXT.md (`01-CONTEXT.md`) — all schema decisions locked by user discussion
- REQUIREMENTS.md — authoritative requirement definitions for MEM-01 through MEM-05
- PITFALLS.md (`.planning/research/PITFALLS.md`) — project-specific pitfall research

### Secondary (MEDIUM confidence)

- PROJECT.md — architectural decisions (files-on-disk, niche isolation, solo creator simplicity)
- STATE.md — confirmed current position and accumulated decisions

### Tertiary (LOW confidence)

- N/A — no external sources needed for this phase; all domain knowledge is internal to the project's own context documents

---

## Metadata

**Confidence breakdown:**
- Directory structure: HIGH — fully locked in CONTEXT.md by user decision
- File schemas: HIGH — locked in CONTEXT.md (YAML fields, body structure, analysis fields)
- Placeholder content wording: MEDIUM — Claude's discretion per CONTEXT.md; recommendations above are reasoned but not locked
- `niches.md` format (table vs list): MEDIUM — Claude's discretion; table recommended for clarity

**Research date:** 2026-03-31
**Valid until:** Phase 2 start — schemas are the contract Phase 2 depends on; no external dependency decay risk

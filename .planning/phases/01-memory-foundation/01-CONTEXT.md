# Phase 1: Memory Foundation - Context

**Gathered:** 2026-03-31
**Status:** Ready for planning

<domain>
## Phase Boundary

Create the niche file system structure on disk that all downstream phases (Analyzer, Generator) depend on. This means directories, file schemas, and placeholder files for `niches/ai-commentary/`. No logic or workflows are built in this phase — just the structure and contracts.

</domain>

<decisions>
## Implementation Decisions

### Directory Structure
- Flat structure inside `niches/ai-commentary/` — no subfolders besides `scripts/`
- `scripts/` — contains individual annotated training script files
- `patterns.md` — synthesized cross-script learnings (Analyzer writes this after training runs)
- `system-prompt.md` — Analyzer-generated from first batch of training scripts
- `anti-patterns.md` — user-maintained list of things Claude must never do in generated scripts
- `feedback.md` — per-generation notes, capped at 5 bullets

### Script File Schema
- **YAML frontmatter fields**: `title`, `views`, `quality_tier`, `hook_type`, `date_analyzed`
- No `source` field — all scripts are from YouTube Shorts, no need to track platform/URL
- **Body structure**:
  - `## Transcript` — labeled [HOOK] / [BUILD] / [PAYOFF] sections
  - `## Analysis` — 6 structured fields, each capped at 2-3 sentences for token efficiency:
    - `writing_style` — tone, register, energy level
    - `hook_patterns` — specific technique used in the hook
    - `pacing` — how energy and timing flow through the script
    - `sentence_structure` — length, rhythm, question vs statement ratio
    - `storytelling_technique` — setup/tension/reveal, journey, contrast, etc.
    - `why_it_works` — overall summary of what makes this script perform

### Placeholder File Scaffolding
- `anti-patterns.md` — schema header explaining purpose + format, empty list (user and Claude populate over time via feedback)
- `feedback.md` — schema header with format docs (one bullet per note, max 5), empty list
- `patterns.md` — schema placeholder with a "not yet generated — run Analyzer to populate" note
- `system-prompt.md` — schema placeholder with a "not yet generated — Analyzer creates this on first run" note

### File Naming Convention
- Pattern: sequential number + hook-type slug
- Examples: `001-shocking-stat.md`, `002-question-hook.md`, `003-viral-claim.md`
- Number ensures predictable sort order; slug describes the defining hook technique

### Claude's Discretion
- Exact wording of placeholder file headers and schema documentation
- Whether `niches.md` (top-level niche registry) uses a simple list or table format

</decisions>

<specifics>
## Specific Ideas

- The training data purpose: Claude ingests scripts, compares them cross-script, extracts what makes each one viral, and stores both the raw annotated transcript AND the learned insights in one file
- Token efficiency is a first-class concern: script files are concise (analysis fields capped at 2-3 sentences), and during generation only 3-5 top training scripts are read — not all ingested scripts
- `patterns.md` is the cross-script synthesis layer — it captures what holds across many scripts, so Claude reads it first before individual script examples during generation

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — this is a greenfield project. No existing components, hooks, or utilities.

### Established Patterns
- None yet — this phase establishes the patterns that subsequent phases follow.

### Integration Points
- `niches/ai-commentary/scripts/` — Analyzer (Phase 2) writes to this directory
- `niches/ai-commentary/patterns.md` — Analyzer updates after each training run; Generator reads during generation
- `niches/ai-commentary/system-prompt.md` — Analyzer generates on first run; Generator reads
- `niches/ai-commentary/anti-patterns.md` — Generator enforces these; user maintains
- `niches/ai-commentary/feedback.md` — Generator reads and appends to

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-memory-foundation*
*Context gathered: 2026-03-31*

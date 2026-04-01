# Phase 2: Analyzer Workflow - Context

**Gathered:** 2026-04-01
**Status:** Ready for planning

<domain>
## Phase Boundary

Build the `/script-generator` Claude Code skill — specifically the Analyzer path. User invokes the skill, selects niche and mode, pastes a transcript with view count and notes, and Claude analyzes it, writes a structured training script to disk, updates patterns.md, and generates system-prompt.md after the 3rd script. No Generator workflow in this phase.

</domain>

<decisions>
## Implementation Decisions

### Skill invocation & routing
- Single `/script-generator` skill file handles the full entry point
- Opening prompt collects niche and mode in one message: `Which niche? (ai-commentary) | Analyzer or Generator?`
- User responds in one message; Claude routes accordingly

### Analyzer input collection
- Guided fill format: Analyzer shows a short example with light structure (not a strict template)
- Example shown: `Views: [number] | Notes: [optional] | Transcript: [paste here]`
- User can paste free-form as long as the three pieces are present; Claude parses from context
- All three inputs collected in one user message (ANLZ-02)

### Analysis behavior
- Claude performs best-effort segmentation into [HOOK] / [BUILD] / [PAYOFF] — no strict sentence-count rules
- Segmentation decisions are visible in the draft for user review
- Analyze in isolation: Analyzer does NOT read existing training scripts before analyzing a new one
- Analyzer reads `anti-patterns.md` and `patterns.md` before analysis (for context), but not individual scripts
- 6 analysis fields used as defined in SCHEMA.md — no additions: `writing_style`, `hook_patterns`, `pacing`, `sentence_structure`, `storytelling_technique`, `why_it_works`
- Each field capped at 2–3 sentences (token efficiency, established in Phase 1)

### Save confirmation UX
- Analyzer shows the complete YAML + segmented transcript + analysis as a draft
- Claude waits for user approval before writing to disk
- User can request corrections to the draft before saving
- After save: one-line summary + next action hint, e.g. `Saved 001-shocking-stat.md (Tier 1, 2.4M views). Ready for another transcript — or run Generator when you have 3+ scripts.`

### Anti-pattern flagging (ANLZ-07)
- Matching phrases are flagged inline in the draft (shown before user approves)
- Flagging does NOT block saving — user sees the warning and decides when approving
- When `anti-patterns.md` is empty: mention it once on the first run only (`anti-patterns.md is empty — add phrases to flag in future runs`), then skip silently on all subsequent runs
- Note: anti-patterns.md is intentionally empty at first — it will be populated retroactively as the user generates scripts and identifies clichés

### patterns.md updates
- After every successful ingestion (script saved), Analyzer re-reads all scripts in `scripts/` and rewrites `patterns.md` with updated cross-script synthesis
- Always kept current — no batching or manual trigger needed

### system-prompt.md generation (ANLZ-06)
- Generates only when `scripts/` contains 3+ scripts AND `system-prompt.md` is still a placeholder
- If <3 scripts after save: mention count in the save summary, e.g. `system-prompt.md will generate after 3 scripts are ingested (currently: 2)`
- When triggered (3rd script saved): Claude shows the generated system prompt as a draft inline, waits for user approval before writing to disk
- Consistent with save confirmation UX — user approves both the script draft and the system prompt draft before anything is written

### Claude's Discretion
- Exact wording of the guided fill prompt shown to users
- How to handle edge cases in input parsing (e.g. notes field omitted)
- File naming: next sequential number + hook-type slug derived from analysis
- How to structure patterns.md synthesis (format and sections)
- How to structure the system-prompt.md content (Claude derives from script analysis)

</decisions>

<specifics>
## Specific Ideas

- The guided fill format should feel lightweight, not like filling out a form — show an example, not a rigid template
- The save summary hint should track progress toward system-prompt.md generation so the user always knows where they stand
- The skill should be as frictionless as possible: single prompt to start, one message to provide input, one approval to save — no unnecessary back-and-forth

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `niches/ai-commentary/SCHEMA.md` — defines the exact file format Analyzer must write; treat as locked spec
- `niches/ai-commentary/anti-patterns.md` — schema + empty list; Analyzer reads this before each analysis
- `niches/ai-commentary/patterns.md` — placeholder with STATUS header; Analyzer overwrites this after each ingestion
- `niches/ai-commentary/system-prompt.md` — placeholder with STATUS header; Analyzer writes here on 3rd script
- `niches/ai-commentary/scripts/` — target directory for all Analyzer output; currently empty

### Established Patterns
- File schema is locked from Phase 1 — do not change field names, types, or structure
- `quality_tier` is integer 1/2/3 (not string); `views` is integer with no abbreviations
- Analysis fields capped at 2–3 sentences — token efficiency is a first-class constraint
- All scripts are YouTube Shorts — no `source` field

### Integration Points
- `/script-generator` skill file lives in `.claude/skills/script-generator/SKILL.md` (or equivalent Claude Code skill path)
- Phase 3 (Generator) reads the files this phase writes — schema compliance is critical
- Analyzer is the write path; Generator is the read path

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 02-analyzer-workflow*
*Context gathered: 2026-04-01*

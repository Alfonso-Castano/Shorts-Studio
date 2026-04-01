# Phase 2: Analyzer Workflow - Research

**Researched:** 2026-04-01
**Domain:** Claude Code skill implementation — text-based workflow, file I/O, YAML authoring, prompt design
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- Single `/script-generator` skill file handles the full entry point
- Opening prompt collects niche and mode in one message: `Which niche? (ai-commentary) | Analyzer or Generator?`
- User responds in one message; Claude routes accordingly
- Guided fill format: Analyzer shows a short example with light structure (not a strict template)
- Example shown: `Views: [number] | Notes: [optional] | Transcript: [paste here]`
- User can paste free-form as long as the three pieces are present; Claude parses from context
- All three inputs collected in one user message (ANLZ-02)
- Claude performs best-effort segmentation into [HOOK] / [BUILD] / [PAYOFF] — no strict sentence-count rules
- Segmentation decisions are visible in the draft for user review
- Analyze in isolation: Analyzer does NOT read existing training scripts before analyzing a new one
- Analyzer reads `anti-patterns.md` and `patterns.md` before analysis (for context), but not individual scripts
- 6 analysis fields used as defined in SCHEMA.md — no additions: `writing_style`, `hook_patterns`, `pacing`, `sentence_structure`, `storytelling_technique`, `why_it_works`
- Each field capped at 2–3 sentences (token efficiency, established in Phase 1)
- Analyzer shows the complete YAML + segmented transcript + analysis as a draft
- Claude waits for user approval before writing to disk
- User can request corrections to the draft before saving
- After save: one-line summary + next action hint
- Anti-pattern matching phrases are flagged inline in the draft (shown before user approves)
- Flagging does NOT block saving — user sees the warning and decides when approving
- When `anti-patterns.md` is empty: mention it once on the first run only, then skip silently
- After every successful ingestion (script saved), Analyzer re-reads all scripts in `scripts/` and rewrites `patterns.md`
- system-prompt.md generates only when `scripts/` contains 3+ scripts AND `system-prompt.md` is still a placeholder
- When triggered (3rd script saved): Claude shows generated system prompt as a draft inline, waits for user approval
- File schema (SCHEMA.md) is locked — do not change field names, types, or structure

### Claude's Discretion

- Exact wording of the guided fill prompt shown to users
- How to handle edge cases in input parsing (e.g. notes field omitted)
- File naming: next sequential number + hook-type slug derived from analysis
- How to structure patterns.md synthesis (format and sections)
- How to structure the system-prompt.md content (Claude derives from script analysis)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| ANLZ-01 | User invokes `/script-generator`, selects niche, selects Analyzer | Skill file placement at `.claude/commands/script-generator.md`; opening prompt pattern from prompt-engineer skill |
| ANLZ-02 | Skill prompts user to paste transcript + view count + notes in one message | Guided fill example pattern; Claude parses free-form input — no strict parser needed |
| ANLZ-03 | Claude analyzes transcript and segments into [HOOK] / [BUILD] / [PAYOFF] with written analysis | SCHEMA.md defines exact 6-field analysis format; segmentation is best-effort, shown in draft |
| ANLZ-04 | Quality tier assigned based on view count (Tier 1 >1M, Tier 2 100K–1M, Tier 3 <100K) | Logic is a simple threshold; SCHEMA.md confirms `quality_tier` is integer 1/2/3 |
| ANLZ-05 | Structured script file written to `niches/{niche}/scripts/` with correct schema | SCHEMA.md is locked spec; naming convention is `{NNN}-{hook-type-slug}.md`; sequential NNN from directory listing |
| ANLZ-06 | If `system-prompt.md` does not yet exist (placeholder), analyzer generates it | STATUS header in system-prompt.md is detection signal; triggered on 3rd script; draft-then-approve UX |
| ANLZ-07 | Analyzer flags any phrases in the transcript that match `anti-patterns.md` | anti-patterns.md is currently empty; skill reads it, does case-insensitive substring match, flags inline before approval |
</phase_requirements>

---

## Summary

This phase implements a single Claude Code skill file (a `.md` command file) that drives an interactive Analyzer workflow entirely through Claude's conversational capability. There is no code to compile, no dependencies to install, and no database — the "application logic" lives inside a well-structured prompt that tells Claude what to read, how to reason, and what to write to disk.

The core challenge is prompt engineering: writing instructions that are precise enough to produce consistent YAML output, handle user input variation gracefully, and sequence multi-step operations (collect input → analyze → show draft → await approval → write files → update patterns.md → maybe write system-prompt.md) without requiring the user to invoke multiple commands.

The secondary challenge is file-state management: Claude must inspect the `scripts/` directory to generate sequential filenames, check `system-prompt.md`'s STATUS header to know whether to generate it, and read `anti-patterns.md` for matching even when it is empty.

**Primary recommendation:** Implement the skill as a single `.claude/commands/script-generator.md` file containing a structured prompt with clear Role, Instructions, Constraints, and a step-by-step decision tree that Claude executes within one conversation session.

---

## Standard Stack

### Core

| Component | Version/Format | Purpose | Why Standard |
|-----------|---------------|---------|--------------|
| Claude Code command file | `.md` in `.claude/commands/` | Skill entry point invoked via `/script-generator` | This IS the platform — no alternatives |
| YAML frontmatter | Per SCHEMA.md spec | Training script output format | Locked from Phase 1 |
| Markdown body | Per SCHEMA.md spec | Transcript + analysis section | Locked from Phase 1 |

### Supporting

| Component | Purpose | When to Use |
|-----------|---------|-------------|
| `niches/ai-commentary/SCHEMA.md` | Output format reference | Skill reads this as spec; planner embeds key rules directly in the prompt |
| `niches/ai-commentary/anti-patterns.md` | Phrase blacklist | Read at analysis start; empty at first run |
| `niches/ai-commentary/patterns.md` | Cross-script synthesis | Read at analysis start; rewritten after every save |
| `niches/ai-commentary/system-prompt.md` | Generator system prompt | Written on 3rd script save when STATUS placeholder detected |
| `niches/ai-commentary/scripts/` | Output directory | All analyzed scripts land here; directory listing drives file numbering |

### No Dependencies to Install

This phase has no `npm install`, no new libraries, no build steps. The entire implementation is a `.md` prompt file.

---

## Architecture Patterns

### Recommended Project Structure

```
.claude/
└── commands/
    └── script-generator.md    ← The skill (one file, this phase)

niches/
└── ai-commentary/
    ├── SCHEMA.md              ← Locked spec (read by skill)
    ├── anti-patterns.md       ← Read before every analysis
    ├── patterns.md            ← Rewritten after every save
    ├── system-prompt.md       ← Written on 3rd script (if placeholder)
    ├── feedback.md            ← Not used by Analyzer (Generator only)
    └── scripts/
        └── {NNN}-{slug}.md   ← Analyzer output files
```

### Pattern 1: Claude Code Command File Structure

**What:** A Markdown file placed in `.claude/commands/` becomes a slash command in Claude Code. The file contains the full prompt Claude receives when the user invokes `/script-generator`.

**When to use:** Always — this is the only mechanism available.

**Key structural sections the prompt must contain:**

```markdown
# Script Generator

## Role
[Who Claude is in this skill]

## On Invocation
[Exactly what Claude does first — the opening question]

## Analyzer Path
[Step-by-step instructions for the Analyzer workflow]

### Step 1: Collect Input
[Show guided fill example, wait for user message]

### Step 2: Read Context Files
[List files to read before analyzing: anti-patterns.md, patterns.md]

### Step 3: Analyze
[Segmentation rules, analysis fields, quality tier logic]

### Step 4: Show Draft
[Exact draft format, flag anti-pattern matches inline]

### Step 5: Await Approval
[Wait for user "save" or correction request]

### Step 6: Write Files
[File naming, write to scripts/, update patterns.md, check system-prompt.md]

### Step 7: Post-Save Summary
[One-line summary + progress hint]

## Generator Path
[Placeholder — Phase 3]

## Constraints
[Schema rules, anti-patterns.md empty behavior, etc.]
```

### Pattern 2: Draft-Then-Approve UX

**What:** Claude generates a full draft (YAML + segmented transcript + analysis), displays it to the user, then explicitly waits for approval before writing anything to disk.

**Why:** Prevents bad data from entering the training corpus. User can catch segmentation errors, wrong tier assignments, or awkward analysis phrasing before committing.

**Implementation:** The prompt must include explicit language like: "Display the full draft below. Do NOT write any files until the user replies with an approval. If the user requests changes, revise and show the updated draft before asking again."

### Pattern 3: Sequential File Numbering

**What:** Filenames follow `{NNN}-{hook-type-slug}.md` where NNN is zero-padded to 3 digits.

**How Claude determines next number:** Before writing, Claude reads the `scripts/` directory, finds the highest existing number, and increments by 1. If `scripts/` is empty, start at `001`.

**Implementation note:** The prompt must instruct Claude to use the `list_directory` or `read_file` tool to inspect `scripts/` before naming the file — not to guess or assume.

### Pattern 4: STATUS Header Detection

**What:** `system-prompt.md` and `patterns.md` both contain a `STATUS: Not yet generated.` line at the top. This is the detection signal for "placeholder vs. real content."

**Why:** Avoids checking file length or modification time. A simple substring check for `STATUS: Not yet generated` is reliable.

**Trigger logic for system-prompt.md:**
1. After saving a script, count `.md` files in `scripts/`
2. If count >= 3 AND `system-prompt.md` contains `STATUS: Not yet generated` → generate
3. If count < 3 → include count in save summary: `system-prompt.md will generate after 3 scripts (currently: N)`

### Pattern 5: patterns.md Synthesis

**What:** After every successful save, Analyzer re-reads all scripts in `scripts/` and rewrites `patterns.md` with updated cross-script synthesis.

**Format for patterns.md (Claude's discretion — recommended):**

```markdown
# Patterns: ai-commentary

**Last updated:** {date} — {N} scripts analyzed

---

## Hook Patterns
[Recurring hook techniques with frequency notes]

## Pacing Patterns
[Energy flow patterns observed across scripts]

## Sentence Structure Norms
[Length, rhythm, question vs statement patterns]

## Storytelling Approaches
[Setups and structures that recur]

## What Consistently Works
[Cross-cutting synthesis]
```

### Anti-Patterns to Avoid

- **Hardcoding logic in multiple places:** The skill file is the single source of truth. Don't split instructions across multiple files.
- **Overly rigid input parsing:** User input is free-form. Instruct Claude to extract the three values from context, not to expect exact formatting.
- **Blocking on empty anti-patterns.md:** When the file is empty, the prompt must tell Claude to skip matching silently (after mentioning it once on the first run).
- **Writing files before approval:** The prompt must explicitly separate the "show draft" step from the "write files" step with a hard wait instruction.
- **Ambiguous approval detection:** Tell Claude exactly what constitutes approval: any affirmative ("save", "looks good", "go ahead") triggers writing. Anything else = revision request or wait.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| YAML generation | Custom YAML serializer | Claude writes YAML directly as text | Claude already knows YAML syntax; schema is simple and fixed |
| File numbering | Complex glob/sort logic | One-line directory read + max + increment instruction in prompt | Claude can list and count files natively |
| Transcript segmentation | Rule-based sentence counter | Claude's best-effort understanding | Segmentation is semantic, not mechanical — LLM outperforms rules here |
| Anti-pattern matching | Regex engine | Simple substring/phrase scan in Claude's response | List is short, phrases are literal, no complex patterns needed |
| patterns.md aggregation | Separate summarization pass | Claude reads all scripts inline and synthesizes | Context window is large enough for N training scripts at this scale |

**Key insight:** This phase is entirely prompt engineering. Every "algorithm" is an instruction to Claude. Resist the urge to write Node.js scripts or helper utilities — the simplicity of the plain-text skill file is a feature, not a limitation.

---

## Common Pitfalls

### Pitfall 1: Prompt Ambiguity on Approval Detection

**What goes wrong:** Claude writes files immediately after showing the draft because the prompt doesn't explicitly say "wait for user reply."
**Why it happens:** LLMs complete the next logical step unless instructed to pause.
**How to avoid:** Include an explicit instruction: "After displaying the draft, stop and wait. Only proceed to file writing when the user explicitly approves."
**Warning signs:** In testing, if Claude says "I've saved the file" in the same message as the draft, the pause instruction is missing.

### Pitfall 2: system-prompt.md Generation Trigger Miscounting

**What goes wrong:** system-prompt.md gets generated on the 2nd script instead of the 3rd, or never generates at all.
**Why it happens:** The count check happens before or after the wrong step (e.g., counting before the new script is written).
**How to avoid:** Count AFTER writing the new script to disk. The correct sequence: write script → count files in scripts/ → if count >= 3 AND placeholder → generate.

### Pitfall 3: patterns.md Write Erases Header

**What goes wrong:** Analyzer overwrites `patterns.md` but loses the file's purpose/header comment.
**How to avoid:** Specify in the prompt that the written patterns.md must include the purpose header, last-updated date, and script count — not just raw pattern content.

### Pitfall 4: YAML Field Type Violations

**What goes wrong:** `views: "2.4M"` or `quality_tier: "Tier 1"` appears in output despite SCHEMA.md.
**Why it happens:** Claude defaults to readable representations unless explicitly constrained.
**How to avoid:** Include explicit rules in the prompt: "views must be a plain integer with no quotes, commas, or abbreviations. quality_tier must be 1, 2, or 3 as an unquoted integer."
**Warning signs:** Any YAML value with quotes around numbers or abbreviated units.

### Pitfall 5: Anti-patterns.md Empty File Behavior

**What goes wrong:** Skill mentions "anti-patterns.md is empty" on every run, becoming noise.
**How to avoid:** Track first-run state via a simple instruction: "If anti-patterns.md contains no bullet entries (only the header/comments), mention this ONCE in the session and then proceed silently on all subsequent analyses in this conversation."

### Pitfall 6: File Path Confusion Across Niches

**What goes wrong:** Skill hardcodes `ai-commentary` paths instead of using the niche selected at invocation.
**How to avoid:** The opening question captures the niche. All subsequent file paths must use the selected niche name as a variable: `niches/{selected-niche}/...`. Even though only `ai-commentary` is active now, the pattern must be correct for Phase 3 and beyond.

---

## Code Examples

Verified patterns from existing project files:

### YAML Frontmatter Output (from SCHEMA.md)

```yaml
---
title: "AI Just Passed the Bar Exam"
views: 2400000
quality_tier: 1
hook_type: shocking-stat
date_analyzed: 2026-04-01
---
```

Key rules (SCHEMA.md confirmed):
- `views`: integer, no quotes, no commas, no abbreviations
- `quality_tier`: integer 1/2/3, not string
- `hook_type`: kebab-case slug
- `date_analyzed`: ISO date string
- No `source` field

### Segmented Transcript Block (from SCHEMA.md)

```
## Transcript

[HOOK]
This AI just passed the bar exam. Who's getting fired first?

[BUILD]
GPT-4 scored in the 90th percentile — better than most human lawyers who took it last year.
But here's what nobody's talking about: it didn't just memorize case law.
It reasoned through hypotheticals. The kind law school tests you on.

[PAYOFF]
Your lawyer charges $400 an hour to do what a $20 API call just did better.
```

### Analysis Block (from SCHEMA.md)

```markdown
## Analysis

**writing_style:** Clipped, declarative sentences. No hedging. Tone is confrontational without being hostile — forces the viewer to pick a side.

**hook_patterns:** Shocking stat delivered as a bald statement, immediately followed by a direct-address implication question. No wind-up, no context-setting.

**pacing:** Opens at maximum tension, sustains through the BUILD with escalating specificity ("90th percentile," "hypotheticals"), lands the PAYOFF as a dollar-amount gut punch.

**sentence_structure:** 70% short declaratives (under 10 words), 30% medium compound. No questions except in the hook. No filler words.

**storytelling_technique:** Contrast — everyone knows AI is smart, but scoring above most lawyers reframes the stakes. The payoff flips from abstract ("AI is impressive") to personal ("your money").

**why_it_works:** The hook assumes the viewer's conclusion before they reach it. The build makes the assumption feel inevitable. The payoff makes it personal.
```

### Quality Tier Logic

```
views >= 1,000,000  → quality_tier: 1
views >= 100,000    → quality_tier: 2
views < 100,000     → quality_tier: 3
```

### Anti-Pattern Flag Example (inline in draft)

```
⚠️ Anti-pattern detected: "you won't believe this" (line 3 of transcript)
```

Flag appears between the YAML block and the analysis section so the user sees it before approving.

### Save Summary Line Format

```
Saved 001-shocking-stat.md (Tier 1, 2.4M views). Ready for another transcript — or run Generator when you have 3+ scripts.
```

When under 3 scripts:

```
Saved 002-contrast-reveal.md (Tier 2, 340K views). system-prompt.md will generate after 3 scripts are ingested (currently: 2).
```

### Guided Fill Prompt (Claude's discretion — recommended wording)

```
Paste the transcript details in one message. Format:

Views: [number]
Notes: [optional — anything notable about this script]
Transcript: [paste here]

Notes can be omitted if you have nothing to add.
```

---

## State of the Art

| Old Approach | Current Approach | Notes |
|--------------|-----------------|-------|
| Separate command files per mode | Single skill file routes by user selection | Simpler invocation, less context fragmentation |
| Code-based YAML generation | LLM writes YAML directly | No serialization layer needed for simple fixed schemas |
| Manual patterns.md maintenance | Auto-synthesized after every Analyzer run | Always current, no manual trigger |

---

## Open Questions

1. **Skill invocation path: `.claude/commands/script-generator.md` vs `.claude/commands/gsd/script-generator.md`**
   - What we know: The GSD commands live in `.claude/commands/gsd/`. The `skills/` directory exists separately but contains reference SKILL.md files, not Claude Code command files.
   - What's unclear: Whether the new skill should live at `.claude/commands/script-generator.md` (top-level) or in a subdirectory.
   - Recommendation: Place at `.claude/commands/script-generator.md` to invoke as `/script-generator` directly. Subdirectory would require `/gsd/script-generator` which contradicts the decision.

2. **patterns.md update on first script save (only 1 script exists)**
   - What we know: patterns.md purpose is cross-script synthesis. With 1 script it has nothing to synthesize.
   - What's unclear: Should Analyzer still rewrite patterns.md after the 1st save, or only starting from the 2nd?
   - Recommendation: Write patterns.md after every save regardless of count. With 1 script, note it: "Patterns from 1 script — limited synthesis available. Add more scripts for reliable patterns."

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected — this phase has no automated test infrastructure |
| Config file | None — see Wave 0 |
| Quick run command | Manual invocation of `/script-generator` in Claude Code |
| Full suite command | Full Analyzer workflow walkthrough (collect input → analyze → draft → approve → verify files written) |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| ANLZ-01 | `/script-generator` invoked, niche + mode selected | manual-only | N/A — Claude Code UI interaction | ❌ Wave 0 |
| ANLZ-02 | Single-message input collection with guided fill | manual-only | N/A — interactive session | ❌ Wave 0 |
| ANLZ-03 | Transcript segmented into HOOK/BUILD/PAYOFF with 6-field analysis | manual-only | N/A — LLM output inspection | ❌ Wave 0 |
| ANLZ-04 | Quality tier correctly computed from view count | manual-only | N/A — output inspection | ❌ Wave 0 |
| ANLZ-05 | Script file appears at correct path with valid YAML | smoke | `ls niches/ai-commentary/scripts/ && head -10 niches/ai-commentary/scripts/001-*.md` | ❌ Wave 0 |
| ANLZ-06 | system-prompt.md written after 3rd script, not before | smoke | `cat niches/ai-commentary/system-prompt.md` (check STATUS line removed) | ❌ Wave 0 |
| ANLZ-07 | Anti-pattern phrases flagged inline in draft | manual-only | N/A — requires live session with non-empty anti-patterns.md | ❌ Wave 0 |

**Note on manual-only:** All ANLZ requirements are satisfied by running the actual skill in Claude Code with a real transcript. There is no application code to unit test — the "tests" are walkthrough verification steps using real file output as evidence.

### Sampling Rate

- **Per task commit:** Run `/script-generator` with one sample transcript and verify output file structure matches SCHEMA.md
- **Per wave merge:** Full Analyzer workflow: 3 transcripts ingested, patterns.md updated, system-prompt.md generated
- **Phase gate:** 3 scripts saved in `niches/ai-commentary/scripts/` with correct YAML + system-prompt.md generated before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] No test files needed — this phase produces no application code to unit test
- [ ] Verification is by file inspection: `ls` and `head` commands confirm file presence and YAML structure
- [ ] ANLZ-07 requires adding at least one anti-pattern entry to `anti-patterns.md` before testing the flagging behavior

*(Wave 0 note: No test framework install needed. Verification is manual + file inspection only.)*

---

## Sources

### Primary (HIGH confidence)

- `niches/ai-commentary/SCHEMA.md` — locked output format, field types, naming conventions
- `niches/ai-commentary/anti-patterns.md` — confirmed empty; schema and format read
- `niches/ai-commentary/patterns.md` — confirmed placeholder STATUS header
- `niches/ai-commentary/system-prompt.md` — confirmed placeholder STATUS header
- `.planning/phases/02-analyzer-workflow/02-CONTEXT.md` — all implementation decisions
- `.planning/REQUIREMENTS.md` — ANLZ-01 through ANLZ-07 definitions
- `.claude/skills/prompt-engineer/SKILL.md` — structured system prompt pattern (Role/Context/Instructions/Constraints/Output format/Examples)

### Secondary (MEDIUM confidence)

- `.planning/STATE.md` — platform decision (Claude Code skills), memory architecture (files on disk)

### Tertiary (LOW confidence)

- None

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no external libraries; all components are existing project files
- Architecture: HIGH — skill file structure is well-defined by the platform; all decisions locked in CONTEXT.md
- Pitfalls: HIGH — derived from locked schema rules and known LLM behavioral patterns; verified against existing file content
- Validation: HIGH — manual-only testing is appropriate and correct for a conversational LLM skill

**Research date:** 2026-04-01
**Valid until:** 2026-05-01 (stable domain — no external dependencies that could change)

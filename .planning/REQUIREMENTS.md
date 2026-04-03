# Requirements: Shorts Studio — Script Generator

**Defined:** 2026-03-30
**Core Value:** Generate scripts that maximize viewer retention and engagement — the script is the highest-leverage part of the content.

## v1 Requirements

### Memory Structure

- [x] **MEM-01**: Niche-isolated directory structure exists (`niches/{niche}/`) with support for multiple niches
- [x] **MEM-02**: `niches.md` lists available niches; user can add new niches by editing it
- [x] **MEM-03**: Each training script stored as a structured markdown file with YAML frontmatter (title, source, views, quality_tier, hook_type)
- [x] **MEM-04**: `anti-patterns.md` exists per niche with documented schema (empty content — user fills)
- [x] **MEM-05**: `feedback.md` exists per niche, capped at 5 bullets, user-appended via conversation

### Analyzer Workflow

- [x] **ANLZ-01**: User invokes `/script-generator`, selects niche, selects Analyzer
- [x] **ANLZ-02**: Skill prompts user to paste transcript + view count + notes in one message
- [x] **ANLZ-03**: Claude analyzes transcript and segments it into [HOOK] / [BUILD] / [PAYOFF] with written analysis (hook technique, build strategy, payoff, why it works)
- [x] **ANLZ-04**: Quality tier assigned based on view count (Tier 1 >1M, Tier 2 100K–1M, Tier 3 <100K)
- [x] **ANLZ-05**: Structured script file written to `niches/{niche}/scripts/` with correct schema
- [x] **ANLZ-06**: If `system-prompt.md` does not yet exist, analyzer generates it from the ingested scripts
- [x] **ANLZ-07**: Analyzer flags any phrases in the transcript that match `anti-patterns.md`

### Generator Workflow

- [ ] **GEN-01**: User invokes `/script-generator`, selects niche, selects Generator
- [ ] **GEN-02**: Skill reads `system-prompt.md`, `anti-patterns.md`, `feedback.md` for selected niche
- [ ] **GEN-03**: Skill reads top 3–5 training scripts, soft-sorted by quality tier (Tier 1 first)
- [ ] **GEN-04**: Skill prompts user for 3–5 sentence video summary and optional direction
- [ ] **GEN-05**: Claude generates 2–3 script variations, each with [HOOK] (~2–3s) / [BUILD] (~15–20s) / [PAYOFF] (~3–5s) and a brief annotation per section
- [ ] **GEN-06**: Voice check runs after generation — flags any phrases that sound generic or interchangeable with any AI commentary channel
- [ ] **GEN-07**: User can say "remember for next time that..." and skill appends to `feedback.md` (trimmed to 5 bullets)

### Tune & Validate

- [ ] **TUNE-01**: Generation with memory enabled produces measurably different output than without memory
- [ ] **TUNE-02**: Creator can use 5 generated scripts with less than 40% manual rewrites

## v2 Requirements

### Future Enhancements

- **FUT-01**: UI for niche selection (replace CLI workflow)
- **FUT-02**: Video file input via AI vision (conditional on cost-efficiency)
- **FUT-03**: Refinement preference learning — track what kinds of edits user makes over time
- **FUT-04**: Provider swapping — swap LLM from Claude to another provider via config change

## Out of Scope

| Feature | Reason |
|---------|--------|
| Video upload / AI vision | Too expensive per clip at scale; text summary produces equal or better context |
| Edit brief / captions | Separate future component |
| Voiceover (ElevenLabs) | Separate future component |
| Web or desktop UI | Deferred — validate quality via CLI first |
| Multiple niches active simultaneously | Structure built, only ai-commentary active this milestone |
| Standalone API/CLI tool (Node.js) | Option A (Claude Code skill) chosen for v1 |
| Strict quality tier filtering | Soft preference only — avoids empty retrieval edge cases |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| MEM-01 | Phase 1 | Complete |
| MEM-02 | Phase 1 | Complete |
| MEM-03 | Phase 1 | Complete |
| MEM-04 | Phase 1 | Complete |
| MEM-05 | Phase 1 | Complete |
| ANLZ-01 | Phase 2 | Complete |
| ANLZ-02 | Phase 2 | Complete |
| ANLZ-03 | Phase 2 | Complete |
| ANLZ-04 | Phase 2 | Complete |
| ANLZ-05 | Phase 2 | Complete |
| ANLZ-06 | Phase 2 | Complete |
| ANLZ-07 | Phase 2 | Complete |
| GEN-01 | Phase 3 | Pending |
| GEN-02 | Phase 3 | Pending |
| GEN-03 | Phase 3 | Pending |
| GEN-04 | Phase 3 | Pending |
| GEN-05 | Phase 3 | Pending |
| GEN-06 | Phase 3 | Pending |
| GEN-07 | Phase 3 | Pending |
| TUNE-01 | Phase 4 | Pending |
| TUNE-02 | Phase 4 | Pending |

**Coverage:**
- v1 requirements: 21 total
- Mapped to phases: 21
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-30*
*Last updated: 2026-04-01 after Phase 2 Plan 01 — ANLZ-01, ANLZ-02, ANLZ-03, ANLZ-04, ANLZ-07 complete*

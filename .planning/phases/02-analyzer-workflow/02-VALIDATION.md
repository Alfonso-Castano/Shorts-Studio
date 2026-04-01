---
phase: 2
slug: analyzer-workflow
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-01
---

# Phase 2 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — manual inspection only (no application code to unit test) |
| **Config file** | None — Wave 0 has no framework install |
| **Quick run command** | Invoke `/script-generator` with one sample transcript; inspect output file |
| **Full suite command** | Full walkthrough: 3 transcripts → verify patterns.md updated → verify system-prompt.md generated |
| **Estimated runtime** | ~5 minutes manual |

---

## Sampling Rate

- **After every task commit:** Invoke `/script-generator` with one sample transcript and verify output file structure matches SCHEMA.md
- **After every plan wave:** Full Analyzer walkthrough — 3 transcripts ingested, patterns.md updated, system-prompt.md generated
- **Before `/gsd:verify-work`:** 3 scripts in `niches/ai-commentary/scripts/` with correct YAML + system-prompt.md generated (STATUS header replaced)
- **Max feedback latency:** ~5 min per verification pass

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 2-01-01 | 01 | 1 | ANLZ-01 | manual | Invoke `/script-generator`, select niche + Analyzer | ❌ Wave 0 | ⬜ pending |
| 2-01-02 | 01 | 1 | ANLZ-02 | manual | Paste guided fill input; confirm single-message collection | ❌ Wave 0 | ⬜ pending |
| 2-01-03 | 01 | 1 | ANLZ-03 | manual | Inspect draft for [HOOK]/[BUILD]/[PAYOFF] and 6 analysis fields | ❌ Wave 0 | ⬜ pending |
| 2-01-04 | 01 | 1 | ANLZ-04 | manual | Input views ≥1M, 100K–1M, <100K; verify tier 1/2/3 assigned | ❌ Wave 0 | ⬜ pending |
| 2-01-05 | 01 | 1 | ANLZ-05 | smoke | `ls niches/ai-commentary/scripts/ && head -10 niches/ai-commentary/scripts/001-*.md` | ❌ Wave 0 | ⬜ pending |
| 2-01-06 | 01 | 1 | ANLZ-06 | smoke | `cat niches/ai-commentary/system-prompt.md` (confirm STATUS header gone after 3rd script) | ❌ Wave 0 | ⬜ pending |
| 2-01-07 | 01 | 1 | ANLZ-07 | manual | Add test phrase to anti-patterns.md; verify inline flag appears in draft | ❌ Wave 0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- No test framework to install — this phase produces no application code
- Verification relies on file inspection (`ls`, `head`, `cat`) and manual Claude Code session walkthrough
- Before testing ANLZ-07: add at least one phrase to `niches/ai-commentary/anti-patterns.md`

*No Wave 0 file stubs needed. Existing directory structure from Phase 1 covers all requirements.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Skill invocation via `/script-generator` | ANLZ-01 | Claude Code UI interaction; no CLI test hook | Run `/script-generator` in Claude Code, confirm opening prompt appears |
| Single-message input collection | ANLZ-02 | Interactive conversational session | Paste views + notes + transcript in one message; confirm Claude parses all 3 |
| HOOK/BUILD/PAYOFF segmentation with analysis | ANLZ-03 | LLM output — semantic quality requires human inspection | Review draft; verify 3 segments labelled + 6 analysis fields present |
| Quality tier assignment | ANLZ-04 | Computed from free-form view input; no separate test surface | Test all 3 tier thresholds with explicit view counts |
| Anti-pattern flagging | ANLZ-07 | Requires live session with non-empty anti-patterns.md | Add test entry, run Analyzer with matching phrase, verify ⚠️ flag in draft |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5 min
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending

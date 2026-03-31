---
phase: 1
slug: memory-foundation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-31
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | none — Phase 1 is static file creation only |
| **Config file** | none |
| **Quick run command** | `ls niches/ai-commentary/` |
| **Full suite command** | manual inspection (see Manual-Only Verifications below) |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `ls niches/ai-commentary/` to confirm files created
- **After every plan wave:** Run full manual inspection checklist below
- **Before `/gsd:verify-work`:** All manual verification rows must be checked
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 1-01-01 | 01 | 1 | MEM-01 | manual | `ls niches/ai-commentary/` | ❌ W0 | ⬜ pending |
| 1-01-02 | 01 | 1 | MEM-02 | manual | `ls niches/ai-commentary/scripts/` | ❌ W0 | ⬜ pending |
| 1-01-03 | 01 | 1 | MEM-03 | manual | `cat niches/ai-commentary/anti-patterns.md` | ❌ W0 | ⬜ pending |
| 1-01-04 | 01 | 1 | MEM-03 | manual | `cat niches/ai-commentary/feedback.md` | ❌ W0 | ⬜ pending |
| 1-01-05 | 01 | 1 | MEM-04 | manual | `cat niches.md` | ❌ W0 | ⬜ pending |
| 1-01-06 | 01 | 1 | MEM-05 | manual | `cat niches/ai-commentary/scripts/example-script.md` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- Phase 1 has no test framework — all verification is manual inspection.

*Existing infrastructure covers all phase requirements via manual file inspection.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| `niches/ai-commentary/` directory exists with all files | MEM-01 | Static files, no runtime | `ls niches/ai-commentary/` — confirm scripts/, anti-patterns.md, feedback.md, patterns.md, system-prompt.md |
| `niches.md` lists ai-commentary niche | MEM-02 | Markdown content check | Open `niches.md`, confirm ai-commentary row exists with correct format |
| `anti-patterns.md` has correct schema header | MEM-03 | Content format check | Open file, confirm YAML or schema header section present |
| `feedback.md` has correct schema header | MEM-03 | Content format check | Open file, confirm schema header and empty list present |
| Example script has valid YAML frontmatter | MEM-05 | Schema correctness | Open training script, confirm title/views/quality_tier/hook_type/date_analyzed fields present with correct types |
| `scripts/` directory visible in git | MEM-04 | Git behavior | `git status niches/` — confirm .gitkeep or placeholder file tracked |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending

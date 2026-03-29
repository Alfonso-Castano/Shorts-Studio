# Shorts Studio — Script Writer

## Current Milestone: v1.0 Script Generator

**Goal:** Build a `/script-generator` Claude Code skill that learns from viral training scripts and generates new YouTube Shorts commentary scripts using niche-isolated memory on disk.

**Target features:**
- Niche-isolated memory structure (`niches/ai-commentary/`)
- Analyzer workflow: ingest viral transcripts → extract patterns → write structured memory files
- Generator workflow: read memory → take video summary → output 2-3 script variations
- System prompt auto-generated from first batch of training scripts
- Anti-patterns enforcement and feedback tracking (token-efficient)

## What This Is

A script generation tool for YouTube Shorts in the AI commentary niche. Takes video context (either AI-analyzed video or a user-provided summary) combined with learned niche patterns, and outputs engaging timed commentary scripts. This is the first standalone component of a larger pipeline that will eventually automate the full Shorts creation workflow.

## Core Value

Generate scripts that maximize viewer retention and engagement — the script is the highest-leverage part of the content and drives views and profit.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Claude Code skill (`/script-generator`) as single entry point with niche selection
- [ ] Niche-isolated memory structure on disk (`niches/{niche}/`)
- [ ] Analyzer workflow: ingest viral transcript + views + notes → structured memory file
- [ ] Auto-generate niche `system-prompt.md` from first batch of training scripts
- [ ] Generator workflow: read memory → video summary input → 2-3 script variations
- [ ] Each variation structured as [HOOK] / [BUILD] / [PAYOFF] with section annotations
- [ ] Voice check pass: flag phrases that sound generic after generation
- [ ] Anti-patterns enforcement on every generation call
- [ ] Feedback tracking: brief per-niche notes, max 5 bullets, included in generation

### Out of Scope

- Edit brief / editing instructions — separate future component
- Captions (1-3 word chunks) — separate future component
- ElevenLabs / voiceover integration — separate future component
- UI (web or otherwise) — interface decision deferred
- Orchestrator combining all components — built after all components exist
- Any niche other than ai-commentary — future expansion
- Full app architecture — building component by component

## Context

- User currently creates YouTube Shorts entirely manually: find TikTok content → download → write script → voiceover via ElevenLabs → edit in CapCut → upload
- The script is the most critical part — it drives engagement, retention, and ultimately profit
- Source videos are TikTok clips, typically 15-30 seconds
- Niche memory stores: hook structures, pacing patterns, high-performing vocabulary
- Training data comes from both user's own top-performing shorts AND competitor videos
- Video analysis approach depends on cost-efficiency of AI vision — need to research whether processing 15-30s clips is token-viable
- Tech stack and interface decisions are deferred to phase discussions
- User wants to explore how Claude Code agents, skills, and MCPs could assist both in building and in the final product architecture

## Constraints

- **Scope**: Script generation ONLY for this milestone — no bundling of edit briefs, captions, or integrations
- **LLM**: Start with Claude API, but architecture must allow easy provider swapping for experimentation
- **Architecture**: Tentative — expected to evolve as phases are discussed and built
- **Efficiency**: Video analysis must not blow token budgets on a single clip
- **Niche isolation**: Memory systems must be completely separate per niche

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build components individually, not monolithic | Validate each piece works before integrating; allows architecture to evolve | — Pending |
| Start with Claude API for LLM | User's current preference; will experiment with others over time | — Pending |
| Script-only output (no edit brief/captions) | Keep milestone tightly scoped to one component | — Pending |
| Defer tech stack to phase discussions | User wants to discuss with Claude before committing | — Pending |
| Dual input: video file OR text summary | Hedge against video analysis being too expensive | — Pending |

---
*Last updated: 2026-03-29 after milestone v1.0 started — Script Generator*

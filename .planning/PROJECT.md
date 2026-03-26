# Shorts Studio — Script Writer

## What This Is

A script generation tool for YouTube Shorts in the AI commentary niche. Takes video context (either AI-analyzed video or a user-provided summary) combined with learned niche patterns, and outputs engaging timed commentary scripts. This is the first standalone component of a larger pipeline that will eventually automate the full Shorts creation workflow.

## Core Value

Generate scripts that maximize viewer retention and engagement — the script is the highest-leverage part of the content and drives views and profit.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Accept video context as input (video file or text summary)
- [ ] Analyze video to identify key engaging moments (if video input is viable)
- [ ] Generate timed commentary script matching video visually
- [ ] Niche memory system that learns patterns from training videos
- [ ] Hook that lands in first 2-3 seconds
- [ ] Tone: mix of hype and conversational
- [ ] Memory builder that extracts patterns from training videos (user's + competitors')
- [ ] Niche isolation — each niche has its own memory, never shared
- [ ] Design for easy LLM provider swapping (start with Claude API)

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
*Last updated: 2026-03-26 after initialization*

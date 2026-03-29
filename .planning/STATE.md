# State: Shorts Studio — Script Writer

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-29)

**Core value:** Generate scripts that maximize viewer retention and engagement — the script is the highest-leverage part of the content.
**Current focus:** Milestone v1.0 — Script Generator

## Current Position

Phase: Not started (defining requirements)
Plan: —
Status: Defining requirements
Last activity: 2026-03-29 — Milestone v1.0 started

## Accumulated Context

- Platform: Claude Code skills (Option A) — free with subscription, no API costs, no infrastructure
- Memory: Files on disk in `niches/{niche}/` directories
- Entry point: single `/script-generator` skill, routes to Analyzer or Generator workflow
- Niche selection happens at skill invocation
- System prompt: auto-generated from first batch of training scripts, updated on demand only
- Anti-patterns: user-filled manually; analyzer flags new candidates
- Feedback: max 5 bullets per niche, user-controlled, appended via conversational instruction
- Quality tiers: soft preference only (Tier 1 first, then Tier 2) — not strict filtering
- Max 3-5 training examples per generation call (lost-in-the-middle prevention)
- Research complete: STACK.md, FEATURES.md, ARCHITECTURE.md, PITFALLS.md all exist
- Detailed implementation plan: C:\Users\alfon\.claude\plans\wise-weaving-starfish.md

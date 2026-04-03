# Roadmap: Shorts Studio — Script Generator

## Overview

Four phases deliver a working `/script-generator` Claude Code skill. Phase 1 establishes the niche memory structure on disk. Phase 2 makes the Analyzer workflow functional — ingesting viral transcripts and writing structured script files. Phase 3 adds the Generator workflow to the same skill, completing the end-to-end path from memory to script variations. Phase 4 validates the system works in real usage and the output quality is high enough to use.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Memory Foundation** - Niche directory structure, schemas, and placeholder files on disk (completed 2026-03-31)
- [x] **Phase 2: Analyzer Workflow** - `/script-generator` skill with Analyzer path fully functional (completed 2026-04-01)
- [ ] **Phase 3: Generator Workflow** - Generator path added to the skill, end-to-end script generation working
- [ ] **Phase 4: Tune & Validate** - Real usage testing, prompt refinement, quality verification

## Phase Details

### Phase 1: Memory Foundation
**Goal**: The niche memory structure exists on disk and is ready to be populated
**Depends on**: Nothing (first phase)
**Requirements**: MEM-01, MEM-02, MEM-03, MEM-04, MEM-05
**Success Criteria** (what must be TRUE):
  1. `niches/ai-commentary/` directory exists with all expected subdirectories and placeholder files
  2. `niches.md` lists ai-commentary as an available niche and a user can add a new niche by editing it
  3. `anti-patterns.md` and `feedback.md` exist per niche with documented schemas ready for user content
  4. A training script file can be placed in `niches/ai-commentary/scripts/` with correct YAML frontmatter and is recognized by the schema
**Plans**: 2 plans

Plans:
- [x] 01-01-PLAN.md — Directory structure, niches.md registry, and training script schema
- [x] 01-02-PLAN.md — Placeholder files (anti-patterns, feedback, patterns, system-prompt)

### Phase 2: Analyzer Workflow
**Goal**: User can invoke `/script-generator`, pick a niche, and run the Analyzer to ingest a viral transcript into structured memory
**Depends on**: Phase 1
**Requirements**: ANLZ-01, ANLZ-02, ANLZ-03, ANLZ-04, ANLZ-05, ANLZ-06, ANLZ-07
**Success Criteria** (what must be TRUE):
  1. User invokes `/script-generator`, selects ai-commentary niche, selects Analyzer, and the skill prompts for transcript + view count + notes in one message
  2. After pasting a transcript, Claude segments it into [HOOK] / [BUILD] / [PAYOFF] with written analysis and assigns the correct quality tier
  3. A structured script file appears in `niches/ai-commentary/scripts/` with all required YAML frontmatter fields populated
  4. When no `system-prompt.md` exists, the first Analyzer run generates one from the ingested scripts
  5. If the transcript contains phrases matching `anti-patterns.md`, the Analyzer flags them before saving
**Plans**: 2 plans

Plans:
- [x] 02-01-PLAN.md — Skill entry point, routing, Analyzer input collection, analysis, and draft display (completed 2026-04-01)
- [ ] 02-02-PLAN.md — Analyzer write path, patterns.md synthesis, system-prompt.md generation, human verification

### Phase 3: Generator Workflow
**Goal**: User can invoke `/script-generator`, pick a niche, and run the Generator to receive 2-3 ready-to-use script variations from memory
**Depends on**: Phase 2
**Requirements**: GEN-01, GEN-02, GEN-03, GEN-04, GEN-05, GEN-06, GEN-07
**Success Criteria** (what must be TRUE):
  1. User invokes `/script-generator`, selects ai-commentary niche, selects Generator, and the skill reads system-prompt, anti-patterns, and feedback before prompting for video summary
  2. After providing a 3-5 sentence video summary, Claude outputs 2-3 script variations each with timed [HOOK] / [BUILD] / [PAYOFF] sections and a brief annotation per section
  3. Voice check runs automatically and flags any phrases that sound generic or interchangeable with other AI commentary channels
  4. User can say "remember for next time that..." and the skill appends the note to `feedback.md`, trimmed to 5 bullets
**Plans**: 2 plans

Plans:
- [ ] 03-01-PLAN.md — To be planned
- [ ] 03-02-PLAN.md — To be planned

### Phase 4: Tune & Validate
**Goal**: The skill produces script output that is demonstrably shaped by niche memory and usable with minimal rewrites
**Depends on**: Phase 3
**Requirements**: TUNE-01, TUNE-02
**Success Criteria** (what must be TRUE):
  1. Running the Generator with memory enabled produces visibly different scripts compared to running without memory context — hooks, vocabulary, and pacing reflect the training scripts
  2. 5 real scripts are generated and the user can use them with less than 40% manual rewriting on average
**Plans**: 2 plans

Plans:
- [ ] 04-01-PLAN.md — To be planned
- [ ] 04-02-PLAN.md — To be planned

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Memory Foundation | 2/2 | Complete   | 2026-03-31 |
| 2. Analyzer Workflow | 2/2 | Complete   | 2026-04-03 |
| 3. Generator Workflow | 0/? | Not started | - |
| 4. Tune & Validate | 0/? | Not started | - |

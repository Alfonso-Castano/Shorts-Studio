## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update persistent knowledge (`STATE.md` or lessons tracking)
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons/state at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management (GSD-Aligned)

1. **Discuss First**
   - Clarify requirements, edge cases, and preferences before planning
   - Capture decisions in structured context (e.g. `CONTEXT.md`)
   - Eliminate ambiguity before execution

2. **Plan Phase**
   - Break work into atomic, verifiable tasks (`PLAN.md`)
   - Ensure each task has:
     - Clear action
     - Defined files
     - Explicit verification steps
   - Keep tasks small enough for isolated execution

3. **Execute in Isolation**
   - Execute tasks in clean, fresh contexts (avoid context rot)
   - Prefer parallel execution where tasks are independent
   - One task = one focused execution unit

4. **Verify Rigorously**
   - Validate against requirements, not assumptions
   - Confirm behavior works end-to-end (not just code correctness)
   - Use tests, logs, and real usage scenarios

5. **Track State, Not Checklists**
   - Persist decisions, progress, and blockers in `STATE.md`
   - Use summaries (`SUMMARY.md`) instead of manual progress logs
   - Maintain system-level awareness of project status

6. **Iterate Until Correct**
   - If verification fails:
     - Diagnose root cause
     - Generate fix plan
     - Re-execute immediately
   - Continue until requirements are satisfied

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
# Pitfalls Research

**Domain:** AI-powered short-form video script generation (YouTube Shorts, AI commentary niche)
**Researched:** 2026-03-26
**Confidence:** HIGH for LLM/API pitfalls (official docs + research); MEDIUM for workflow/UX pitfalls (community + patterns)

---

## Critical Pitfalls

### Pitfall 1: Typicality Bias — The LLM Writes Generic AI Commentary, Not Your Commentary

**What goes wrong:**
The model produces scripts that could have been written by any AI commentary channel. Phrases like "this is wild," "let me break this down," and "you won't believe this" appear in every output. Tone oscillates between hype and flatness in a predictable pattern. The output sounds like "AI writing" rather than a specific creator's voice. Research confirms this: LLMs exhibit systematic "typicality bias" — a preference for predictable, statistically common outputs over distinctive ones. The AI commentary niche amplifies this because the training data is already saturated with the same hook structures.

**Why it happens:**
RLHF alignment steers models toward average, safe outputs. Without concrete anchors (real examples from this creator, explicit vocabulary rules, forbidden phrases), the model defaults to what it has seen most. A vague system prompt like "write in a hype, conversational tone" is not enough — that describes 10,000 channels.

**How to avoid:**
The niche memory system must include anti-patterns (phrases to actively avoid), not just positive examples. For each hook structure stored, also store what makes it distinct from generic hooks. Provide 3-5 verbatim lines from the creator's actual top-performing scripts as part of every generation prompt — not summarized patterns, actual lines. Use a "voice check" evaluation pass: after generating, re-prompt asking "does this sound generic? List 3 phrases that could come from any AI commentary channel."

**Warning signs:**
- Generated hook contains "you won't believe," "buckle up," "here's what's crazy," or similar phrases without explicit instruction
- Two scripts generated for different videos feel interchangeable in tone
- The creator reads the output and says "it doesn't sound like me"
- Scripts have no vocabulary that is specific to the AI niche (no technical terms, no platform-specific language)

**Phase to address:**
Script generation core phase (Phase 1 or 2). The memory schema and prompt architecture must enforce voice specificity from day one — retrofitting is expensive.

---

### Pitfall 2: Video Analysis Token Blowout

**What goes wrong:**
The naive approach to video analysis — extract frames at 1fps and send them all to the vision API — blows up the token budget on a 15-30 second TikTok clip. At 1fps, a 30-second clip = 30 frames. Using Claude's formula (width × height / 750), a 1080×1920 portrait TikTok frame at full resolution costs ~2,765 tokens per frame. 30 frames × 2,765 = ~82,950 tokens just for the images, at $3/M input tokens on Sonnet = ~$0.25 per video analyzed. At scale (analyzing 20 training videos + every new source clip), costs accumulate fast and latency becomes unacceptable.

**Why it happens:**
The frame-extraction approach feels natural — "just send the video as images." Developers don't calculate the per-clip cost upfront. The mistake is treating video analysis like text processing.

**How to avoid:**
Downsample aggressively before passing to the API. For a 9:16 portrait clip, target 384×682 pixels — that's ~349 tokens per frame, an 8x reduction. Use sparse sampling: 3-5 key frames from a 30-second clip is sufficient to identify content type, on-screen text, and visual context — not every frame. The official docs confirm images can be resized to as small as 200px before quality degrades. For the dual-input design already planned (video file OR text summary), the text summary path is the guaranteed cost-safe fallback and should be the default for training/memory building where the user can write the summary themselves.

**Warning signs:**
- Per-analysis cost exceeds $0.05 (it should be <$0.01 with smart sampling)
- API calls for a single video take more than 10 seconds to return
- Context window warnings appear because image tokens are exhausting the limit
- You're sending more than 5 frames for a sub-30-second clip

**Phase to address:**
Video analysis feasibility phase (Phase 2 or 3 if video input is built). Must establish a hard token budget per analysis call before building the feature.

---

### Pitfall 3: Niche Memory That Stores Patterns But Doesn't Change Outputs

**What goes wrong:**
You build a memory system that dutifully stores hook structures, pacing patterns, and vocabulary from training videos. The memory grows. But scripts generated with memory enabled are not measurably better than scripts generated without it. The memory becomes a false confidence layer — it looks like the system is learning, but output quality has plateaued.

**Why it happens:**
Three failure modes compound:
1. **Pattern extraction is too abstract.** "Hook type: curiosity gap" is not actionable for generation. The model already knows about curiosity gaps. What it needs is the specific *execution* — the exact sentence structure, the vocabulary density, the length.
2. **Memory is retrieved but not weighted.** When the system prompt includes 20 stored patterns, the "lost in the middle" problem causes the model to weight only the first and last items, ignoring middle patterns that may be most relevant.
3. **No quality signal.** Memory accumulates examples without knowing which ones performed. High-performing and low-performing scripts are treated equally.

**How to avoid:**
Store verbatim examples, not abstracted patterns. "Hook: [Creator] always opens with a 1-sentence statement of the bizarre fact, then a direct-address question. Example: 'This AI just passed the bar exam. Who's getting fired first?'" is far more useful than "Hook: curiosity gap, direct address." Limit retrieval to 3-5 examples per generation call — more causes dilution. Tag all training examples with a quality tier (tier 1 = top-performing, tier 2 = typical) and weight tier 1 examples first. Build a test harness early: generate 5 scripts with memory disabled vs. 5 with memory enabled, compare side-by-side.

**Warning signs:**
- Memory has 20+ patterns stored but scripts don't noticeably incorporate them
- Retrieval returns patterns from the "middle" of a long memory document
- No differentiation between examples from viral videos vs. average-performing videos
- The creator reviews generated scripts and says "it's not using my style"

**Phase to address:**
Memory system design phase. The storage schema and retrieval strategy must be decided before building — changing them after content is stored means re-processing all training data.

---

### Pitfall 4: Timed Script Drift — Timestamps That Don't Match What Happens on Screen

**What goes wrong:**
The system generates a script with timestamps (e.g., "[0:03] Oh no way, look at this—") but the timestamps are either fabricated or approximate. When the creator uses the script, the commentary doesn't align with what's visually happening. The voiceover says "watch what happens next" three seconds before the thing actually happens on screen, or calls out a visual element that the viewer can't see yet.

**Why it happens:**
The LLM has no ground truth about what happens at specific timestamps unless the input explicitly provides this. Even with frame extraction, the model is seeing static frames without understanding the precise moment when a visual event occurs. The model generates plausible-sounding timestamps without understanding what they mean for a human editor trying to sync audio.

**How to avoid:**
Make the timestamp system explicit in the input. The user (or a pre-processing step) should provide a rough timeline: "0:00-0:05: person walks in, 0:05-0:12: the reveal happens, 0:12-0:30: reaction." Scripts generated from this timeline will have defensible timestamps. If video analysis is used, the extracted frames should be labeled with their exact timestamp before being passed to the model. Treat timestamps as editing cues ("align with the reveal") rather than absolute second-marks, since actual sync happens in CapCut anyway.

**Warning signs:**
- Generated script has timestamps at 1-second intervals suggesting they were fabricated
- "Timestamps" in the script don't reference any visual events — they're just word-count checkpoints
- The creator reports needing to manually re-time every generated script before using it

**Phase to address:**
Script format design phase (Phase 1). The output schema for scripts must handle timing annotations in a way that is honest about what's estimated vs. verified.

---

### Pitfall 5: Over-Engineering for a Solo Creator Workflow

**What goes wrong:**
The system is built with the architecture of a SaaS product: a vector database for memory, a web API, a queue system, a separate embedding service, and environment management for multiple deployment stages. The creator needs to run an orchestration layer just to generate one script. The tool becomes a liability — it breaks when any one service goes down, requires maintenance overhead, and makes iteration slow.

**Why it happens:**
Developers default to "production-grade" patterns even when the user is one person running 5-10 scripts per week. The component-by-component approach specified in PROJECT.md is correct philosophy — but each component can still be over-engineered individually.

**How to avoid:**
For a solo creator tool with no uptime requirement, a JSON file on disk is a legitimate memory store. SQLite is a legitimate database. A Python or Node script run from the command line is a legitimate interface. The first working version should have zero running services — everything reads from disk, calls the API, and writes back to disk. Introduce infrastructure only when a specific scaling need is proven, not anticipated. The out-of-scope list in PROJECT.md (no UI, no orchestrator, no integrations) is the right call — honor it rigorously and resist the urge to "just add" these things.

**Warning signs:**
- The development environment requires more than two commands to start
- A memory system is being designed before a single working script has been generated
- Architecture discussions reference horizontal scaling, sharding, or multi-tenancy
- More time is spent on infrastructure than on prompt quality

**Phase to address:**
Phase 1 (foundation). This is a mindset pitfall that must be consciously addressed in the first phase. The default should be the simplest possible implementation — upgrade only when proven necessary.

---

### Pitfall 6: Hook Over-Optimization at the Expense of Retention

**What goes wrong:**
The system is tuned to generate strong hooks — and it does. The first 2 seconds are punchy and stop the scroll. But the middle 10-20 seconds of the script lose momentum, and viewers swipe away before the payoff. YouTube's algorithm (which measures the full view rate, not just hook retention) buries the video despite the strong open. The system optimizes for the metric that's easiest to measure (hook quality) at the expense of the metric that actually drives growth (completion rate, re-watches).

**Why it happens:**
Hook structures are well-documented, easy to extract as patterns, and easy to evaluate qualitatively. Middle-of-script pacing is harder to articulate as a rule and harder to teach to a model. Memory systems naturally accumulate more hook patterns than mid-script patterns because hooks are the most discussed part of Short-form content strategy.

**How to avoid:**
Explicitly build memory for three-beat arc structure: hook (0-3s), escalation/payoff (3-20s), final punch (last 2-3s). Train the model to identify and store *what happens in seconds 5-15* of high-performing videos, not just the hook. In the script output format, require all three segments to be present and labeled — this forces the model to reason about the full arc rather than front-loading effort on the hook. Evaluate generated scripts for the "buried lead" problem: if the most interesting information is after the 15-second mark, the script has failed.

**Warning signs:**
- Generated scripts consistently have strong first lines but flat middles
- Memory patterns are 80% hook examples and 20% everything else
- The creator reports good initial retention but high drop-off after 8-10 seconds
- Script word count suggests the "payoff" is compressed to the last 2-3 seconds

**Phase to address:**
Script generation and memory system phases. The memory schema must capture full-arc patterns, and the generation prompt must enforce three-beat structure.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcode Claude API, ignore provider abstraction | Faster to ship Phase 1 | All experiments with other models require refactoring | Never — PROJECT.md explicitly requires swappable providers; the abstraction layer is cheap to add upfront |
| Store memory as unstructured text in a single file | Simple to implement | Pattern retrieval degrades as file grows; no quality weighting; hard to update individual examples | Acceptable in Phase 1 only if the schema is designed to be migrated |
| Generate scripts without a quality evaluation pass | Faster generation loop | Outputs plateau at mediocre quality; no signal for memory improvement | Never for production use, acceptable in early testing |
| Skip timestamp annotations in script output | Simpler output format | Creator must manually figure out sync every time, negating the tool's time savings | Never — timestamps are core to the use case |
| Use full-resolution frames for video analysis | Marginally better vision quality | Token costs become unsustainable after a few dozen videos | Never — downsample to 384px shortest side before sending |
| Accumulate all training examples without quality filtering | Simple to build | Memory becomes diluted with average-performing content; generic outputs follow | Never after initial testing phase |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Claude Vision API | Sending full-resolution portrait video frames (1080×1920) thinking more pixels = better analysis | Downsample to 384×682 (~349 tokens) before calling. Quality is not meaningfully better at full resolution for content identification tasks |
| Claude Messages API | Injecting the entire memory document into every generation call | Retrieve only the 3-5 most relevant examples per call. Use quality-tier filtering to ensure only top-performing examples surface |
| Claude API (provider swap) | Wrapping the API call in a thin function and calling it done | The real abstraction point is the *prompt template*, not just the HTTP client. Different providers have different system prompt behaviors and token limits — abstract at the prompt-builder level, not just the HTTP level |
| Frame extraction (ffmpeg or similar) | Extracting at 1fps and sending all frames | Extract 3-5 keyframes using scene-change detection rather than fixed interval; label each with its timestamp before sending to the model |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| No-cache memory retrieval: re-reading the entire memory file on every generation call | Latency increases as memory grows; no apparent upper bound on generation time | Keep memory index (metadata + quality tier) separate from full examples; load only what's needed | Noticeable at 50+ stored examples |
| Sending all frames in a single API call | Request size limit errors (32 MB); context window exhaustion; slow time-to-first-token | Process a small number of frames (3-5) per call; use Files API for repeated use of the same source video | At 10+ frames per clip or full-resolution images |
| Prompt length creep | Generation quality becomes inconsistent; outputs feel less focused; costs climb silently | Monitor token count of the combined system prompt + memory + user input before each call; set a hard budget (e.g., 4,000 input tokens max) | When the combined prompt exceeds ~8,000 tokens the "lost in the middle" effect becomes measurable |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Storing API keys in the script or config files committed to git | Key leaked publicly, unexpected charges, key revocation needed | Use environment variables or a .env file, add .env to .gitignore from the start |
| Niche memory stored in a shared or cloud-synced location without access control | Training data (which may include competitor analysis and proprietary hook strategies) exposed | Keep memory files local by default; document clearly that cloud sync paths should be excluded |
| No rate-limit handling in the API client | A burst of generation calls (e.g., batch-processing training videos) hits API rate limits, fails silently or with unhelpful errors | Implement exponential backoff from day one; log rate-limit responses explicitly |

---

## UX Pitfalls

*Note: This tool has no UI in scope, but "UX" here means the creator's workflow experience using the CLI/script output.*

| Pitfall | Creator Impact | Better Approach |
|---------|----------------|-----------------|
| Script output is a wall of text with no visual structure | Creator has to parse the output to find the hook, middle, and close | Output a clearly segmented format: [HOOK], [BUILD], [PAYOFF], each on its own line with a timestamp estimate |
| No explanation of why a pattern was used | Creator can't learn from the output or improve the memory system | Include a brief annotation per script section: "Hook: curiosity gap + direct address (from training video: 'AI passes bar exam' format)" |
| Generated script requires the creator to rewrite every line | The tool adds work instead of removing it | Run a "usability threshold" test: if the creator edits more than 40% of lines, the prompt needs work before shipping |
| Text summary input path is buried as a fallback | Creator feels like they're using a "downgraded" feature | Text summary should be the primary, recommended input — it's faster, cheaper, and often produces better results because the creator provides exactly the context needed |

---

## "Looks Done But Isn't" Checklist

- [ ] **Hook generation:** Often missing — verify the hook is specific to the *source video's content*, not a generic hook template applied to any topic
- [ ] **Niche memory:** Often missing — verify the memory actually *changes the output* by running a comparison test (memory enabled vs. disabled) and diffing the results
- [ ] **Timestamps:** Often missing — verify timestamps correspond to actual visual events, not evenly-spaced line breaks
- [ ] **Voice consistency:** Often missing — verify the creator can read the script aloud and it sounds like them, not like generic AI commentary
- [ ] **Full-arc structure:** Often missing — verify there is a meaningful escalation between the hook and the close, not just hook + padding + payoff
- [ ] **Provider swap:** Often missing — verify swapping the LLM client from Claude to another provider requires changing only one config value, not prompt rewrites
- [ ] **Token budget:** Often missing — verify total input tokens per generation call are logged and within budget

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Typicality bias baked into generation prompts | MEDIUM | Add anti-pattern list to system prompt; add verbatim voice examples; run 10-script comparison to validate improvement |
| Token blowout from full-resolution frames | LOW | Add downsampling step before frame extraction; rebuild analysis pipeline with capped frame count |
| Memory system accumulates without quality signal | HIGH | Re-tag all stored examples with quality tiers; rebuild retrieval to weight by tier; this requires re-processing all training data |
| Over-engineered infrastructure blocking iteration | HIGH | Revert to file-based memory and CLI runner; strip services until the tool runs with zero background processes |
| Timestamps are fabricated/wrong | LOW | Remove timestamp generation from the LLM; require timestamps to come from user input or frame metadata; regenerate affected scripts |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Typicality bias / generic output | Phase 1 (prompt architecture) | Side-by-side comparison: same video, generic prompt vs. voice-anchored prompt — creator can't tell which is which = fail |
| Token blowout on video analysis | Phase 2 (video input) | Cost per analysis call < $0.02; latency < 8 seconds per clip |
| Memory that doesn't improve outputs | Phase 2 (memory system design) | Measurable difference between memory-enabled and memory-disabled generation on the same input |
| Timed script drift | Phase 1 (output schema) | Creator uses 3 generated scripts without manually re-timing them = pass |
| Over-engineering for solo workflow | Phase 1 (foundation) | Tool runs with a single command and zero background services = pass |
| Hook over-optimization | Phase 2 (generation quality) | Generated scripts have non-trivial content between hook and payoff; creator reports usable mid-script pacing |
| Niche isolation failure | Phase 2 (memory architecture) | Adding a new niche never touches the existing niche's stored examples — verified by inspection |

---

## Sources

- Anthropic Vision API official documentation: https://platform.claude.com/docs/en/build-with-claude/vision (image token cost formula, frame size limits — HIGH confidence)
- LLM creative writing typicality bias research (2025): https://arxiv.org/html/2602.16162 (MEDIUM confidence — academic preprint)
- "Lost in the middle" attention degradation: Liu et al. (2024) — well-documented, confirmed by multiple sources (HIGH confidence)
- Few-shot prompting diminishing returns: https://www.prompthub.us/blog/the-few-shot-prompting-guide (MEDIUM confidence)
- YouTube Shorts hook retention data: https://miraflow.ai/blog/why-youtube-shorts-fail-first-3-seconds (MEDIUM confidence — community data)
- YouTube Shorts algorithm view counting changes (March 2025): The Verge report, confirmed by https://www.shortimize.com/blog/how-does-youtube-shorts-algorithm-work (MEDIUM confidence)
- Style guide complexity anti-patterns: https://infomanagementcenter.com/bridging-the-gap/ (MEDIUM confidence)
- YouTube AI content monetization policy (2025): https://www.searchenginejournal.com/youtubes-ai-slop-problem-and-how-marketers-can-compete/567297/ (MEDIUM confidence)

---
*Pitfalls research for: AI-powered YouTube Shorts script generation (AI commentary niche)*
*Researched: 2026-03-26*

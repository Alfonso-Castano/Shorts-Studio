# Feature Research

**Domain:** AI Script Generation for Short-Form Video Commentary (YouTube Shorts)
**Researched:** 2026-03-26
**Confidence:** MEDIUM-HIGH (core script structure features HIGH; niche memory and video analysis MEDIUM due to limited direct comparable tools)

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Hook generation (0–3 second) | Every short-form script tool includes this; it's the highest-leverage element — 60%+ three-second retention is the minimum algorithmic threshold | MEDIUM | Hook must be delivered in 2–2.5 seconds, leaving buffer before the 3-second cutoff. Reverse storytelling ("start with payoff") outperforms setup-then-reveal. |
| Timed/timestamped script output | Creators need to know *when* each line lands relative to the source video — this is the core output contract | MEDIUM | Output format: `[MM:SS] spoken line`. Must map commentary to visual moments in the source clip, not just generic narration. |
| Tone/style specification | "Hype + conversational" is a specific voice requirement; generic AI tone is unusable for commentary content | LOW | Encoded in the system prompt for the niche. Not a runtime parameter — it's baked into how the niche memory is structured. |
| Script pacing structure | Fast-paced structure (Hook → Build → Payoff → Abrupt end) is the industry standard for Shorts. Slow builds kill retention. | MEDIUM | Optimal Shorts structure: Hook (0–3s), Intrigue/Build (3–15s), Climax/Content (15–50s), Payoff/Abrupt end (50–60s). No "Hey guys…" openers. |
| Input flexibility (video or text) | Video analysis may be expensive; users need a fallback path for providing context without uploading video | LOW | Dual-mode input is a hedge against token costs. Text summary input is low-complexity fallback; video input is the premium path. |
| Key moment identification | The script must reference specific visual moments from the source video — a script that ignores what's on screen is useless for commentary | HIGH | Core challenge: identify 3–5 peak engagement moments in a 15–30s clip and build the script around them. Requires video understanding. |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Niche memory system (per-niche isolation) | Learns hook structures, vocabulary, and pacing patterns from the user's own top-performing Shorts AND competitors — scripts improve over time rather than starting from scratch every time | HIGH | This is the core competitive moat. Memory is stored in `memory.json` per niche. Never shared across niches. Trained on training videos, not manually configured. |
| Pattern extraction from training videos | Automatically learns what vocabulary, hook styles, and pacing patterns correlate with performance in the specific niche — human-curated intuition encoded into structured memory | HIGH | Memory builder reads training videos (user's + competitor's), extracts hook structures, high-performing vocabulary, pacing rhythms, and writes them to memory.json. |
| Commentary-specific output format | 1–3 word caption chunks written into the script so the creator can feed directly to CapCut without reformatting — no translation step | LOW | Market gap: most tools output prose scripts. This outputs caption-ready chunks already broken into visual display units. Deferred to a future component but worth flagging as a differentiator. |
| Edit brief output | Separate companion document with editing instructions (what to cut to, when, pacing notes) so the creator has a production plan, not just a script | MEDIUM | Currently out of scope for Phase 1 but is a meaningful differentiator — most competitors stop at the script. |
| Multi-variant generation | Generate 2–3 script variations per video with different hook styles (question hook vs. bold statement vs. pattern interrupt) — creator picks the winner | MEDIUM | Reduces the risk of any single hook failing. Low additional cost if the core generation is already working. Tools like Subscribr show 6+ hook style variants as a standard feature. |
| Virality/quality self-assessment | Script scores its own hook strength, pacing, and CTA placement, flagging weak points before the creator wastes time recording | MEDIUM | OpusClip's Virality Score and quso.ai's engagement predictor show this is an expected feature in adjacent tools. For script-only output, this can be a structured critique embedded in the output (no ML scoring needed — just LLM self-critique). |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Generic tone/style settings (sliders or dropdowns) | Seems like user control | Tone is niche-specific, not universal. "Hype level: 7" is meaningless — what makes commentary feel authentic is learned from training data, not dialed in | Encode tone in the system prompt; let the niche memory system express style through learned vocabulary and patterns |
| Full video editing automation (CapCut integration) | Creators want end-to-end automation | Scope bloat; CapCut automation is a separate pipeline stage. Bundling it dilutes focus on script quality, which is the actual high-leverage output | Build the script component to a high quality first; add integrations as separate pipeline components once the script is validated |
| Cross-niche memory sharing | Seems like "more data = better" | AI commentary tone and patterns are fundamentally different from, say, fitness commentary. Cross-contamination degrades output quality for both niches | Hard-enforce niche isolation from the start — separate directories, separate memory files, zero shared context |
| Real-time streaming script output | Feels responsive and modern | Adds infrastructure complexity (SSE/WebSockets) with minimal value — scripts are short (60–90 seconds of spoken content) and generate fast anyway | Single-shot generation with a clear "generating…" indicator; save streaming for longer-form content where wait time is significant |
| Auto-post / scheduling integration | Saves time | Requires social API credentials, OAuth flows, platform TOS compliance — a separate engineering domain entirely | Manual posting remains manual for now; focus tooling on content quality, not distribution |
| Emoji/hashtag generation | Common in social media tools | Not relevant to commentary scripts intended for voiceover. Adds noise to the output and suggests the tool doesn't understand its own use case | Omit entirely. If hashtags are needed, they belong in a separate metadata generation step |
| Web UI for script editing | Nice to have for polish | A UI layer before the core script logic is validated is premature investment — the user confirmed CLI/Node.js is fine for this phase | Build the script engine first; defer UI to a phase after the output quality is validated |

## Feature Dependencies

```
[Niche Memory System]
    └──requires──> [Training Video Ingestion]
                       └──requires──> [Video Analysis / Key Moment Extraction]

[Script Generation]
    └──requires──> [Niche Memory System] (for patterns)
    └──requires──> [Video Analysis / Key Moment Extraction] (for visual synchronization)
                       └──alternative──> [Text Summary Input] (cheaper fallback)

[Multi-Variant Generation]
    └──requires──> [Script Generation] (generates multiple instances)

[Virality Self-Assessment]
    └──enhances──> [Script Generation] (post-processing critique layer)

[Edit Brief Output]
    └──enhances──> [Script Generation] (companion document, same video understanding)

[Caption Chunk Formatting]
    └──enhances──> [Script Generation] (reformats output, no new understanding needed)

[Caption Chunk Formatting] ──conflicts──> [Edit Brief Output]
    (Both consume the same script output but serve different downstream steps — keep separate)
```

### Dependency Notes

- **Script Generation requires Niche Memory System:** Without learned patterns, the script is generic AI output indistinguishable from a basic ChatGPT prompt. The memory system is what makes the tool valuable over time.
- **Script Generation requires Video Analysis OR Text Summary:** Video analysis gives the tightest visual synchronization; text summary is the fallback when token cost is prohibitive. Both paths must produce the same output format.
- **Training Video Ingestion requires Video Analysis:** The memory builder needs to extract patterns (hooks, pacing, vocabulary) from training videos — it cannot do this from filenames alone.
- **Multi-Variant Generation enhances Script Generation:** Cheap to add once the core generation loop works — just run it N times with different hook style instructions.
- **Virality Self-Assessment enhances Script Generation:** Can be implemented as a second LLM call that critiques the generated script. No separate ML model needed.

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept.

- [ ] Niche folder structure with isolated memory (niches/ai-commentary/) — required for all other components to have a home
- [ ] Memory builder — reads training videos, extracts hook structures / vocabulary / pacing patterns, writes to memory.json
- [ ] Script generation — takes video input (or text summary fallback) + memory, outputs timed script with timestamps
- [ ] Hook in first 2–3 seconds — non-negotiable structural requirement; failure here means the script is unusable
- [ ] Timed output format — [MM:SS] lines mapped to visual moments in the source video

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Multi-variant generation (2–3 hook styles per run) — add when a single script feels too risky to commit to
- [ ] LLM self-critique / quality assessment — add when the user starts questioning script quality consistently
- [ ] Caption chunk reformatting (1–3 word display chunks) — add when the CapCut workflow is confirmed as the bottleneck

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Edit brief companion output — defer until script quality is validated and the user wants to accelerate the editing step
- [ ] Additional niches (fitness, finance, etc.) — defer until ai-commentary niche proves the memory architecture works
- [ ] ElevenLabs TTS integration — separate pipeline component, builds on validated script output
- [ ] UI layer — defer until it's clear what the user actually interacts with repeatedly
- [ ] Virality/performance tracking feedback loop — defer until enough Shorts are posted to correlate script patterns with actual performance data

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Hook generation (0–3 second) | HIGH | MEDIUM | P1 |
| Timed/timestamped script output | HIGH | MEDIUM | P1 |
| Video key moment identification | HIGH | HIGH | P1 |
| Niche memory system (isolation) | HIGH | HIGH | P1 |
| Training video pattern extraction | HIGH | HIGH | P1 |
| Text summary fallback input | MEDIUM | LOW | P1 |
| Multi-variant generation | MEDIUM | LOW | P2 |
| Script self-critique / quality flag | MEDIUM | LOW | P2 |
| Caption chunk formatting | MEDIUM | LOW | P2 |
| Edit brief output | MEDIUM | MEDIUM | P2 |
| LLM provider swap architecture | LOW (user) / HIGH (developer) | LOW | P1 (architectural) |
| UI layer | LOW | HIGH | P3 |
| ElevenLabs integration | LOW (for this milestone) | MEDIUM | P3 |
| Cross-niche memory | LOW | MEDIUM | Anti-feature |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | Subscribr | OpusClip | Our Approach |
|---------|-----------|----------|--------------|
| Hook generation | Yes — 6 hook style frameworks, psychological hooks extracted from top-performing videos | Yes — analyzes viral hooks, assigns Virality Score | Hooks generated from niche memory (learned from training videos) + hook formula templates in system prompt |
| Niche learning | Channel-level style learning ("learns how you write") | Category-based analysis | Per-niche isolated memory.json with explicit pattern storage — more structured and intentional than channel-level style |
| Video understanding | Imports YouTube transcripts + URLs; not direct video file analysis | Direct video analysis — identifies engaging moments, generates clips | Direct video file analysis (frames + audio) for visual synchronization; text fallback for cost control |
| Pacing analysis | Real-time pacing critique in script editor | Engagement signals from audio/visual patterns | Pacing encoded in system prompt + memory; no real-time editor (CLI output) |
| Timestamp output | Not present — scripts are prose, not timed | Timestamps on output clips | Core output requirement — every script line gets a [MM:SS] marker |
| Style memory | Per-channel (user provides channel context) | Not applicable | Per-niche (fully isolated directories, never shared) |
| Quality scoring | Hook anatomy breakdown, pacing critique | Virality Score (1–100) with explanation | LLM self-critique as a post-generation pass (P2 feature) |
| Caption formatting | Standard prose script | Auto-captions on output clips | 1–3 word chunk formatting (P2, separate component) |

## Sources

- Subscribr feature analysis: https://subscribr.ai/ (fetched directly)
- OpusClip features and Virality Score: https://www.opus.pro/blog/youtube-shorts-hook-formulas and https://skywork.ai/blog/opusclip-review-2025-ai-auto-clipping-virality-score-scheduler/
- YouTube Shorts hook formulas and timing data: https://subscribr.ai/p/perfect-youtube-short-length-structure-hooks and https://www.retentionrabbit.com/blog/youtube-hook-strategy-to-keep-viewers-watching
- Short-form video script structure: https://virvid.ai/blog/ai-shorts-hooks-templates
- Caption chunk display standards (CapCut/TikTok): https://www.opus.pro/blog/tiktok-caption-subtitle-best-practices
- Virality scoring tools: https://quso.ai/products/virality-score and https://www.clipgoat.com/features/ai-virality-score
- AI video analysis capabilities 2026: https://mixpeek.com/blog/video-analysis-ai

---
*Feature research for: AI-powered YouTube Shorts script generation (commentary niche)*
*Researched: 2026-03-26*

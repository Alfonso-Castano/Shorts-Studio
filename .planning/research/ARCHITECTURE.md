# Architecture Research

**Domain:** AI script generation for short-form video (YouTube Shorts / TikTok commentary)
**Researched:** 2026-03-26
**Confidence:** MEDIUM (component patterns HIGH, timing/memory integration MEDIUM)

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                          INPUT LAYER                                │
│  ┌──────────────────────┐      ┌──────────────────────────────────┐ │
│  │   Video File Input   │      │     Text Summary Input           │ │
│  │  (mp4, 15-30s clip)  │      │  (user-written or pasted)        │ │
│  └──────────┬───────────┘      └──────────────┬───────────────────┘ │
└─────────────┼────────────────────────────────-┼─────────────────────┘
              │                                 │
┌─────────────▼─────────────────────────────────▼─────────────────────┐
│                      INPUT PROCESSOR                                 │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Video Analyzer (optional path)                              │    │
│  │  - Frame extraction (1 FPS via ffmpeg/opencv)                │    │
│  │  - Audio transcription (Whisper or provider ASR)             │    │
│  │  - Moment detection (scene change, peak action frames)       │    │
│  │  - Outputs: VideoContext (timestamps + descriptions)         │    │
│  └──────────────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Summary Normalizer (fast path)                              │    │
│  │  - Parses free-text summary into structured VideoContext     │    │
│  │  - Extracts key moments, tone, subject from text             │    │
│  └──────────────────────────────────────────────────────────────┘    │
│  Produces: VideoContext { moments[], duration, tone, subject }       │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│                      NICHE MEMORY LAYER                               │
│  ┌──────────────────────┐    ┌────────────────────────────────────┐  │
│  │  Memory Reader       │    │  Memory Builder (separate flow)    │  │
│  │  - Retrieves niche   │    │  - Ingests training videos         │  │
│  │    patterns for      │    │  - Extracts: hook structures,      │  │
│  │    current context   │    │    pacing, vocabulary, endings     │  │
│  │  - Returns: NichePat │    │  - Writes to NicheStore            │  │
│  │    {hooks[], vocab,  │    │                                    │  │
│  │     pacing, examples}│    │                                    │  │
│  └──────────────────────┘    └────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  NicheStore (per-niche isolated file/DB)                     │    │
│  │  ai-commentary/ → hooks.json, vocab.json, examples/          │    │
│  └──────────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│                     SCRIPT GENERATOR                                  │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Prompt Builder                                              │    │
│  │  - Assembles system prompt from NichePatterns                │    │
│  │  - Injects VideoContext as user context                      │    │
│  │  - Adds few-shot examples from NicheStore                    │    │
│  │  - Specifies output schema (timed segments)                  │    │
│  └──────────────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  LLM Client (provider-agnostic interface)                    │    │
│  │  - Accepts: prompt + generation params                       │    │
│  │  - Returns: structured JSON response                         │    │
│  │  - Adapters: Claude, OpenAI, Gemini, local (swappable)       │    │
│  └──────────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│                     OUTPUT FORMATTER                                  │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Script Validator                                            │    │
│  │  - Checks hook lands in ≤3 seconds                          │    │
│  │  - Validates total duration coverage                         │    │
│  │  - Confirms segment timing adds up                           │    │
│  └──────────────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  Script Serializer                                           │    │
│  │  - Emits TimedScript { segments[], metadata }               │    │
│  │  - Formats: JSON (primary), plain text (human-readable)      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│  Produces: TimedScript ready for downstream pipeline                 │
└──────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Communicates With |
|-----------|----------------|-------------------|
| Video Analyzer | Extracts frames, transcribes audio, detects moments from raw video | Input Processor only |
| Summary Normalizer | Converts free-text summary into structured VideoContext | Input Processor only |
| Input Processor | Routes input type and produces unified VideoContext | Niche Memory Layer |
| Memory Reader | Retrieves relevant niche patterns from NicheStore | Script Generator |
| Memory Builder | Ingests training videos and writes patterns to NicheStore | NicheStore only |
| NicheStore | Persists and isolates per-niche patterns | Memory Reader, Memory Builder |
| Prompt Builder | Assembles final LLM prompt from VideoContext + NichePatterns | LLM Client |
| LLM Client | Provider-agnostic wrapper around LLM APIs | Prompt Builder, Output Formatter |
| Script Validator | Validates timing constraints and structural rules | Output Formatter |
| Script Serializer | Converts LLM output into final TimedScript schema | Downstream pipeline |

## Recommended Project Structure

```
src/
├── input/                    # Input processing pipeline
│   ├── video-analyzer.ts     # Frame extraction, ASR, moment detection
│   ├── summary-normalizer.ts # Text summary → VideoContext
│   ├── input-processor.ts    # Routes input type, produces VideoContext
│   └── types.ts              # VideoContext, Moment, etc.
│
├── memory/                   # Niche memory subsystem
│   ├── memory-reader.ts      # Retrieves NichePatterns from store
│   ├── memory-builder.ts     # Extracts patterns from training videos
│   ├── niche-store.ts        # Read/write abstraction for niche data
│   └── types.ts              # NichePatterns, HookStructure, etc.
│
├── generator/                # Script generation core
│   ├── prompt-builder.ts     # Assembles LLM prompt from context
│   ├── script-generator.ts   # Orchestrates generation flow
│   └── types.ts              # GenerationRequest, GenerationResult
│
├── llm/                      # Provider-agnostic LLM layer
│   ├── client.ts             # LLMClient interface definition
│   ├── claude-adapter.ts     # Anthropic Claude implementation
│   ├── openai-adapter.ts     # OpenAI implementation (future)
│   ├── gemini-adapter.ts     # Google Gemini implementation (future)
│   └── factory.ts            # Creates adapter from config/env
│
├── output/                   # Output formatting and validation
│   ├── script-validator.ts   # Timing and structural checks
│   ├── script-serializer.ts  # TimedScript → JSON / plain text
│   └── types.ts              # TimedScript, ScriptSegment, etc.
│
├── prompts/                  # Versioned prompt templates
│   ├── system/
│   │   └── ai-commentary.md  # System prompt for niche
│   └── examples/
│       └── ai-commentary/    # Few-shot examples per niche
│
├── niches/                   # Niche memory data (gitignored or committed)
│   └── ai-commentary/
│       ├── hooks.json        # Extracted hook structures
│       ├── vocab.json        # High-performing vocabulary
│       └── examples/         # Full script examples
│
└── index.ts                  # CLI / programmatic entry point
```

### Structure Rationale

- **input/:** Isolated so video analysis can be swapped or disabled without touching generation logic. The two input paths (video vs. text) converge here before anything downstream sees the data.
- **memory/:** Completely separate from generation. Memory Builder is a distinct offline workflow; Memory Reader is the runtime lookup. This separation makes it easy to rebuild memory without touching scripts.
- **llm/:** The provider-abstraction layer. Adding a new provider means adding one adapter file and registering it in factory.ts — zero changes elsewhere.
- **prompts/:** Prompt templates are versioned text files, not hardcoded strings. This enables iteration, diffing, and niche-specific variants without code changes.
- **niches/:** Data lives alongside code during early development. Can migrate to a database later without changing the NicheStore interface.

## Architectural Patterns

### Pattern 1: Provider Adapter (LLM Swappability)

**What:** Define a single `LLMClient` interface. Each provider is an adapter implementing that interface. A factory selects the adapter based on config.

**When to use:** Any time the application needs to be LLM-agnostic. This is the standard pattern used by LiteLLM, the Vercel AI SDK, and LangChain.

**Trade-offs:** Adds one indirection layer. Worth it from day one given explicit requirement to swap providers.

**Example:**
```typescript
// llm/client.ts — the interface everything depends on
interface LLMClient {
  complete(prompt: LLMPrompt, params: GenerationParams): Promise<LLMResponse>;
}

// llm/claude-adapter.ts — one implementation
class ClaudeAdapter implements LLMClient {
  async complete(prompt: LLMPrompt, params: GenerationParams): Promise<LLMResponse> {
    // Anthropic SDK call here
  }
}

// llm/factory.ts — selected at runtime from env/config
function createLLMClient(provider: string): LLMClient {
  if (provider === "claude") return new ClaudeAdapter();
  if (provider === "openai") return new OpenAIAdapter();
  throw new Error(`Unknown provider: ${provider}`);
}
```

### Pattern 2: Dual-Path Input Convergence

**What:** Two input paths (video file, text summary) each produce the same `VideoContext` object. The rest of the system only knows about `VideoContext`, never about the raw input type.

**When to use:** When input formats may expand or the expensive path (video analysis) may be bypassed for cost reasons. This project explicitly needs it.

**Trade-offs:** Requires disciplined schema design for `VideoContext` — if the schema is too lossy, script quality suffers on the video path.

**Example:**
```typescript
// Unified output type — downstream only sees this
interface VideoContext {
  duration: number;           // seconds
  subject: string;            // what the video is about
  moments: TimedMoment[];     // [{timestamp, description, intensity}]
  originalTranscript?: string; // if available from video path
  tone: "hype" | "calm" | "mixed";
}

// input-processor.ts routes to the right path
async function processInput(input: VideoInput | SummaryInput): Promise<VideoContext> {
  if (input.type === "video") return analyzeVideo(input.file);
  return normalizeSummary(input.text);
}
```

### Pattern 3: Niche-Scoped Memory Store

**What:** Each niche gets its own isolated storage namespace. Memory is never shared across niches. The NicheStore interface abstracts the storage backend (files today, vector DB later).

**When to use:** Any system that must learn per-domain patterns without cross-contamination. Explicit requirement for this project.

**Trade-offs:** More storage complexity than a global store, but the isolation requirement makes this non-negotiable.

**Example:**
```typescript
// The interface — storage backend is hidden
interface NicheStore {
  getPatterns(niche: string): Promise<NichePatterns>;
  savePatterns(niche: string, patterns: NichePatterns): Promise<void>;
}

// File-system implementation for MVP
class FileNicheStore implements NicheStore {
  private basePath = "./niches";
  async getPatterns(niche: string): Promise<NichePatterns> {
    return JSON.parse(await fs.readFile(`${this.basePath}/${niche}/patterns.json`, "utf8"));
  }
}
```

### Pattern 4: Few-Shot Prompt Assembly

**What:** The Prompt Builder injects retrieved niche examples directly into the LLM prompt as few-shot demonstrations, not as RAG context. For short-form creative tasks (scripts), showing examples of the desired format outperforms vector similarity retrieval.

**When to use:** Creative generation tasks where format, tone, and structure must match a style. Fewer retrieved examples with high quality beats many mediocre ones.

**Trade-offs:** Context window cost per call. For 15-30s scripts with 2-3 examples, this is well within budget. Token cost is not a concern at this scale.

**Example prompt structure:**
```
SYSTEM:
You are a YouTube Shorts scriptwriter for the AI commentary niche.
Style rules: [extracted from NichePatterns]
Hook patterns: [top hook structures]

EXAMPLES:
<example_1>Video context: ... | Script: ...</example_1>
<example_2>Video context: ... | Script: ...</example_2>

USER:
Video context: [VideoContext serialized]
Generate a timed script in this JSON format: [schema]
```

## Data Flow

### Primary Generation Flow

```
User provides input (video file OR text summary)
    │
    ▼
Input Processor
    ├── [video path] → Video Analyzer → VideoContext
    └── [text path]  → Summary Normalizer → VideoContext
    │
    ▼
Memory Reader
    └── reads NicheStore[niche] → NichePatterns
    │
    ▼
Prompt Builder
    └── VideoContext + NichePatterns → assembled LLMPrompt
    │
    ▼
LLM Client (adapter)
    └── LLMPrompt → raw LLM response (JSON)
    │
    ▼
Script Validator
    └── checks timing, hook position, duration coverage
    │
    ▼
Script Serializer
    └── TimedScript { segments[], metadata } → JSON file / stdout
```

### Memory Build Flow (offline, runs separately)

```
User provides training videos (own + competitor)
    │
    ▼
Memory Builder
    ├── runs Input Processor on each video → VideoContext
    ├── extracts patterns: hooks, pacing, vocab, endings
    └── writes to NicheStore[niche]
```

### Key Data Contracts

1. **VideoContext → Prompt Builder:** The quality of the script depends entirely on what VideoContext captures. Moments need timestamps and intensity scores, not just descriptions.
2. **NichePatterns → Prompt Builder:** Patterns are injected as structured data. The Prompt Builder knows how to serialize them into the prompt format — NichePatterns does not contain raw prompt text.
3. **LLM response → Script Validator:** LLM must return structured JSON (enforced via output schema in prompt). Validator catches structural failures before they reach the caller.

## Suggested Build Order

Build order is determined by data dependencies — each layer requires its output types to be defined before the next layer can be implemented:

1. **Define core types** (`VideoContext`, `NichePatterns`, `TimedScript`) — everything depends on these contracts
2. **LLM Client + Claude adapter** — enables end-to-end testing early; other components need something to call
3. **Prompt Builder + hardcoded context** — validates that LLM produces correct output format before wiring inputs
4. **Summary Normalizer (text path)** — fast path, no dependencies, enables full pipeline testing without video costs
5. **Output Formatter (validator + serializer)** — completes the fast path end-to-end flow
6. **Memory Builder + NicheStore** — offline tool to build initial niche memory; can run on manual examples before video analysis works
7. **Memory Reader** — wire memory into generation; now scripts use learned patterns
8. **Video Analyzer** — expensive path, build last after text path is validated; token cost needs real measurement before committing to this path

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Anthropic Claude API | Adapter in `llm/claude-adapter.ts` | Start here; SDK is `@anthropic-ai/sdk` |
| OpenAI API | Adapter in `llm/openai-adapter.ts` | Add when provider comparison is needed |
| Google Gemini API | Adapter in `llm/gemini-adapter.ts` | Best native video understanding; evaluate for video path |
| ffmpeg | Shell subprocess or `fluent-ffmpeg` npm | Frame extraction for video path |
| OpenAI Whisper (or equivalent) | HTTP API or local model | Audio transcription for video path |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Input Processor → Memory Layer | VideoContext struct | One-way; Input does not know about Memory |
| Memory Layer → Generator | NichePatterns struct | One-way; Memory does not know about LLM |
| Generator → LLM Client | LLMPrompt + GenerationParams | LLM Client is the only thing with provider knowledge |
| Generator → Output Formatter | Raw LLM JSON response | Formatter validates and reshapes, does not regenerate |
| Memory Builder → NicheStore | NichePatterns write | Memory Builder is write-only; Memory Reader is read-only |

## Anti-Patterns

### Anti-Pattern 1: Embedding Prompt Text in NicheStore

**What people do:** Store fully-formed prompt snippets like "Use excited language and start with..." in the niche memory files.

**Why it's wrong:** Prompt logic is entangled with data. When you change prompt structure or switch providers, you must update every niche's stored data. The niche store becomes a prompt template, not a data store.

**Do this instead:** Store structured data (`hookType: "question"`, `energyLevel: "high"`). The Prompt Builder translates data into prompt language. Data and prompt logic stay in separate layers.

### Anti-Pattern 2: Building Video Analysis Before Validating the Text Path

**What people do:** Start with the most technically interesting component (video vision analysis) before the rest of the pipeline works.

**Why it's wrong:** Video analysis is expensive (Gemini: ~300 tokens/second, meaning 9,000 tokens for a 30-second clip), slow to iterate on, and adds cost to every test run. If the generation or output layer has bugs, you burn API budget discovering them.

**Do this instead:** Build the full pipeline using the text summary path first. Once the pipeline produces correct output reliably, add video analysis as the premium input path.

### Anti-Pattern 3: Hardcoding the Niche in the Generator

**What people do:** Write the AI commentary prompt directly into the generator code because there's only one niche right now.

**Why it's wrong:** The project already anticipates niche expansion. Hardcoding means a rewrite when adding niche 2. The `niche` parameter threading through the system is cheap to add now and expensive to retrofit.

**Do this instead:** Pass `niche: string` as a parameter from the entry point through Input Processor → Memory Reader → Prompt Builder. NicheStore is always keyed by niche ID.

### Anti-Pattern 4: Calling LLM Directly from Multiple Components

**What people do:** Import the Anthropic SDK in both the generator and the memory builder (for pattern extraction).

**Why it's wrong:** LLM calls are scattered. When you switch providers or add cost tracking, you must update every call site. Provider knowledge leaks outside the LLM layer.

**Do this instead:** All LLM calls go through `LLMClient`. Memory Builder uses the same client interface. The factory resolves the provider once at startup.

## Scaling Considerations

This is a single-user CLI tool initially. Scaling is not a primary concern, but design decisions now affect extensibility later.

| Scale | Architecture Adjustments |
|-------|--------------------------|
| Single user, CLI | Current design is correct — no server, no queue needed |
| Multi-user web app | Add API server layer above current components; components don't change |
| High-volume generation | Add request queue + async processing; LLM Client becomes async pool |
| Multiple niches, large memory | Migrate NicheStore from files to SQLite or vector DB (interface stays the same) |

### Scaling Priorities

1. **First bottleneck:** LLM API rate limits. The LLM Client layer is the right place to add retry logic, backoff, and fallback provider routing — all within the existing interface.
2. **Second bottleneck:** NicheStore file I/O at high pattern count. The `NicheStore` interface makes swapping to SQLite/Qdrant a single-file change.

## Sources

- [Multi-provider LLM orchestration in production: A 2026 Guide](https://dev.to/ash_dubai/multi-provider-llm-orchestration-in-production-a-2026-guide-1g10)
- [LiteLLM - Getting Started](https://docs.litellm.ai/docs/) — provider abstraction reference implementation
- [Interoperability Patterns to Abstract Large Language Model Providers](https://brics-econ.org/interoperability-patterns-to-abstract-large-language-model-providers)
- [Design Patterns for Long-Term Memory in LLM-Powered Architectures](https://serokell.io/blog/design-patterns-for-long-term-memory-in-llm-powered-architectures)
- [Video understanding - Gemini API](https://ai.google.dev/gemini-api/docs/video-understanding) — token cost data (300 tokens/sec default, 100 tokens/sec low-res)
- [10 Viral Hook Templates for 1M+ Views (2026 Guide)](https://virvid.ai/blog/ai-shorts-script-hook-ultimate-guide-2026) — script structure: Hook (0-3s) / Problem (3-15s) / Solution (15-45s) / CTA
- [Prompt Engineering and LLMs with Langchain - Pinecone](https://www.pinecone.io/learn/series/langchain/langchain-prompt-templates/) — few-shot template patterns
- [LLM Abstraction Layer - continuedev/continue DeepWiki](https://deepwiki.com/continuedev/continue/4.1-extension-architecture) — real-world provider abstraction implementation

---
*Architecture research for: AI script generation, short-form video, YouTube Shorts*
*Researched: 2026-03-26*

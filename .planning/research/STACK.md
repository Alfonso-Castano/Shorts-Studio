# Stack Research

**Domain:** AI-powered creative script generation (YouTube Shorts / short-form video)
**Researched:** 2026-03-26
**Confidence:** HIGH (LLM stack, runtime) | MEDIUM (video analysis cost) | HIGH (pattern storage)

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| TypeScript | 5.x | Primary language | Type safety for LLM I/O boundaries; catches schema mismatches at compile time; industry standard for production Node.js tooling |
| Node.js | 22 LTS | Runtime | Sufficient for LLM API orchestration; first-class TypeScript support; no async penalty for API-call-heavy workloads; avoids Python env management overhead |
| `@anthropic-ai/sdk` | 0.80.x | Claude API client | Official SDK; type-safe message API; built-in streaming support; easy model swap via `model` parameter; prompt caching built in |
| Claude Sonnet 4.6 (`claude-sonnet-4-6`) | — | Script generation LLM | Best speed/intelligence balance at $3/$15 per MTok; 64k max output; 1M context; training data through Jan 2026. Use Haiku 4.5 for cheap pattern extraction passes |
| `@google/genai` | 1.46.x | Gemini video analysis | Only viable JS SDK for native video understanding; 263 tokens/sec at default resolution; 15s clip ≈ 3,945 tokens ≈ $0.002 with Gemini 2.5 Flash — token-viable |
| Zod | 3.x | LLM output validation | TypeScript-first runtime schema validation; enforce structured script output shape; throw early on malformed LLM responses; widely supported |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `fluent-ffmpeg` | 2.x | Local video preprocessing | Extract key frames or audio from TikTok clips before sending to vision API; reduces token spend by sampling rather than sending raw video |
| `ffmpeg-static` | 5.x | Bundles ffmpeg binary | Zero-install ffmpeg for the developer; pairs with fluent-ffmpeg; remove if target machine has system ffmpeg |
| `dotenv` | 16.x | Environment config | Load `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, per-niche paths; keep secrets out of source |
| `better-sqlite3` | 11.x | Pattern/memory store | Native Node.js SQLite binding; zero external service; stores hook structures, vocabulary patterns, pacing data per niche; WAL mode for concurrent reads |
| `commander` | 12.x | CLI interface | Lightweight arg parsing for `--niche`, `--input-video`, `--summary`, `--output`; avoids yargs bloat for a single-component CLI tool |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `tsx` | Run TypeScript directly without compile step | Use for dev: `tsx src/index.ts`; faster iteration than `tsc && node` |
| `vitest` | Unit testing | Matches Jest API; native ESM; fast for testing prompt assembly and schema validation logic |
| `eslint` + `@typescript-eslint` | Linting | Catch unsafe `any` types at LLM boundaries before they hide bugs |
| `prettier` | Code formatting | One less thing to configure |

---

## Installation

```bash
# Core
npm install @anthropic-ai/sdk @google/genai zod better-sqlite3 fluent-ffmpeg ffmpeg-static dotenv commander

# Dev dependencies
npm install -D typescript tsx vitest eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier @types/better-sqlite3 @types/fluent-ffmpeg @types/node
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|------------------------|
| Claude Sonnet 4.6 (default LLM) | GPT-4o (`gpt-4o`) via OpenAI SDK | If user wants to A/B test creativity; OpenAI SDK for Node.js is equally mature. Architecture must allow swapping — use a thin `LLMClient` interface |
| Gemini 2.5 Flash for video analysis | GPT-4o Vision (frame-by-frame) | GPT-4o doesn't accept video natively — requires manual frame extraction + image arrays, adding pipeline complexity. Gemini accepts raw video files directly via File API |
| Gemini 2.5 Flash for video analysis | Claude vision (image frames) | Claude has no native video input; would require same frame-extraction hack as GPT-4o; adds pipeline steps; Gemini is purpose-built for this |
| `better-sqlite3` (pattern storage) | ChromaDB / Pinecone | Overkill for early stage: pattern memory is structured data (hook templates, vocabulary lists, pacing rules), NOT similarity search over embeddings. Add a vector DB only when doing semantic retrieval over hundreds of training examples |
| `better-sqlite3` (pattern storage) | Plain JSON files | JSON works until two concurrent processes write simultaneously; SQLite WAL mode handles this cleanly; Node.js 22 ships `node:sqlite` experimentally but `better-sqlite3` has a stable synchronous API today |
| Node.js / TypeScript | Python | Python wins for model training and data pipelines; this tool does neither. Pure API orchestration with typed schemas favors TypeScript's DX. Avoids managing virtual environments (`venv`, `pip`, `poetry`) for a CLI tool |
| `fluent-ffmpeg` (local preprocessing) | Sending raw video to Gemini inline | Inline video under 20MB is supported, but strategic frame sampling with ffmpeg reduces token spend significantly. A 30s clip at 263 tok/sec = 7,890 tokens vs. 5 sampled frames ≈ 1,290 tokens |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| LangChain / LlamaIndex | Adds 3-5 layers of abstraction for a single LLM call; obscures what prompt is actually sent; breaks frequently across versions; unnecessary for a focused script generation tool | Direct `@anthropic-ai/sdk` calls with a thin wrapper |
| OpenAI SDK for video analysis | OpenAI has no native video input as of March 2026; GPT-4.1-mini does not accept video; frame extraction pipeline needed | `@google/genai` with Gemini 2.5 Flash |
| Claude Opus 4.6 for every call | 5x the cost of Sonnet 4.6; speed is slower; the creative script task does not require Opus-tier reasoning | Claude Sonnet 4.6 for generation; Haiku 4.5 for pattern extraction and classification |
| `@anthropic-ai/sdk` version pinned below 0.70 | Older versions predate the 4.x model family API IDs and extended thinking support | Always use latest `^0.80.0` |
| Vercel AI SDK (`ai` package) | Adds abstraction appropriate for web-streaming frontends; this is a backend CLI — that abstraction is net overhead | Direct Anthropic SDK |
| ChromaDB for pattern storage at MVP | Requires Python runtime or Docker sidecar; Python client only; adds infra complexity before any semantic retrieval is needed | `better-sqlite3` until retrieval requirements demand embeddings |

---

## Stack Patterns by Variant

**If video input is provided (TikTok .mp4):**
- Use `fluent-ffmpeg` to extract 3-5 key frames at even intervals
- Send frames as images to Claude vision OR send the raw video file to Gemini 2.5 Flash
- Gemini path is lower-friction (single API call), Claude path gives you one SDK for everything
- Start with Gemini for video analysis, Claude for script generation

**If text summary is provided (no video file):**
- Skip video analysis entirely
- Send summary directly in the Claude prompt as context
- This path is always faster and cheaper — preserve it as a first-class input mode

**If pattern memory grows beyond ~500 training examples:**
- Migrate from structured SQLite rows to embeddings in a vector store
- Add `openai` text-embedding-3-small or Voyage AI embeddings for semantic retrieval
- Do NOT build this before it is needed

**If LLM provider needs to be swapped (experimentation):**
- Wrap all generation calls in a `ScriptGenerator` class with a standard interface
- Constructor accepts model name as a string; swap `@anthropic-ai/sdk` for `openai` SDK or `@google/genai` as needed
- Zod schemas for output validation remain provider-agnostic

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| `@anthropic-ai/sdk@^0.80.0` | Node.js 18+ | Requires `fetch` global (native in Node.js 18+); do not polyfill |
| `@google/genai@^1.46.0` | Node.js 18+ | Official Google Gen AI SDK (replaces deprecated `@google/generative-ai`); check npm for package name — old package is no longer updated |
| `better-sqlite3@^11.0.0` | Node.js 16+ | Native addon; requires build tools (`node-gyp`); on Windows: Visual Studio Build Tools or `npm install --global windows-build-tools` |
| `fluent-ffmpeg@^2.1.0` | `ffmpeg-static@^5.0.0` | `ffmpeg-static` sets `process.env.FFMPEG_PATH`; fluent-ffmpeg reads it automatically; no manual path config needed |
| TypeScript 5.x | Node.js 22 LTS | Use `"moduleResolution": "bundler"` or `"NodeNext"` in tsconfig; `"type": "module"` in package.json for ESM |

---

## Cost Model (Reference)

For a single script generation run with video input:

| Step | Tokens (approx.) | Cost (approx.) |
|------|-----------------|----------------|
| Video analysis (15s clip, Gemini 2.5 Flash) | ~3,945 input | ~$0.002 |
| Script generation prompt + patterns (Claude Sonnet 4.6) | ~2,000 input / ~800 output | ~$0.018 |
| **Total per script** | | **~$0.02** |

Pattern extraction during training (Haiku 4.5, batch mode):
- ~50% batch discount; Haiku at $1/$5 per MTok = pennies per video analyzed

Prompt caching on repeated system prompts (niche context) saves ~90% on cached tokens.

---

## Sources

- [Anthropic Models Overview](https://platform.claude.com/docs/en/about-claude/models/overview) — verified model IDs, pricing, context windows (HIGH confidence)
- [Gemini Video Understanding Docs](https://ai.google.dev/gemini-api/docs/video-understanding) — token rate (263/sec default, 100/sec low), inline vs File API threshold (HIGH confidence)
- [@anthropic-ai/sdk on npm](https://www.npmjs.com/package/@anthropic-ai/sdk) — version 0.80.0 current as of March 2026 (HIGH confidence)
- [@google/genai on npm](https://www.npmjs.com/package/@google/genai) — version 1.46.x confirmed (HIGH confidence)
- [AI API Pricing Comparison 2026 — IntuitionLabs](https://intuitionlabs.ai/articles/ai-api-pricing-comparison-grok-gemini-openai-claude) — cross-provider pricing validation (MEDIUM confidence)
- [Node.js vs Python for AI-First Backends 2026 — GroovyWeb](https://www.groovyweb.co/blog/nodejs-vs-python-backend-comparison-2026) — runtime decision rationale (MEDIUM confidence)
- [SQLite vs Chroma for Vector Embeddings — stephencollins.tech](https://stephencollins.tech/posts/sqlite-vs-chroma-comparative-analysis) — storage pattern comparison (MEDIUM confidence)

---

*Stack research for: Shorts Studio — AI Script Generation Component*
*Researched: 2026-03-26*

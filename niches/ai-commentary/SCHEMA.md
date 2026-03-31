# Training Script Schema: ai-commentary

**Purpose:** Defines the exact format for training script files in `scripts/`. The Analyzer (Phase 2) writes files in this format. The Generator (Phase 3) reads them.
**Location:** `niches/ai-commentary/scripts/{NNN}-{hook-type-slug}.md`

---

## YAML Frontmatter

All five fields are required. Use native YAML types — not strings that contain numbers.

| Field | Type | Valid Values | Example |
|-------|------|--------------|---------|
| `title` | string | Any — the script's topic or headline | `"AI Just Passed the Bar Exam"` |
| `views` | integer | Whole number, no commas or abbreviations | `2400000` |
| `quality_tier` | integer | `1` (>1M views), `2` (100K–1M), `3` (<100K) | `1` |
| `hook_type` | string | kebab-case slug describing the hook technique | `"shocking-stat"` |
| `date_analyzed` | string | ISO date YYYY-MM-DD | `"2026-03-31"` |

**Do NOT add extra fields.** The `source` field was explicitly removed — all scripts are YouTube Shorts.

---

## Body Structure

### ## Transcript

Label each section. No blank lines between the label and the first line of content.

```
[HOOK]
{hook text — typically 1–2 sentences, ~2–3 seconds when spoken}

[BUILD]
{build text — typically 3–6 sentences, ~15–20 seconds when spoken}

[PAYOFF]
{payoff text — typically 1–2 sentences, ~3–5 seconds when spoken}
```

### ## Analysis

Six fields. Each capped at 2–3 sentences for token efficiency. Use bold field names.

| Field | What it captures |
|-------|-----------------|
| `writing_style` | Tone, register, energy level |
| `hook_patterns` | Specific technique used in the hook |
| `pacing` | How energy and timing flow through the script |
| `sentence_structure` | Length, rhythm, question vs statement ratio |
| `storytelling_technique` | Setup/tension/reveal, journey, contrast, etc. |
| `why_it_works` | Overall summary of what makes this script perform |

---

## Complete Example

```markdown
---
title: "AI Just Passed the Bar Exam"
views: 2400000
quality_tier: 1
hook_type: shocking-stat
date_analyzed: 2026-03-31
---

## Transcript

[HOOK]
This AI just passed the bar exam. Who's getting fired first?

[BUILD]
GPT-4 scored in the 90th percentile — better than most human lawyers who took it last year.
But here's what nobody's talking about: it didn't just memorize case law.
It reasoned through hypotheticals. The kind law school tests you on.

[PAYOFF]
Your lawyer charges $400 an hour to do what a $20 API call just did better.

## Analysis

**writing_style:** Clipped, declarative sentences. No hedging. Tone is confrontational without being hostile — forces the viewer to pick a side.

**hook_patterns:** Shocking stat delivered as a bald statement, immediately followed by a direct-address implication question. No wind-up, no context-setting.

**pacing:** Opens at maximum tension, sustains through the BUILD with escalating specificity ("90th percentile," "hypotheticals"), lands the PAYOFF as a dollar-amount gut punch.

**sentence_structure:** 70% short declaratives (under 10 words), 30% medium compound. No questions except in the hook. No filler words.

**storytelling_technique:** Contrast — everyone knows AI is smart, but scoring above most lawyers reframes the stakes. The payoff flips from abstract ("AI is impressive") to personal ("your money").

**why_it_works:** The hook assumes the viewer's conclusion before they reach it. The build makes the assumption feel inevitable. The payoff makes it personal.
```

---

## Common Mistakes to Avoid

- `views: "2.4M"` — wrong. Use `views: 2400000` (integer, no quotes)
- `quality_tier: "Tier 1"` — wrong. Use `quality_tier: 1` (integer)
- Adding a `source:` field — explicitly removed. Do not add it.
- Analysis fields longer than 2–3 sentences — hurts token efficiency at generation time

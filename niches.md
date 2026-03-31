# Niche Registry

**Purpose:** Lists all active niches. To add a new niche, add a row to the table and create the corresponding directory under `niches/`.

| Niche | Directory | Status | Description |
|-------|-----------|--------|-------------|
| ai-commentary | `niches/ai-commentary/` | Active | AI news commentary for YouTube Shorts |

## Adding a New Niche

1. Add a row to the table above
2. Create the directory: `niches/{niche-slug}/`
3. Create subdirectory: `niches/{niche-slug}/scripts/`
4. Copy `anti-patterns.md` and `feedback.md` from an existing niche and update the headers
5. Create placeholder `patterns.md` and `system-prompt.md` (see existing niche for format)
6. Add a `SCHEMA.md` from an existing niche — script format is the same across niches
7. Run the Analyzer to begin ingesting training scripts

## File Naming Convention for Training Scripts

Files in `scripts/` follow this pattern: `{NNN}-{hook-type-slug}.md`

- `NNN` is a zero-padded sequential number (001, 002, 003...)
- `hook-type-slug` is a kebab-case description of the defining hook technique

Examples: `001-shocking-stat.md`, `002-question-hook.md`, `003-viral-claim.md`

The number ensures predictable sort order. The slug describes the hook at a glance.

CONTEXT.md — paste this into your project root
# Shorts Studio — Claude Code Context Briefing

## What this project is
A tool to help create high-engagement YouTube Shorts in the AI commentary niche.
The end goal is a multi-niche app, but we are building one part at a time.
The first part to build is the script writer + AI trainer.

## The YouTube Shorts workflow (full picture)
1. Find viral TikTok content (human)
2. Download video + upload to CapCut (human)
3. Generate script from raw video — THIS IS WHAT WE ARE BUILDING FIRST
4. Run script through ElevenLabs TTS (manual for now)
5. Upload audio to CapCut (manual for now)
6. Edit video in CapCut to match script (human)
7. Download + schedule post at 4–7 PM ET (human)

## What the script writer must do
- Accept a raw video file as input
- Analyze the video to identify key engaging moments
- Generate a timed commentary script that matches the video visually
- Output: script with timestamps + a separate edit brief
- Script segments must match visual events in the video
- Captions must be written in 1–3 word chunks (for CapCut caption display)
- Tone: mix of hype and conversational
- Hook must land in the first 2–3 seconds

## Niche memory architecture (important)
- Each niche has its own isolated memory (system prompt + learned patterns)
- Current niche: ai-commentary
- Memory is built by analyzing uploaded training videos
- Memory stores: hook structures, pacing patterns, high-performing vocabulary
- Future niches will have separate memory — they must never share context

## Source video specs
- Platform: TikTok
- Length: 15–30 seconds typical
- User provides: raw video + optional timestamp notes

## What to build right now (Phase 1 only)
1. Niche folder structure: niches/ai-commentary/
2. A memory file: niches/ai-commentary/memory.json (starts empty)
3. A system prompt file: niches/ai-commentary/system-prompt.md
4. A memory builder: reads training videos, extracts patterns, writes to memory.json
5. A script writer: takes raw video + memory, outputs timed script + edit brief

## What NOT to build yet
- ElevenLabs integration
- Any UI
- Orchestrator / main.js
- Any niche other than ai-commentary

## How the user works with Claude Code
- Not a developer — Claude Code handles all code
- VS Code + Node.js installed
- Step-by-step, one part at a time
- Ask before assuming; confirm before building
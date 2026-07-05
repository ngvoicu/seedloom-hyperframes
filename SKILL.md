---
name: seedloom-hyperframes
description: Produce complete AI-generated videos by combining Seedloom (Seedance video clips, Seed TTS narration, Seedream images — local files via the seedloom CLI) with HyperFrames (deterministic HTML compositions - captions, typography, timing, rendering). Use when the user wants to make a video, explainer, promo, or narrative piece with AI-generated footage or narration, wire generated clips/voiceover/stills into a HyperFrames composition, chain Seedance shots for continuity, or QA generated clips before integration. Provider-agnostic for images (Seedream, GPT Image 2 via ConsensFlow or Codex) and for image-to-video (Seedance scriptable; Flow/Veo manual handoff).
---

# Seedloom × HyperFrames — AI video production

The pipeline: **generate media as local files** (Seedloom or another provider) → **compose deterministically** (HyperFrames owns timing, captions, readable text, rendering). Each tool has its own skill for depth (`seedloom`, `hyperframes`); this skill is the glue — the workflow, the contracts, and the judgment calls.

## Prerequisites

- HyperFrames: `npx hyperframes doctor` (see the `hyperframes` skills for install).
- Seedloom: `npx skills add ngvoicu/seedloom -g` — the CLI zero-installs via `npx -y github:ngvoicu/seedloom`. Keys: `ARK_API_KEY` (video/images/QA), `BYTEPLUS_VOICE_API_KEY` (TTS) — `seedloom doctor` explains both; each unlocks its own family independently.

## The file contract (why this pairing needs no adapter)

| Seedloom output | HyperFrames input |
|---|---|
| `clip.mp4` | `<video>` layer — direct child of the host root; the framework owns playback |
| `narration.mp3\|wav` | the voiceover track |
| word timings — `seedloom tts --words` writes `narration.words.json` natively on TTS 1.0/ICL voices; otherwise `npx hyperframes transcribe narration.wav` | captions / karaoke — flat `[{id,text,start,end}]` |
| `image.png` | stills, title cards, background plates |
| `last_frame.png` (from `--last-frame`) | the `--image` of the NEXT clip — visual continuity across shots |

Every seedloom run writes its artifacts + `result.json` under `./seedloom-runs/<id>/` — copy accepted assets into the project's `assets/` tree; never reference a generator's run dir from a composition.

## The pipeline

1. **Script first.** Write the narration + beat plan. Nothing is generated before the words exist.
2. **Narration.** `seedloom tts "<text>" --voice <id>` (Seed voices, `--tone "warm, reassuring"` for delivery) — or HyperFrames' own TTS (`npx hyperframes tts`) if no voice key. Then word timings: `--words` provides them natively on TTS 1.0/ICL voices (`narration.words.json`); otherwise `npx hyperframes transcribe`. **All picture timing derives from the real audio durations** — never stretch audio to fit picture.
3. **Stills.** Generate title cards, plates, and first frames (see "Choosing an image generator").
4. **Motion.** Build the full composition with stills + typography first (the no-i2v cut). Then upgrade the shots that earn it: `seedloom video "<shot prompt>" --image first.png --dur 5 --model fast` for drafts. Chain shots with `--last-frame` → next shot's `--image`.
5. **QA before integrating.** `seedloom qa clip.mp4 "<the prompt>"` — seed-1.8 watches the clip and reports artifacts/mismatches for cents, before you spend human review. Reject clips that are worse than their still fallback; log why.
6. **Compose.** Wire accepted files into the HyperFrames composition; captions and all readable text as HyperFrames layers.
7. **Render & verify.** `npx hyperframes render` → check duration/fps/audio with `ffprobe`, scan a contact sheet, check caption-heavy frames.

## Choosing an image generator

Multiple backends produce equivalent stills. Resolution order:

1. **Seedream via `seedloom image`** — when `ARK_API_KEY` is set. Same billing as Seedance; keeps one provider aesthetic across stills and video.
2. **GPT Image 2 via ConsensFlow** — when ConsensFlow is installed with an image participant (e.g. `@pygmalion`); interactive one-offs.
3. **GPT Image 2 via `codex exec`** — when the Codex CLI is authenticated; non-interactive batches.

If more than one is configured, **ask the user once per project which to standardize on** (style consistency beats convenience) and record the choice in the project docs. If none is configured, say what each needs — don't silently fall back to describing images in text.

Prompting stills, regardless of backend: state style first, then composition/lighting/mood; 16:9 (or the project ratio); and **never readable text, logos, or UI in generated images** — those are HyperFrames layers.

## Division of labor (the rule that keeps quality)

**Generated media** carries mood, environments, characters, motion. **HyperFrames** carries everything the viewer must read or trust: captions, titles, commands, diagrams, dashboards, metrics, timelines. If a beat requires the audience to read exact text or follow cause-and-effect, end the generated clip before that beat and build the reveal programmatically.

- One caption system per video — same font, geometry, offsets everywhere; never baked into source media.
- i2v clips are an **upgrade layer** over stills, not the default. Keep the still fallback until the clip proves better.
- Strip or mute generated-clip audio on integration; the narration track and HyperFrames own sound (Seedance's `--no-audio` at generation time also works).
- Archive every raw generation with the prompt that made it; version renders, never overwrite an accepted cut.

## Seedance-specific judgment

- **No real human faces** in reference media on the international endpoint (moderation rejects them). Stylized/generated characters are fine; photo-real people are not — for those, use a manual i2v provider (below).
- Draft on `--model fast` or `mini`, finalize on `standard` (a 5s 1080p standard clip ≈ $1; drafts are cents).
- `--dur 4-15`; concurrency is ~3 tasks (1 for 4K) on individual accounts — generate serially in scripts.
- Continuity: `--last-frame` + next shot's `--image` beats trying to describe continuity in prose.

## Manual i2v providers (Flow / Veo)

When a shot needs photo-real humans or a provider Seedloom doesn't cover, hand off manually. Give the operator only: the source image(s), settings, the positive prompt, negative constraints, and the expected returned file path. On return: inspect with `ffprobe` + a contact sheet (late frames especially — drift lives there), archive the raw file, integrate muted, reapply captions programmatically. Reject on face drift, morphing, overacting, or composition jumps — a clean still with subtle programmatic motion beats a flashy broken clip.

## Minimal project layout

```
my-video/
  script.md          # narration + beats (timings filled from real audio)
  assets/            # accepted files, copied in from seedloom-runs/
  composition/       # the HyperFrames project
  renders/           # versioned outputs; finals never overwritten
```

Keep a short STATUS note (current cut, blockers, what's verified) when the production spans sessions.

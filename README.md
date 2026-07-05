# seedloom-hyperframes

[![license: MIT](https://img.shields.io/badge/license-MIT-2ea44f.svg)](LICENSE)
![type: universal skill](https://img.shields.io/badge/type-universal%20skill-63C7B2.svg)
![install](https://img.shields.io/badge/install-npx%20skills%20add-blue.svg)

**Teach your AI coding agent to produce complete videos** — generated footage, narration, word-timed captions, rendered MP4 — by pairing two tools that were made for each other:

- **[Seedloom](https://github.com/ngvoicu/seedloom)** generates the media: Seedance video clips, Seed TTS narration (500+ voices, $2 voice cloning), Seedream stills — always as plain **local files**.
- **[HyperFrames](https://hyperframes.heygen.com)** composes and renders **deterministically**: captions, typography, diagrams, timing — HTML in, MP4 out.

This skill is the glue: the pipeline, the file contracts, and the judgment calls that decide what gets *generated* and what stays *programmatic*. That division is what separates videos that look AI-made from videos that are simply made.

```text
script.md ──► seedloom tts ──► narration.mp3 ──► hyperframes transcribe ──► word timings
                                                                                │
seedloom image ──► stills / title plates ─────────────────────────────┐        │
                                                                      ▼        ▼
seedloom video --image still.png ──► clip.mp4 ──► seedloom qa ──► HyperFrames composition
        │                                          (seed-1.8         (captions, titles,
        └── last_frame.png ──► next shot's --image  reviews it)       timing, readable text)
                                                                                │
                                                                                ▼
                                                                    npx hyperframes render
```

## Install

```bash
npx skills add ngvoicu/seedloom-hyperframes -g     # all AI tools; or -a claude-code | codex | cursor
```

Companions — each needed only when the pipeline reaches its step:

```bash
npx skills add ngvoicu/seedloom -g     # media engine; its CLI zero-installs on demand
npx hyperframes doctor                 # composition/render engine (HyperFrames plugin)
```

Keys: `ARK_API_KEY` (video / images / QA) and `BYTEPLUS_VOICE_API_KEY` (TTS) are independent — set only what you use; `seedloom doctor` explains both, and narration can fall back to HyperFrames' own free local TTS with no key at all.

## What a session looks like

An agent producing a 60-second explainer, end to end. (Examples use `seedloom`; if it isn't on PATH, prefix with `npx -y github:ngvoicu/seedloom`.)

```bash
# 1. Narration first — all picture timing derives from real audio
seedloom tts "$(cat script.md)" --voice en_male_tim_uranus_bigtts --tone "clear, warm"
npx hyperframes transcribe assets/narration.mp3        # → word timings for captions

# 2. Stills: title plates and first frames (Seedream here; see provider selection below)
seedloom image "isometric server room at dusk, teal palette, cinematic, no text"

# 3. Animate the shots that earn it — draft cheap, chain for continuity
seedloom video "slow push-in over the server room, dust motes in the light" \
        --image assets/hero.png --dur 5 --model fast --last-frame
seedloom video "the racks flicker to green as systems come online" \
        --image seedloom-runs/<id>/last_frame.png --dur 5 --model fast

# 4. Let a video-understanding LLM review before you do (cents, catches artifacts)
seedloom qa seedloom-runs/<id>/clip.mp4 "slow push-in over the server room, dust motes"

# 5. Compose and ship — captions, titles, commands are HyperFrames layers, never baked in
npx hyperframes preview composition/
npx hyperframes render composition/ -o renders/v1.mp4
```

Every seedloom run leaves its artifacts + a `result.json` in `./seedloom-runs/<id>/`; accepted files get copied into the project's `assets/` tree.

## What the skill actually teaches

**The file contract** — why this pairing needs no adapter:

| Seedloom output | HyperFrames input |
|---|---|
| `clip.mp4` | a `<video>` layer (framework owns playback) |
| `narration.mp3\|wav` (+ `narration.words.json` from `--words` on TTS 1.0/ICL voices, or transcription) | voiceover track + word-timed captions `[{id,text,start,end}]` |
| `image.png` | stills, title cards, background plates |
| `last_frame.png` | the *next* clip's `--image` — visual continuity across shots |

**Provider-agnostic image generation.** Stills can come from Seedream (`seedloom image`), GPT Image 2 via a [ConsensFlow](https://consensflow.ngvoicu.dev) image participant, or GPT Image 2 via `codex exec`. The skill defines the resolution order and — when several are configured — has the agent **ask once per project** which to standardize on, because style consistency beats convenience.

**The division of labor** that keeps quality:

- Generated media carries mood, environments, characters, motion.
- HyperFrames carries everything the viewer must **read or trust**: captions, titles, commands, diagrams, dashboards, metrics.
- Image-to-video clips are an *upgrade layer* over stills — the still fallback survives until the clip proves better.
- Generated-clip audio is muted on integration; narration and sound design belong to the composition.
- Archive every raw generation with its prompt; version renders; never overwrite an accepted cut.

**Seedance judgment**: draft on `fast`/`mini` (cents), finalize on `standard` (~$1 per 5s 1080p clip); no real human faces in reference media on the international endpoint (moderation) — for photo-real people the skill routes to a **manual Flow/Veo handoff**, with the checklist for clean handoffs and integration QA.

## What it costs (order of magnitude)

A typical 60-second piece with 6–8 generated clips: **~$1–2 drafted** on fast/mini, **~$4–8 finalized** on standard, narration **~$0.04**, QA reviews **~$0.01 each**. TTS has a 20k-character free trial. (Numbers from the verified research in the [seedloom repo](https://github.com/ngvoicu/seedloom); token-based, July 2026.)

## FAQ

**Do I need both tools?** For the full pipeline, yes — but each step degrades gracefully: no voice key → HyperFrames' local TTS; no Ark key → stills from GPT Image 2 providers and a typography-driven cut with no generated video at all (often the right call for technical explainers).

**Why two API keys?** They unlock different BytePlus services (ModelArk vs Seed Speech) with separate consoles and billing — a platform property, not a Seedloom choice. Each key is optional and unlocks its own family.

**Where did the narrative-film playbook go?** This skill absorbed an earlier private production skill; the deep reference for character-driven work (identity continuity, Flow prompt surgery, speaking-shot timing) is preserved at [`docs/legacy-ai-video-production.md`](docs/legacy-ai-video-production.md).

**Why local files instead of URLs?** BytePlus URLs expire in ~24h. Seedloom downloads on success, always — and local files are exactly what HyperFrames (or any editor) consumes. The artifact is the file.

---

<!-- ngvoicu author section — identical across all ngvoicu repos, keep in sync -->
## AI-native toolkit

This project is part of a larger AI-native toolkit — and of a way of working your whole team can adopt: talks (["Becoming an AI Native Company"](https://ngvoicu.dev/becoming-an-ai-native-company/)), hands-on team training that teaches employees to use AI, and [AI adoption consulting for engineering teams](https://ngvoicu.dev/#consulting).

- Site: [ngvoicu.dev](https://ngvoicu.dev)
- Contact: [office@ngvoicu.dev](mailto:office@ngvoicu.dev) · +40 734 704 910

Toolkit: [Specmint](https://specmint.ngvoicu.dev) (durable AI coding specs) · [Kluris](https://kluris.ngvoicu.dev) (team knowledge brains) · [ConsensFlow](https://consensflow.ngvoicu.dev) (cross-agent second opinions)

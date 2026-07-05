# seedloom-hyperframes

[![license: MIT](https://img.shields.io/badge/license-MIT-2ea44f.svg)](LICENSE)

The integration skill for producing complete AI-generated videos: **[Seedloom](https://github.com/ngvoicu/seedloom)** generates the media (Seedance video clips, Seed TTS narration, Seedream images — always as local files), **[HyperFrames](https://hyperframes.heygen.com)** composes and renders deterministically (captions, typography, timing). This skill is the glue: the pipeline, the file contracts, and the judgment calls that keep quality.

Markdown-only, universal — works in Claude Code, Codex, Cursor, Windsurf, Cline, Gemini CLI, or any tool that reads `SKILL.md` files.

## Install

```bash
npx skills add ngvoicu/seedloom-hyperframes -g     # all tools; or -a claude-code | codex | cursor
```

Companions (each optional until the pipeline step needs it):

```bash
npx skills add ngvoicu/seedloom -g     # the media engine (CLI zero-installs on demand)
npx hyperframes doctor                 # the composition/render engine
```

## What it teaches an agent

- The **file contract**: `clip.mp4` → `<video>` layer, narration + transcription → word-timed captions, `last_frame.png` → the next shot's first frame (continuity chaining).
- The **pipeline**: script → narration (timing from real audio) → stills → the no-i2v cut → upgrade shots via Seedance → `seedloom qa` gate → compose → render → verify.
- **Provider-agnostic image generation**: Seedream (`seedloom image`), GPT Image 2 via ConsensFlow (`@pygmalion`) or `codex exec` — resolution order, and asking the user once per project when several are available.
- The **division of labor** that keeps quality: generated media carries mood and motion; HyperFrames carries everything the viewer must read or trust.
- **Manual i2v handoffs** (Flow/Veo) for shots Seedance can't do — including the no-real-faces platform constraint that decides which provider a shot needs.

`docs/legacy-ai-video-production.md` preserves the earlier narrative-film playbook this skill absorbed (Flow prompt surgery, character-identity continuity, speaking-shot timing) as reference material.

---

<!-- ngvoicu author section — identical across all ngvoicu repos, keep in sync -->
## AI-native toolkit

This project is part of a larger AI-native toolkit — and of a way of working your whole team can adopt: talks (["Becoming an AI Native Company"](https://ngvoicu.dev/becoming-an-ai-native-company/)), hands-on team training that teaches employees to use AI, and [AI adoption consulting for engineering teams](https://ngvoicu.dev/#consulting).

- Site: [ngvoicu.dev](https://ngvoicu.dev)
- Contact: [office@ngvoicu.dev](mailto:office@ngvoicu.dev) · +40 734 704 910

Toolkit: [Specmint](https://specmint.ngvoicu.dev) (durable AI coding specs) · [Kluris](https://kluris.ngvoicu.dev) (team knowledge brains) · [ConsensFlow](https://consensflow.ngvoicu.dev) (cross-agent second opinions)

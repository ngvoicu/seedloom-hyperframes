# seedloom-hyperframes — agent & contributor guide

Kept byte-identical in `AGENTS.md` and `CLAUDE.md`. Edit one, copy to the other.

## What this is

The **integration skill** (markdown-only, Spec Mint pattern) that teaches agents to produce complete videos from Seedloom media + HyperFrames compositions. It sits above two tool skills and never duplicates their depth: `seedloom` (the engine's own skill, same-commit-synced in its repo) and the `hyperframes` skill family (ships with the HyperFrames plugin).

## Layout

- `SKILL.md` — the canonical skill (root). The only teaching artifact.
- `skills/seedloom-hyperframes/SKILL.md` — symlink → `../../SKILL.md` for skills-dir discovery. Never replace with a copy here.
- `docs/legacy-ai-video-production.md` — the retired narrative-film playbook this skill absorbed (Flow handoff surgery, character continuity, speaking-shot timing). Reference material, not an active skill; mine it when narrative-mode guidance needs deepening.

## Rules

- **Workflow skill, not a tool skill.** Commands, flags, and API facts belong to the tool skills/repos; this skill owns the pipeline, contracts between tools, and judgment calls. When seedloom or HyperFrames change surface, update the referenced commands here in the same sitting.
- **Provider-agnostic by design.** Image generation and i2v guidance must never assume one vendor; the resolution-order + ask-the-user pattern is the contract. Nothing tailored to any one person's setup — this is a public skill.
- **No invented facts.** Costs, limits, and constraints quoted here must trace to the seedloom repo's verified research or HyperFrames docs.
- Markdown-only: no tests. Validate by reading the rendered SKILL.md and `ls -la skills/seedloom-hyperframes/` (symlink resolves).

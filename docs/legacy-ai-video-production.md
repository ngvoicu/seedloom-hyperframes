---
name: ai-video-production
description: Plan, build, revise, and QA AI-assisted narrative videos or short films with generated stills, image-to-video handoffs such as Flow/Gemini/Veo, TTS or premium voices, captions, character continuity, and render integration. Use when preparing scene prompts, first/last/source frames, voice/caption timing, Flow retry prompts, video clip integration, production bibles, asset logs, status files, or reusable workflows for future video projects.
---

# AI Video Production

## Operating Rule

Use this skill as the deep reference for narrative/cinematic production mechanics — image-to-video handoffs, character continuity, voice timing, integration QA. Keep project-specific facts in that project's docs, not inside this skill. For product/explainer videos in the ngvoicu youtubes workspace, the workspace skill (ngvoicu-video-production) is the routing layer and defers here for narrative mechanics.

For every project, first identify or create these source-of-truth files:

- Production bible: story thesis, tone, visual language, acceptance rules.
- Character bible: names, roles, physical locks, wardrobe, behavior, reference images.
- Voice bible: draft voices, premium voice IDs, speed policy, audition notes.
- Scene plan: scene purpose, beats, visual candidates, Flow candidates.
- Asset log: generated assets, source paths, rejected attempts, accepted versions.
- Project status: current render, current final copies, active blockers, verification state.

If the repo already has an `AGENTS.md`, local skill, status file, or asset log, read those before editing.

## Build Strategy

Default to a no-Flow-first edit. Prove story, voice timing, captions, and visual rhythm before waiting for image-to-video exports.

Use an image-to-video provider as an upgrade layer. Flow/Gemini/Veo are the manual-handoff paths; Seedance via the `seedloom` CLI (`seedloom video --image first.png …`) is the scriptable path — but note two constraints: seedloom's first live validation is still pending, and BytePlus's international endpoint FORBIDS real human faces in reference media, so photo-real locked-character continuity work stays on Flow/Veo; use Seedance for stylized characters, environments, and no-face B-roll.

The upgrade-layer loop:

1. Build a clean still or HyperFrames fallback.
2. Prepare source frame(s) and prompt.
3. Archive the raw returned MP4.
4. Inspect the returned clip before integrating.
5. Integrate only if it improves the cut.

Never let a flashy generated video replace a cleaner still if it causes face drift, morphs, hand artifacts, overacting, weak sync, or unclear story.

Use Flow for the parts it is good at: mood, time passing, human reaction, and subtle physical presence. Keep complex reasoning, dashboards, readable UI, incident timelines, and cause/effect explanations in HyperFrames or another programmatic layer so the viewer can read and understand them.

## Versioning

Preserve prior versions. Create a new version name for meaningful visual, audio, timing, or Flow changes.

For each meaningful render or handoff, update:

- render notes or handoff prompt
- sync map or explicit timing reference
- contact sheet or key check frames
- asset log
- project status

Do not overwrite accepted renders unless the user explicitly asks.

## Voice And Timing

Keep one movie-wide voice-speed policy.

- Generate raw narrator/dialogue at natural provider speed unless the user explicitly approves a speed experiment.
- Build captions, pauses, and visual timing from actual audio durations.
- Do not fit individual lines to old subtitle windows or Flow motion by stretching/slowing a single line.
- If a Flow clip does not sync, move picture timing, edit pauses, media start, or regenerate the Flow prompt with better timing.

Keep Flow-generated audio non-canon by default. Strip or mute returned video audio during integration unless the user explicitly approves a comparison or exception. When a specific Flow-audio exception is approved, extract that audio into the master track and keep the video element muted so the final render cannot double-play the same sound.

When using premium voices, store exact voice names, IDs, model, settings, and audition notes in the voice bible before using them across scenes.

## Captions And Text

Use one runtime/programmatic caption layer.

- Keep captions separate from generated images and Flow clips.
- Do not bake subtitles into source images.
- Do not add a global bottom/top shade bar behind every frame.
- Match caption geometry across scenes: font, weight, pill width rules, padding, bottom offset, outline, and shadow.
- Verify long captions on check frames so text does not overflow or sit off-center.

Keep all readable UI and labels programmatic. Generated images may contain blurred or abstract screens, but not production-critical readable text.

## Image Generation

Use the project-approved image generator for new stills, character references, background plates, and Flow source frames. Default generator is GPT Image 2 — via Codex CLI for non-interactive batches (below), or via the ConsensFlow `@pygmalion` participant for one-off interactive generations (same backend, same quota).

### Codex CLI For Image Generation

When Claude Code needs to generate images, use the Codex CLI (`codex exec`) installed at `/opt/homebrew/bin/codex`. This calls GPT Image 2 non-interactively:

```bash
codex exec -s danger-full-access -C <project-dir> --skip-git-repo-check \
  "Generate a GPT Image 2 image: <detailed prompt>. Save to <absolute-path>.png"
```

Key rules for Codex image generation:

- Use `codex exec` with `-s danger-full-access` and `--skip-git-repo-check` for non-interactive operation.
- Always specify the full absolute save path in the prompt.
- For multiple images, batch them in one prompt listing each with its own description and save path.
- Generated images land at `~/.codex/generated_images/<uuid>/` and are also saved to the requested path.
- After generation, copy accepted images into the project's `assets/images/` tree.
- Run Codex exec with a long timeout (image generation takes 30-120 seconds per image).
- When calling from Claude Code, use `run_in_background` and monitor for completion.

Prompt style for cinematic stills:

- State the film genre and visual style first.
- Describe composition, lighting, camera angle, and mood.
- Name specific elements: monitors, dashboards, coffee cups, cable clutter, etc.
- Specify aspect ratio (16:9) and that the image should be photorealistic.
- Exclude readable text, captions, subtitles, logos, or UI copy from generated images.

### Source Frame Rules

- Generate or choose clean 16:9 frames with no captions, no labels, no readable UI, no logos, no watermarks.
- Copy selected generated images into the project. Do not leave referenced assets only in a generator cache.
- Use locked character portraits and all-angle/reference sheets when continuity matters.
- If a scene has multiple characters, make the image prompt explicit about who speaks, who stays silent, and who is only a background witness.
- Prefer stills where body posture already encodes the intended acting, because Flow tends to amplify posture.

For character-critical Flow shots, identity continuity is the priority:

- Start from a source still generated or selected against the locked character bible and reference images, not from a loose generic prompt.
- Upload the source still plus the strongest face/all-angle references if the image-to-video tool supports references; list the reference order in the handoff.
- Repeat the character's age, face, hair, wardrobe, role, and behavior in both the source-image prompt and the Flow prompt.
- Keep one active speaking character whenever possible. Make everyone else silent, still, or absent.
- Forbid duplicate versions of the character, face/age/wardrobe changes, new glasses/beards, extra people, shot changes, and camera angle changes unless the scene explicitly needs them.
- If exact character identity matters more than motion and the Flow output drifts, keep the still/HyperFrames fallback.

For first/last-frame Flow attempts, use near-identical composition. If endpoint frames are generated separately, Flow may bridge them with a morph or shot jump. Safer patterns:

- one source frame plus short subtle motion
- a last frame edited from the exact same composition
- short non-speaking B-roll rather than dialogue replacement

## Flow Handoff Format

Every Flow handoff should include:

- source image path(s)
- optional character reference paths, ordered by priority
- scene slot absolute timing
- slot-relative dialogue timing when anyone speaks
- recommended duration and aspect ratio
- expected returned MP4 path
- audio policy
- prompt
- negative prompt / constraints
- integration notes

When the user asks what to paste into Flow, give only:

- upload image(s)
- settings
- positive prompt
- negative constraints
- expected returned MP4 path

Do not tell the user to paste repo metadata, fallback render notes, or integration notes into Flow.

## Silent Narration-Bounded Flow Clips

For non-speaking Flow shots under narrator voiceover, define the clip as picture-only and give it a clear story boundary.

Use this when a Flow clip should end on a narration beat, then hand off to a programmatic section or a tonal turn:

- State that nobody speaks and no mouth movement is needed.
- Include the narrator line range that will play over the clip.
- Name the exact final beat where the clip should end, such as "end immediately after the narrator says, 'Faster than any human team could.'"
- Keep generated audio non-canon; final integration uses the separate narrator audio and runtime captions.
- If the next beat requires readable text, dashboards, technical cause/effect, or a logical reveal, end the Flow clip before that beat and build the reveal programmatically.

Good uses:

- seasonal or time-passing montage
- quiet team reaction around a screen
- restrained relief, concern, hesitation, fatigue, or silence
- private realization by one locked character, especially when the final audio comes from the voice bible
- narrator-only B-roll with no dialogue sync requirement

Poor uses:

- precise incident dashboards
- lists of technical fixes
- root-cause diagrams
- business metric screens or customer-impact dashboards where exact labels and numbers matter
- screens where the audience must read exact text
- anything where a generated watermark, morph, or unreadable text would confuse the story

## Speaking Flow Prompts

For speaking shots, always include exact dialogue and slot-relative seconds.

Use this pattern:

```text
0.000s-0.250s: quiet, nobody speaks.
0.250s-1.700s: Maya says, "How did nobody notice this?"
1.700s-4.750s: pause, room reacts quietly.
4.750s-6.600s: Daniel says, "I didn't write it. I only reviewed it."
6.600s-8.000s: quiet aftermath.
```

Tell Flow that perfect lip sync is not required, but visible mouth movement should begin and end near those windows.

Constrain performance hard:

- keep the locked character's face, age, wardrobe, and body type consistent with the supplied references
- performance level 2 or 3 out of 10
- seated if seated in the source
- no standing up or sitting down
- no pointing, waving, hand-on-chest, open-palmed accusation, or theatrical gestures
- no shame/embarrassment unless the script requires it
- silent characters remain silent witnesses with tiny eye/breathing shifts

If Flow does not support voice/audio upload in the current workflow, do not ask the user to upload voice samples. Use text and timing only.

## Flow Integration

When the user provides a Flow MP4:

1. Inspect with `ffprobe`.
2. Create a contact sheet across the full clip, including late frames.
3. Check for face drift, hand artifacts, morphs, camera jumps, composition changes, watermark placement, and overacting.
4. Decide whether the whole clip, a sub-window, or none of it is usable.
5. Archive the raw export.
6. Create a muted/video-only copy if integrating as picture.
7. Place the clip only in its intended slot.
8. Reapply captions and readable overlays programmatically above the clip.
9. Keep master voice speed unchanged.
10. If the Flow audio is explicitly approved, insert the extracted audio into the master mix and keep the video element muted.
11. Verify final render with contact sheet, targeted caption frames, `ffprobe`, and black-frame checks.

Reject the clip if it is worse than the fallback. Log why it was rejected so the next prompt targets the failure mode.

## Common Failure Modes

When Flow overacts:

- lower performance level
- remove dramatic adjectives
- specify seated/still posture
- forbid big hand gestures
- describe tone as professional, tired, restrained, quiet

When Flow drifts identity:

- upload locked face and all-angle references if supported
- describe each named character precisely
- reduce the number of active speakers
- make secondary characters silent and still

When Flow morphs between endpoint frames:

- retry with one source frame only
- shorten the duration
- use a last frame derived from the same source composition
- forbid shot changes, crossfades, morphing, angle changes, and composition changes

When Flow sync is poor:

- include exact text and seconds in the prompt
- keep the returned audio only for inspection unless approved
- move picture timing or pauses in the edit
- do not stretch voices to match generated motion

## HyperFrames Or Programmatic Alternatives

Use HyperFrames/programmatic motion when the scene needs control more than photoreal video:

- conceptual slides
- dashboards and readable UI
- captions and typography
- diagrams
- pulse/highlight indicators
- subtle still-image life: flicker, parallax, low camera drift

Avoid decorative motion that makes approved plates worse: unnecessary zoom-outs, global sheen, random gold bars, or overlays that cover faces/captions.

## Business And Metric Scenes

When a scene contrasts metric success with business harm:

- Keep exact metrics, dashboards, complaint lists, maps, and AI status messages programmatic.
- Do not ask image/video generation to create readable business text or exact numbers.
- Prefer an existing stakeholder character for accountability dialogue instead of inventing a new speaker and voice, unless the script truly needs a new person.
- Use active-speaker visual language for meeting dialogue so the viewer always knows who is speaking.
- Use Flow only for the human emotional beat, such as one locked character privately reviewing consequences or silently realizing the metric was wrong.
- In the Flow prompt, include exact dialogue timing if the character speaks, but keep final audio from the voice bible unless a comparison/exception is approved.
- If the generated solo realization shot weakens character identity, use the still frame with subtle programmatic motion instead.

## Final Check

Before calling a scene done, verify:

- latest render path and final-copy path
- duration, resolution, fps, audio codec
- blackdetect or equivalent blank-frame check
- contact sheet visual audit
- targeted frames for captions and known risky moments
- audio levels when sound design changed
- project status and asset log updated

If a result is rejected, preserve it, name the reason, and create a better retry prompt or fallback path.

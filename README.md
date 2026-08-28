# Runware Skills

Agent Skills for [Runware](https://runware.ai) - packaged, outcome-oriented know-how that lets an AI agent get great results from Runware's models without knowing the catalog.

Each skill is a folder under `skills/` with a `SKILL.md`. An agent loads a skill on demand (by its `description`), follows the workflow, and drives the Runware API (SDK / MCP / CLI) to produce the result.

## Install

**Claude Code** - add the marketplace, then install the plugin:

```
/plugin marketplace add runware/runware-skills
/plugin install skills@runware
```

Migrating from another provider? Install [`migrate@runware`](#migration) instead - or as well.

**Any other agent** - these follow the open Agent Skills standard, so Cursor, OpenClaw, Codex, and others can use them too. Clone the repo and put the folders where that agent loads skills from (`.cursor/skills/`, `.claude/skills/`, `.agents/skills/`, and so on):

```
git clone https://github.com/runware/runware-skills.git
```

Full per-platform steps (including Claude.ai) are in the [Skills docs](https://runware.ai/docs/platform/skills).

## How they fit together

- **Foundation skills** - shared machinery every outcome skill leans on:
  - `runware-run` - the execution contract (resolve schema → upload → run → poll).
  - `runware-models` - which model for a task, looked up live so nothing goes stale.
  - `runware-prompting` - per-model-family prompt craft.
- **Outcome skills** - the customer-facing recipes (`product-photography`, `animate-image`, `voiceover`, …). Each routes to the best models of the moment and cross-links the foundation.

A skill is a stable interface over a changing catalog: when a better model ships, you update the skill's routing and every user gets the upgrade without changing anything.

## Catalog

**Foundation** (shared machinery every skill uses)
- `runware-run` - the execution contract: inspect the schema, run sync or async, read the result.
- `runware-models` - pick the right model for a task and keep the choice current.
- `runware-prompting` - per-model-family prompt craft.

**Image**
- `product-photography` - packshot to studio, lifestyle, and hero shots.
- `character-consistency` - keep a character or product identical across scenes.
- `text-in-image` - posters, packaging, and UI with legible, exact copy.
- `edit-image` - add, remove, replace, recolor, relight, inpaint, outpaint.
- `restore-and-upscale` - deblur, denoise, upscale, restore old photos.
- `photoreal-stills` - realistic, non-AI-looking imagery.
- `composite-scene` - merge product, subject, and backdrop without masking.
- `logos-and-vectors` - true SVG logos, icons, scalable art.
- `controlled-generation` - pose, edge, or depth guided generation (ControlNet).
- `game-assets-2d` - sprites, stickers, consistent asset sets.

**Video**
- `product-ad-video` - product image to ad clip.
- `ugc-ad` - creator-style demo, testimonial, and talking-head ads.
- `animate-image` - turn a still into motion.
- `talking-avatar` - drive a talking head from a script or audio.
- `multi-shot-video` - a multi-shot story in one call.
- `cinematic-video` - deliberate shot language and color grade.
- `edit-video` - transform/restyle or surgically edit footage.
- `replace-in-video` - swap a character, product, or wardrobe in a clip.
- `reframe-video` - change aspect ratio without cropping.
- `add-audio-to-video` - sound effects, narration, or a music bed.
- `video-upscale` - increase resolution toward 4K.

**Audio**
- `voiceover` - TTS narration with emotion and voice control.
- `dialogue-audio` - multi-speaker conversation in one file.
- `voice-cloning` - clone a voice from a sample or design one.
- `music` - songs, instrumental beds, jingles.

**3D**
- `image-to-3d-asset` - photo or prompt to a textured GLB mesh.

**Text**
- `llm-agent` - chat and tool-calling over your own functions.
- `vision-understanding` - image, video, and document understanding and captioning.

**Custom models**
- `train-style-model` - fine-tune a reusable brand/style LoRA.
- `bring-your-own-model` - upload and use your own LoRA or checkpoint.

## Migration

A separate plugin under `migrate/`, for developers arriving at Runware rather than already building on it. Paste a call from your current provider and the skill reports what maps cleanly, what needs adjustment (with a concrete before/after), and what has no equivalent at all. They are compatibility checkers, not auto-translators.

```
/plugin install migrate@runware
```

- `integrate-with-runware` - the API contract for a new integration: request/response shape, auth, delivery methods, errors.
- `fal-to-runware` - port a [Fal.ai](https://runware.ai/fal-alternative) call.
- `replicate-to-runware` - port a [Replicate](https://runware.ai/replicate-alternative) call.
- `wavespeed-to-runware` - port a [WaveSpeed](https://runware.ai/wavespeed-alternative) call.

Installed separately from the main catalog, because the audiences don't overlap: the generation skills are for people already on Runware, and a migration skill has served its purpose the day after you finish migrating.

## Authoring

Every skill is a folder - `skills/<kebab-name>/SKILL.md` for the generation catalog, `migrate/<kebab-name>/SKILL.md` for the migration plugin - and each `SKILL.md` follows the same skeleton. Depth that only matters some of the time goes in an optional `references/` folder next to it, so it loads on demand rather than every time the skill triggers.

Skills route to models by AIR and defer to the live schema rather than restating it, which is what keeps them from going stale as the catalog changes.

---
name: product-photography
description: >
  Turn a plain product shot or packshot into clean studio, lifestyle, or hero imagery for
  e-commerce and ads. Use when the user says "make this a product photo", "studio shot of my
  product", "put it on a white background", "lifestyle shot for my store", "hero image for the
  ad", "relight this packshot", or wants catalog-ready product images. Often a pipeline: generate
  or clean, cut the background, then relight. For photoreal images of people, food, or scenes with
  no product focus, use photoreal-stills. To merge a product into an existing scene photo, use
  composite-scene.
---

# Product photography

Produce catalog-, ad-, or hero-ready images of a product from a plain shot or a text brief. The work is usually a short pipeline: generate or clean the subject, isolate it from its background, then place and relight it. Each stage is a different model, so route by sub-task rather than reaching for one model to do everything.

## Inputs to collect

- **The product image,** if the user has one (a packshot, phone snap, or existing render). Skip this for a from-scratch brief.
- **Target look:** studio (clean seamless or white sweep), lifestyle (in a real setting), or hero (dramatic, ad-grade).
- **Background outcome:** keep as generated, cut to transparent or pure white, or replace with a new scene.
- **Brand constraints:** exact background color, a palette to hold to, aspect ratio, output count.
- **Honesty guardrails:** no invented logos, awards, claims, or on-pack copy. Only render text the user supplied verbatim.

## Models

Confirm each pick is `live` and inspect its schema via `runware-models` + `runware-run` before calling. Never hardcode a stale choice.

- **Generate / restyle the shot (text-to-image flagships):**
  - **FLUX.2 [max]** (`bfl:7@1`) for premium photoreal hero work.
  - **Qwen-Image-2.0-Pro** (`alibaba:qwen-image@2.0-pro`) as a strong alternative, good with detail and text.
  - **Seedream** (latest live flagship, e.g. `bytedance:seedream@4.5`) for fast, attractive drafts. Confirm the current version via `runware-models`.
  - **Recraft V4.1** (`recraft:v4.1@0`) when you need locked brand color or flat, predictable mockups (see Technique).
- **Remove or replace the background:**
  - **BiRefNet** (`runware:112@5` General, `runware:112@9` Matting, `runware:112@10` Portrait) for clean cutouts. Pick the variant by subject.
  - **Bria RMBG v2.0** (`bria:2@1`) as an alternative matte.
- **Relight / recolor / recompose:**
  - **Bria FIBO Edit** (`bria:21@1`) for instruction-driven relight, recolor, generative fill, and background replacement that preserves the product's structure.

## Workflow

1. Resolve each model's schema (`runware-run`) and confirm the exact taskType and field names before calling. For background removal the taskType differs by model, so verify it against the live schema rather than assuming.
2. **Stage 1, get a clean subject.** If the user has a usable product image, keep it. Otherwise generate one with a flagship via `imageInference`, briefing it like a photographer (lens, light, surface). Run synchronously.
3. **Stage 2, isolate.** Send the subject to BiRefNet or Bria RMBG to cut a clean alpha matte, or to FIBO Edit if you want to replace the background in place.
4. **Stage 3, place and relight.** Composite onto the target background (or generate it), then use FIBO Edit to match the lighting and shadow to the new scene so the product does not look pasted in.
5. Review against the Quality bar. Retry the weak stage only, not the whole pipeline.

Skip stages you do not need. A clean studio shot on white may be Stage 1 plus a cutout. A lifestyle shot may be all three.

## Technique

Grounded in the Recraft V4.1 prompting and color-palette guides.

- **Brief it like a photographer.** Name the lens, the lighting setup, and the surface, in that order of importance: subject, environment, framing, lighting, camera, mood. A product shot often needs only subject, surface, lighting, and camera angle. Example shape: "wireless earbuds in a matte black case on raw concrete, single soft studio light from upper left, top-down 45-degree angle, minimal background, premium tech aesthetic."
- **Lighting is the lever.** "Soft studio light from the upper left" reads completely differently from "harsh midday sun" or "single tungsten bulb above." State direction and quality explicitly. This is what separates a hero shot from a flat packshot.
- **Lens references shape depth.** Adding "shot at 85mm f/1.8" or "wide 24mm" changes the spatial feel and depth of field, and these flagships handle it accurately.
- **Short vs structured.** For exploration, a short prompt lets the model make tasteful lighting and composition choices. For a locked composition, layer the full brief. Recraft V4.1 in particular reads short prompts well and follows long structured ones with high fidelity.
- **Lock brand color with Recraft.** On the Recraft family, `settings.colors` (array of `{ "rgb": [r, g, b] }`, 0-255) constrains the whole palette to your brand tones, and `settings.backgroundColor` (single `{ "rgb": [...] }`) sets the canvas behind the subject. Pass the exact RGB from brand guidelines, do not approximate. Use 2-4 colors for a tight constraint, more for a looser suggestion. Background color works best on simple, isolated-subject compositions, which is exactly the product-shot case.
- **Flat, predictable mockups.** When you need a front-facing, evenly lit catalog asset rather than a creative interpretation, reach for a Recraft Utility variant and pair it with `settings.colors`.
- **Relight instead of regenerating.** When the product itself is right but the light or background is wrong, edit with FIBO Edit rather than re-rolling a generation. It preserves the product's structure while changing lighting and surroundings, which keeps the product on-brand.
- **Match the shadow.** After compositing a cutout onto a new background, the giveaway is a missing or wrong-direction contact shadow. Use the relight step to ground the product with a shadow consistent with the new scene's light direction.

**Build order.** Fill this in before writing the prompt. Drop a line when it does not matter for the shot.

- **Product invariant:** what must stay exactly as-is (shape, materials, colors, any supplied on-pack text).
- **Surface / set:** what it sits on or in (seamless sweep, concrete, wooden bench, real room).
- **Lighting:** direction and quality (soft softbox from upper left, warm side sun, single tungsten above).
- **Lens / angle:** focal length and viewpoint (85mm, top-down 45-degree, eye level).
- **Background:** keep, cut to transparent or white, or replace with a named scene.
- **Brand palette:** exact RGB colors and canvas color, Recraft only, from brand guidelines.

For two or three worked end-to-end recipes (clean packshot, lifestyle relight, palette-locked brand shot) with the real requests and result shapes, load `references/examples.md`.

## Parameters that matter

- `positivePrompt`, carries the studio direction (lens, light, surface). Quote any on-pack text exactly, and only when the user supplied it.
- `width` / `height`, set to the target aspect (square for catalog grids, vertical or wide for ads). Confirm allowed dimensions against the live schema.
- `settings.colors` / `settings.backgroundColor`, Recraft family only, for brand palette and canvas color. RGB integers 0-255. Confirm the exact field path against the live schema before relying on it.
- Background-removal and FIBO Edit input fields (source image, mask, instruction): names vary by model. Mirror the live schema exactly, never guess.
- `seed`, fix it to keep a consistent look across a product set, vary it for alternates.
- `includeCost` / dry-run, check the price before a batch, since a three-stage pipeline multiplies per-image cost (see `runware-run`).

## Quality bar

- The product reads as photographed, not generated: natural materials, purposeful light, a clean and quiet background.
- Cutouts have clean edges with no halo or leftover background fringe, especially around fine detail and reflective surfaces.
- A composited product sits in its scene with a correct contact shadow and matching light direction, not pasted on top.
- Brand color and background are exactly what was specified, not approximated.
- No invented logos, awards, claims, or copy. Any on-pack text matches the user's verbatim input. Retry the failing stage only.

## Related skills

`runware-run`, `runware-models`, `runware-prompting`; `edit-image` (relight, recolor, background swap on an existing shot), `character-consistency` (same product across multiple shots), `composite-scene` (place the product with other elements).

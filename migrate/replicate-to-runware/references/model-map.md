# Verified Replicate ↔ Runware model pairs

These pairs come from Runware's own Replicate↔AIR model mapping data, synced as of 2026-08-24 and re-checked against live catalog status on 2026-08-27 (51 mapped slugs remain after that re-check — some of the originally-probed rows were among those removed for being deprecated). Many of these were also spot-checked with a real, paid Replicate API call, not just a name-based pairing — but after the 2026-08-27 removals, don't assume a specific row was one of the probed ones without checking `worked-example.md`; treat every remaining row as a verified *pairing*, not necessarily a verified *end-to-end call*. Every AIR listed here was confirmed `"status": "live"` in https://runware.ai/docs/models/index.json as of the re-check date; rows whose AIR had gone `deprecated`, `coming-soon`, or disappeared from the catalog entirely were removed.

This is a point-in-time sync, not a live link — it will drift out of date as both catalogs add/remove models. A pairing being missing here doesn't mean no equivalent exists, only that it hasn't been synced into this file yet.

**Don't trust a model's display name at face value** — version numbers and naming aren't always self-consistent across platforms (a model called "Veo 3" in one place can actually correspond to a 3.1-generation model elsewhere). If a pairing here looks off, verify against a second source — the model's actual behavior, or a real test call — rather than assuming either this file or a display name is automatically right.

Where a single Runware AIR maps to different Replicate slugs depending on operation (text-to-video vs. image-to-video), both are listed as separate rows.

## Image (text-to-image)

| Runware AIR | Replicate slug |
|---|---|
| `bfl:2@1` | `black-forest-labs/flux-1.1-pro` |
| `bfl:2@2` | `black-forest-labs/flux-1.1-pro-ultra` |
| `bfl:5@1` | `black-forest-labs/flux-2-pro` |
| `bfl:6@1` | `black-forest-labs/flux-2-flex` |
| `bfl:7@1` | `black-forest-labs/flux-2-max` |
| `runware:100@1` | `black-forest-labs/flux-schnell` |
| `runware:101@1` | `black-forest-labs/flux-dev` |
| `runware:400@1` | `black-forest-labs/flux-2-dev` |
| `ideogram:2@1` | `ideogram-ai/ideogram-v2a` |
| `ideogram:3@1` | `ideogram-ai/ideogram-v2` |
| `ideogram:4@1` | `ideogram-ai/ideogram-v3-quality` |
| `recraft:v4@0` | `recraft-ai/recraft-v4` |
| `recraft:v4-pro@0` | `recraft-ai/recraft-v4-pro` |
| `recraft:v4.1@0` | `recraft-ai/recraft-v4.1` |
| `recraft:v4.1-pro@0` | `recraft-ai/recraft-v4.1-pro` |
| `runware:5@1` | `stability-ai/stable-diffusion-3` |
| `civitai:101055@128078` | `stability-ai/sdxl` |
| `openai:1@1` | `openai/gpt-image-1` |
| `openai:gpt-image@2` | `openai/gpt-image-2` |
| `alibaba:wan@2.7-image` | `wan-video/wan-2.7-image` |

Note: `openai/gpt-image-1` is a BYOK model (you supply your own OpenAI billing) and had no scrapable Replicate-side pricing as of the last sync — worth confirming current billing behavior directly if cost matters for the migration. `stability-ai/sdxl` also had no scrapable price at last check.

## Image editing / upscaling / utility

**The `Operation` column is not all `taskType: "imageInference"`** — verified per-model against the live schema: `upscale` rows use `taskType: "upscale"`, `vectorize` rows use `taskType: "vectorize"`, and `background removal` rows use `taskType: "removeBackground"`. Every other operation here (edit, inpaint/fill) is confirmed `imageInference` — the references/mask image is a parameter on the task, not a different taskType. Still confirm against the specific model's own `schema.json` rather than pattern-matching from this note alone.

| Runware AIR | Replicate slug | Operation |
|---|---|---|
| `bfl:3@1` | `black-forest-labs/flux-kontext-pro` | edit |
| `bfl:4@1` | `black-forest-labs/flux-kontext-max` | edit |
| `runware:106@1` | `black-forest-labs/flux-kontext-dev` | edit |
| `bfl:1@2` | `black-forest-labs/flux-fill-pro` | inpaint/fill |
| `runware:102@1` | `black-forest-labs/flux-fill-dev` | inpaint/fill |
| `recraft:1@1` | `recraft-ai/recraft-vectorize` | vectorize |
| `runware:504@1` | `nightmareai/real-esrgan` | upscale |
| `runware:500@1` | `philz1337x/clarity-upscaler` | upscale |
| `bria:2@1` | `bria/remove-background` | background removal |

Note: `philz1337x/clarity-upscaler` had no scrapable Replicate-side price at last check.

## Video

| Runware AIR | Operation | Replicate slug |
|---|---|---|
| `klingai:6@1` | text-to-video | `kwaivgi/kling-v2.5-turbo-pro` |
| `bytedance:2@1` | text-to-video | `bytedance/seedance-1-pro` |
| `bytedance:2@2` | text-to-video | `bytedance/seedance-1-pro-fast` |
| `bytedance:seedance@1.5-pro` | text-to-video | `bytedance/seedance-1.5-pro` |
| `bytedance:seedance@2.0` | text-to-video | `bytedance/seedance-2.0` |
| `bytedance:seedance@2.0-fast` | text-to-video | `bytedance/seedance-2.0-fast` |
| `bytedance:5@1` | image-to-video | `bytedance/omni-human` |
| `bytedance:5@2` | text-to-video | `bytedance/omni-human-1.5` |
| `google:3@2` | text-to-video | `google/veo-3.1` |
| `google:3@3` | text-to-video | `google/veo-3.1-fast` |
| `google:veo@3.1-lite` | text-to-video | `google/veo-3.1-lite` |
| `alibaba:wan@2.6` | text-to-video | `wan-video/wan-2.6-t2v` |
| `alibaba:wan@2.7` | text-to-video | `wan-video/wan-2.7-t2v` |
| `runware:201@1` | text-to-video | `wan-video/wan-2.5-t2v` |
| `minimax:3@1` | text-to-video | `minimax/hailuo-02` |
| `minimax:4@2` | image-to-video | `minimax/hailuo-2.3-fast` |
| `pixverse:1@2` | text-to-video | `pixverse/pixverse-v4` |
| `pixverse:1@3` | text-to-video | `pixverse/pixverse-v4.5` |
| `pixverse:1@5` | text-to-video | `pixverse/pixverse-v5` |
| `pixverse:1@8` | text-to-video | `pixverse/pixverse-v6` |
| `lightricks:ltx@2.3-fast` | text-to-video | `lightricks/ltx-2.3-fast` |
| `xai:grok-imagine@video` | text-to-video | `xai/grok-imagine-video` |
| `heygen:avatar@4` | avatar/digital-twin | `heygen/avatar-iv` |

Note: `pixverse/pixverse-v4` and `pixverse/pixverse-v4.5` had ambiguous, non-standard pricing signals at last check (didn't look like a normal $-per-video rate) — verify actual billing directly rather than trusting a quick scrape for these two specifically. `heygen/avatar-iv` requires a real `avatar_id`/`voice_id` to call at all — it's not a generic-input model like the others.

# Verified Fal ↔ Runware model pairs

These pairs come from Runware's own Fal↔AIR model mapping data, synced as of 2026-08-24 and re-checked against live catalog status on 2026-08-27 — not inferred from model names. Every AIR listed here was confirmed `"status": "live"` in https://runware.ai/docs/models/index.json as of the re-check date; rows whose AIR had gone `deprecated`, `coming-soon`, or disappeared from the catalog entirely were removed. Treat every remaining row here as a verified, currently-live pairing. If a model isn't listed, don't guess a pairing — search https://runware.ai/docs/models/index.json for the same creator/family and say explicitly that the pairing is inferred, not verified.

This is a point-in-time sync, not a live link — it will drift out of date as both catalogs add/remove models. A pairing being missing here doesn't mean no equivalent exists, only that it hasn't been synced into this file yet.

Where a single Runware AIR maps to different Fal endpoints depending on operation (text-to-video vs. image-to-video), both are listed as separate rows.

## Image (text-to-image)

| Runware AIR | Fal endpoint |
|---|---|
| `runware:z-image@turbo` | `fal-ai/z-image/turbo` |
| `runware:z-image@0` | `fal-ai/z-image/base` |
| `runware:5@1` | `fal-ai/stable-diffusion-v3-medium` |
| `runware:97@1` | `fal-ai/hidream-i1-full` |
| `runware:97@2` | `fal-ai/hidream-i1-dev` |
| `runware:97@3` | `fal-ai/hidream-i1-fast` |
| `runware:100@1` | `fal-ai/flux/schnell` |
| `runware:101@1` | `fal-ai/flux/dev` |
| `runware:400@1` | `fal-ai/flux-2` |
| `runware:400@2` | `fal-ai/flux-2/klein/9b` |
| `runware:400@3` | `fal-ai/flux-2/klein/9b/base` |
| `runware:400@4` | `fal-ai/flux-2/klein/4b` |
| `runware:400@5` | `fal-ai/flux-2/klein/4b/base` |
| `runware:201@10` | `fal-ai/wan-25-preview/text-to-image` |
| `bfl:2@1` | `fal-ai/flux-pro/v1.1` |
| `bfl:2@2` | `fal-ai/flux-pro/v1.1-ultra` |
| `bfl:5@1` | `fal-ai/flux-2-pro` |
| `bfl:6@1` | `fal-ai/flux-2-flex` |
| `bfl:7@1` | `fal-ai/flux-2-max` |
| `ideogram:2@1` | `fal-ai/ideogram/v2a` |
| `ideogram:3@1` | `fal-ai/ideogram/v2` |
| `ideogram:4@1` | `fal-ai/ideogram/v3` |
| `ideogram:4@0` | `ideogram/v4` |
| `google:4@1` | `fal-ai/nano-banana` |
| `google:4@2` | `fal-ai/nano-banana-pro` |
| `google:4@3` | `fal-ai/nano-banana-2` |
| `bytedance:5@0` | `fal-ai/bytedance/seedream/v4/text-to-image` |
| `bytedance:seedream@4.5` | `fal-ai/bytedance/seedream/v4.5/text-to-image` |
| `bytedance:seedream@5.0-lite` | `fal-ai/bytedance/seedream/v5/lite/text-to-image` |
| `xai:grok-imagine@image` | `xai/grok-imagine-image` |
| `xai:grok-imagine@image-quality` | `xai/grok-imagine-image/quality/text-to-image` |
| `openai:1@1` | `fal-ai/gpt-image-1/text-to-image` |
| `openai:gpt-image@2` | `openai/gpt-image-2` |
| `recraft:v4@0` | `fal-ai/recraft/v4/text-to-image` |
| `recraft:v4-pro@0` | `fal-ai/recraft/v4/pro/text-to-image` |
| `recraft:v4.1@0` | `fal-ai/recraft/v4.1/text-to-image` |
| `recraft:v4.1-pro@0` | `fal-ai/recraft/v4.1/pro/text-to-image` |
| `recraft:v4.1-utility@0` | `fal-ai/recraft/v4.1/utility/text-to-image` |
| `krea:krea@2-large` | `krea/v2/large/text-to-image` |
| `krea:krea@2-medium` | `krea/v2/medium/text-to-image` |
| `rundiffusion:120@100` | `rundiffusion-fal/juggernaut-flux/base` |
| `rundiffusion:110@101` | `rundiffusion-fal/juggernaut-flux/lightning` |
| `rundiffusion:130@100` | `rundiffusion-fal/juggernaut-flux/pro` |
| `bria:20@1` | `bria/fibo/generate` |
| `bria:20@3` | `bria/fibo-lite/generate` |
| `klingai:kling-image@3` | `fal-ai/kling-image/v3/text-to-image` |
| `klingai:kling-image@o3` | `fal-ai/kling-image/o3/text-to-image` |
| `klingai:kling-image@o1` | `fal-ai/kling-image/o1` |
| `alibaba:wan@2.6-image` | `wan/v2.6/text-to-image` |
| `alibaba:wan@2.7-image` | `fal-ai/wan/v2.7/text-to-image` |
| `alibaba:wan@2.7-image-pro` | `fal-ai/wan/v2.7/pro/text-to-image` |
| `alibaba:qwen-image@2.0` | `fal-ai/qwen-image-2/text-to-image` |
| `alibaba:qwen-image@2.0-pro` | `fal-ai/qwen-image-2/pro/text-to-image` |
| `alibaba:qwen-image@2512` | `fal-ai/qwen-image-2512` |
| `runware:108@1` | `fal-ai/qwen-image-2512` |
| `civitai:101055@128078` | `fal-ai/fast-sdxl` |
| `luma:uni@1` | `luma/agent/uni-1/v1/text-to-image` |
| `luma:uni@1-max` | `luma/agent/uni-1/v1/max` |
| `imagineart:1@5` | `imagineart/imagineart-1.5-preview/text-to-image` |
| `baidu:ernie-image@0` | `fal-ai/ernie-image` |
| `baidu:ernie-image@turbo` | `fal-ai/ernie-image/turbo` |
| `runware:107@1` | `fal-ai/flux-1/krea` |

Note: `runware:108@1` and `alibaba:qwen-image@2512` both point at the same Fal endpoint (`fal-ai/qwen-image-2512`) — two Runware AIRs, one underlying Fal model. Not a typo.

## Image editing / upscaling / utility

**The `Operation` column is not all `taskType: "imageInference"`** — verified per-model against the live schema: `upscale` rows use `taskType: "upscale"`, `vectorize`/`text-to-vector` rows use `taskType: "vectorize"`, and `background removal` rows use `taskType: "removeBackground"`. Every other operation here (edit, inpaint/fill, erase, virtual try-on, outpaint, remix, layered edit) is confirmed `imageInference` — the references/mask image is a parameter on the task, not a different taskType. Still confirm against the specific model's own `schema.json` rather than pattern-matching from this note alone.

| Runware AIR | Fal endpoint | Operation |
|---|---|---|
| `bfl:3@1` | `fal-ai/flux-pro/kontext` | edit |
| `bfl:4@1` | `fal-ai/flux-pro/kontext/max` | edit |
| `bfl:1@2` | `fal-ai/flux-pro/v1/fill` | inpaint/fill |
| `bfl:flux@erase` | `fal-ai/flux-pro/v1/erase` | erase |
| `bfl:flux@vto` | `fal-ai/flux-2-lora-gallery/virtual-tryon` | virtual try-on |
| `bfl:flux@outpainting` | `fal-ai/flux-2-pro/outpaint` | outpaint |
| `ideogram:3@2` | `fal-ai/ideogram/v2/remix` | remix |
| `ideogram:4@2` | `fal-ai/ideogram/v3/remix` | remix |
| `ideogram:4@3` | `fal-ai/ideogram/v3/edit` | edit |
| `runware:106@1` | `fal-ai/flux-kontext/dev` | edit |
| `runware:108@20` | `fal-ai/qwen-image-edit` | edit |
| `runware:108@22` | `fal-ai/qwen-image-edit-plus` | edit |
| `runware:102@1` | `fal-ai/flux/fill/dev` | inpaint/fill |
| `alibaba:qwen-image@layered` | `fal-ai/qwen-image-layered` | layered edit |
| `bria:21@1` | `bria/fibo-edit/edit` | edit |
| `bria:2@1` | `fal-ai/bria/background/remove` | background removal |
| `runware:112@1`, `@2`, `@3`, `@5`, `@6`, `@7`, `@8`, `@9`, `@10` | `fal-ai/birefnet/v2` | background removal (all these Runware AIR variants point at the same Fal model) |
| `runware:500@1` | `fal-ai/clarity-upscaler` | upscale |
| `runware:504@1` | `fal-ai/esrgan` | upscale |
| `recraft:1@1` | `fal-ai/recraft/vectorize` | vectorize |
| `recraft:v4@vector` | `fal-ai/recraft/v4/text-to-vector` | text-to-vector |
| `recraft:v4-pro@vector` | `fal-ai/recraft/v4/pro/text-to-vector` | text-to-vector |

## Video

| Runware AIR | Operation | Fal endpoint |
|---|---|---|
| `klingai:6@1` | text-to-video | `fal-ai/kling-video/v2.5-turbo/pro/text-to-video` |
| `klingai:kling-video@2.6-pro` | text-to-video | `fal-ai/kling-video/v2.6/pro/text-to-video` |
| `klingai:kling-video@3-standard` | text-to-video | `fal-ai/kling-video/v3/standard/text-to-video` |
| `klingai:kling-video@3-pro` | text-to-video | `fal-ai/kling-video/v3/pro/text-to-video` |
| `klingai:kling-video@3-4k` | text-to-video | `fal-ai/kling-video/v3/4k/text-to-video` |
| `klingai:kling-video@3.0-turbo` | text-to-video | `fal-ai/kling-video/v3/turbo/standard/text-to-video` |
| `klingai:kling-video@o3-standard` | text-to-video | `fal-ai/kling-video/o3/standard/text-to-video` |
| `klingai:kling-video@o3-pro` | text-to-video | `fal-ai/kling-video/o3/pro/text-to-video` |
| `klingai:kling-video@o3-4k` | text-to-video | `fal-ai/kling-video/o3/4k/text-to-video` |
| `klingai:kling@o1` | text-to-video | `fal-ai/kling-video/o1/pro/text-to-video` |
| `klingai:kling@o1-standard` | text-to-video | `fal-ai/kling-video/o1/standard/text-to-video` |
| `runware:201@1` | text-to-video | `fal-ai/wan-25-preview/text-to-video` |
| `alibaba:wan@2.6` | text-to-video | `wan/v2.6/text-to-video` |
| `alibaba:wan@2.6-flash` | image-to-video | `wan/v2.6/image-to-video/flash` |
| `alibaba:wan@2.7` | text-to-video | `fal-ai/wan/v2.7/text-to-video` |
| `alibaba:happyhorse@1.0` | text-to-video | `alibaba/happy-horse/text-to-video` |
| `alibaba:happyhorse@1.0` | image-to-video | `alibaba/happy-horse/image-to-video` |
| `alibaba:happyhorse@1.1` | text-to-video | `alibaba/happy-horse/v1.1/text-to-video` |
| `alibaba:happyhorse@1.1` | image-to-video | `alibaba/happy-horse/v1.1/image-to-video` |
| `minimax:3@1` | text-to-video | `fal-ai/minimax/hailuo-02/pro/text-to-video` |
| `minimax:4@1` | text-to-video | `fal-ai/minimax/hailuo-2.3/pro/text-to-video` |
| `lightricks:ltx@2` | text-to-video | `fal-ai/ltx-2/text-to-video` |
| `lightricks:ltx@2.3` | text-to-video | `fal-ai/ltx-2.3/text-to-video` |
| `lightricks:ltx@2.3-fast` | text-to-video | `fal-ai/ltx-2.3/text-to-video/fast` |
| `bytedance:2@1` | text-to-video | `fal-ai/bytedance/seedance/v1/pro/text-to-video` |
| `bytedance:2@2` | text-to-video | `fal-ai/bytedance/seedance/v1/pro/fast/text-to-video` |
| `bytedance:seedance@1.5-pro` | text-to-video | `fal-ai/bytedance/seedance/v1.5/pro/text-to-video` |
| `bytedance:seedance@2.0` | text-to-video | `bytedance/seedance-2.0/text-to-video` |
| `bytedance:seedance@2.0-fast` | text-to-video | `bytedance/seedance-2.0/fast/text-to-video` |
| `pixverse:1@2` | text-to-video | `fal-ai/pixverse/v4/text-to-video` |
| `pixverse:1@3` | text-to-video | `fal-ai/pixverse/v4.5/text-to-video` |
| `pixverse:1@5` | text-to-video | `fal-ai/pixverse/v5/text-to-video` |
| `pixverse:1@6` | text-to-video | `fal-ai/pixverse/v5.5/text-to-video` |
| `pixverse:1@6` | image-to-video | `fal-ai/pixverse/v5.5/image-to-video` |
| `pixverse:1@8` | text-to-video | `fal-ai/pixverse/v6/text-to-video` |
| `pixverse:1@8` | image-to-video | `fal-ai/pixverse/v6/image-to-video` |
| `vidu:1@1` | text-to-video | `fal-ai/vidu/q1/text-to-video` |
| `vidu:2@0` | text-to-video | `fal-ai/vidu/q2/text-to-video` |
| `vidu:3@2` | image-to-video | `fal-ai/vidu/q2/image-to-video/turbo` |
| `vidu:4@1` | text-to-video | `fal-ai/vidu/q3/text-to-video` |
| `vidu:4@1` | image-to-video | `fal-ai/vidu/q3/image-to-video` |
| `vidu:4@2` | text-to-video | `fal-ai/vidu/q3/text-to-video/turbo` |
| `heygen:avatar@4` | digital twin / avatar | `fal-ai/heygen/avatar4/digital-twin` |
| `xai:grok-imagine@video` | text-to-video | `xai/grok-imagine-video/text-to-video` |
| `google:3@2` | text-to-video | `fal-ai/veo3.1` |
| `google:3@3` | text-to-video | `fal-ai/veo3.1/fast` |
| `google:veo@3.1-lite` | text-to-video | `fal-ai/veo3.1/lite` |
| `luma:ray@3.2` | text-to-video | `luma/agent/ray/v3.2/text-to-video` |

Note: several `klingai` and `pixverse` AIRs pair with *different* Fal endpoints depending on text-to-video vs. image-to-video — always match on operation, not just model family, when picking a row.

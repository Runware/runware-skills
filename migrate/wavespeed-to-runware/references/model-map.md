# Verified WaveSpeed ↔ Runware model pairs

These pairs come from Runware's own WaveSpeed↔AIR model mapping data, synced as of 2026-08-24 and re-checked against live catalog status on 2026-08-27. Most of these are catalog-matched by ID, not individually confirmed by an executed generation call — treat them as "the ID pairing is real and comes from a matched catalog entry," not "the request/response behavior has been confirmed end to end" for that specific model. One pairing, `runware:101@1` ↔ `wavespeed-ai/flux-dev`, has been confirmed with a real, fully executed generation call (see `worked-example.md`) — it's a reliable template for what the general request/response pattern looks like, even for models that haven't been individually tested. Every AIR listed here was confirmed `"status": "live"` in https://runware.ai/docs/models/index.json as of the re-check date; rows whose AIR had gone `deprecated`, `coming-soon`, or disappeared from the catalog entirely were removed.

This is a point-in-time sync, not a live link — it will drift out of date as both catalogs change. A pairing missing here doesn't mean no equivalent exists, only that it hasn't been synced into this file yet.

Where a single Runware AIR maps to different WaveSpeed model IDs depending on operation (text-to-image vs. image-to-image, text-to-video vs. image-to-video), both are listed as separate rows.

## Image (text-to-image)

| Runware AIR | WaveSpeed model ID |
|---|---|
| `runware:100@1` | `wavespeed-ai/flux-schnell` |
| `runware:101@1` | `wavespeed-ai/flux-dev` |
| `bfl:2@1` | `wavespeed-ai/flux-1.1-pro` |
| `bfl:2@2` | `wavespeed-ai/flux-1.1-pro-ultra` |
| `runware:400@1` | `wavespeed-ai/flux-2-dev/text-to-image` |
| `bfl:5@1` | `wavespeed-ai/flux-2-pro/text-to-image` |
| `bfl:6@1` | `wavespeed-ai/flux-2-flex/text-to-image` |
| `bfl:7@1` | `wavespeed-ai/flux-2-max/text-to-image` |
| `runware:400@2` | `wavespeed-ai/flux-2-klein-9b/text-to-image` |
| `runware:400@3` | `wavespeed-ai/flux-2-klein-base-9b/text-to-image` |
| `runware:400@4` | `wavespeed-ai/flux-2-klein-4b/text-to-image` |
| `runware:400@5` | `wavespeed-ai/flux-2-klein-base-4b/text-to-image` |
| `ideogram:2@1` | `ideogram-ai/ideogram-v2a-turbo` |
| `ideogram:3@1` | `ideogram-ai/ideogram-v2` |
| `ideogram:4@1` | `ideogram-ai/ideogram-v3-quality` |
| `ideogram:4@0` | `ideogram-ai/ideogram-v4` |
| `recraft:v4@0` | `recraft-ai/recraft-v4/text-to-image` |
| `recraft:v4-pro@0` | `recraft-ai/recraft-v4-pro/text-to-image` |
| `recraft:v4.1@0` | `recraft-ai/recraft-v4.1/text-to-image` |
| `recraft:v4.1-pro@0` | `recraft-ai/recraft-v4.1-pro/text-to-image` |
| `recraft:v4.1-utility@0` | `recraft-ai/recraft-v4.1/text-to-image-utility` |
| `runware:97@1` | `wavespeed-ai/hidream-i1-full` |
| `runware:97@2` | `wavespeed-ai/hidream-i1-dev` |
| `runware:108@1` | `wavespeed-ai/qwen-image/text-to-image` |
| `alibaba:qwen-image@2.0` | `wavespeed-ai/qwen-image-2.0/text-to-image` |
| `alibaba:qwen-image@2.0-pro` | `wavespeed-ai/qwen-image-2.0-pro/text-to-image` |
| `alibaba:qwen-image@2512` | `wavespeed-ai/qwen-image/text-to-image-2512` |
| `runware:z-image@turbo` | `wavespeed-ai/z-image/turbo` |
| `runware:z-image@0` | `wavespeed-ai/z-image/base` |
| `openai:gpt-image@2` | `openai/gpt-image-2/text-to-image` |
| `xai:grok-imagine@image` | `x-ai/grok-imagine-image/text-to-image` |
| `xai:grok-imagine@image-quality` | `x-ai/grok-imagine-image-quality/text-to-image` |
| `bytedance:5@0` | `bytedance/seedream-v4` |
| `bytedance:seedream@4.5` | `bytedance/seedream-v4.5` |
| `bytedance:seedream@5.0-lite` | `bytedance/seedream-v5.0-lite` |
| `google:4@1` | `google/nano-banana/text-to-image` |
| `google:4@2` | `google/nano-banana-pro/text-to-image` |
| `google:4@3` | `google/nano-banana-2/text-to-image` |
| `alibaba:wan@2.6-image` | `alibaba/wan-2.6/text-to-image` |
| `alibaba:wan@2.7-image` | `alibaba/wan-2.7/text-to-image` |
| `alibaba:wan@2.7-image-pro` | `alibaba/wan-2.7/text-to-image-pro` |
| `runware:201@10` | `alibaba/wan-2.5/text-to-image` |
| `krea:krea@2-large` | `wavespeed-ai/krea-v2-large/text-to-image` |
| `krea:krea@2-medium` | `wavespeed-ai/krea-v2-medium/text-to-image` |
| `bria:20@1` | `bria/fibo` |
| `baidu:ernie-image@0` | `wavespeed-ai/ernie-image/text-to-image` |
| `baidu:ernie-image@turbo` | `wavespeed-ai/ernie-image/text-to-image-turbo` |
| `luma:uni@1` | `luma/uni-v1/text-to-image` |
| `klingai:kling-image@3` | `kwaivgi/kling-image-v3/text-to-image` |
| `klingai:kling-image@o3` | `kwaivgi/kling-image-o3/text-to-image` |
| `klingai:kling-image@o1` | `kwaivgi/kling-image-o1` |

## Image editing / upscaling / utility

**The `Operation` column is not all `taskType: "imageInference"`** — verified per-model against the live schema: `upscale` rows use `taskType: "upscale"`, `vectorize` rows use `taskType: "vectorize"`, and `background removal` rows use `taskType: "removeBackground"`. Every other operation here (edit, layered edit) is confirmed `imageInference` — the references/mask image is a parameter on the task, not a different taskType. Still confirm against the specific model's own `schema.json` rather than pattern-matching from this note alone.

| Runware AIR | WaveSpeed model ID | Operation |
|---|---|---|
| `bfl:3@1` | `wavespeed-ai/flux-kontext-pro` | edit |
| `bfl:4@1` | `wavespeed-ai/flux-kontext-max` | edit |
| `runware:106@1` | `wavespeed-ai/flux-kontext-dev` | edit |
| `runware:102@1` | `wavespeed-ai/flux-fill-dev` | inpaint/fill |
| `bfl:flux@erase` | `wavespeed-ai/flux-v1-pro/erase` | erase |
| `runware:400@1` | `wavespeed-ai/flux-2-dev/edit` | edit |
| `bfl:5@1` | `wavespeed-ai/flux-2-pro/edit` | edit |
| `bfl:6@1` | `wavespeed-ai/flux-2-flex/edit` | edit |
| `bfl:7@1` | `wavespeed-ai/flux-2-max/edit` | edit |
| `runware:400@2` | `wavespeed-ai/flux-2-klein-9b/edit` | edit |
| `runware:400@3` | `wavespeed-ai/flux-2-klein-base-9b/edit` | edit |
| `runware:400@4` | `wavespeed-ai/flux-2-klein-4b/edit` | edit |
| `runware:400@5` | `wavespeed-ai/flux-2-klein-base-4b/edit` | edit |
| `runware:108@20` | `wavespeed-ai/qwen-image/edit` | edit |
| `runware:108@22` | `wavespeed-ai/qwen-image/edit-plus` | edit |
| `alibaba:qwen-image@layered` | `wavespeed-ai/qwen-image/layered` | layered edit |
| `xai:grok-imagine@image` | `x-ai/grok-imagine-image/edit` | edit |
| `bytedance:5@0` | `bytedance/seedream-v4/edit` | edit |
| `bytedance:seedream@4.5` | `bytedance/seedream-v4.5/edit` | edit |
| `bytedance:seedream@5.0-lite` | `bytedance/seedream-v5.0-lite/edit` | edit |
| `google:4@1` | `google/nano-banana/edit` | edit |
| `google:4@2` | `google/nano-banana-pro/edit` | edit |
| `google:4@3` | `google/nano-banana-2/edit` | edit |
| `alibaba:wan@2.6-image` | `alibaba/wan-2.6/image-edit` | edit |
| `alibaba:wan@2.7-image` | `alibaba/wan-2.7/image-edit` | edit |
| `alibaba:wan@2.7-image-pro` | `alibaba/wan-2.7/image-edit-pro` | edit |
| `luma:uni@1` | `luma/uni-v1/edit` | edit |
| `klingai:kling-image@3` | `kwaivgi/kling-image-v3/edit` | edit |
| `klingai:kling-image@o3` | `kwaivgi/kling-image-o3/edit` | edit |
| `recraft:v4@vector` | `recraft-ai/recraft-v4/text-to-vector` | vectorize |
| `recraft:v4-pro@vector` | `recraft-ai/recraft-v4-pro/text-to-vector` | vectorize |
| `runware:504@1` | `wavespeed-ai/real-esrgan` | upscale |
| `bria:2@1` | `bria/remove-background` | background removal |
| `bria:21@1` | `bria/fibo/edit` | edit |

## Video

| Runware AIR | Operation | WaveSpeed model ID |
|---|---|---|
| `klingai:6@1` | text-to-video | `kwaivgi/kling-v2.5-turbo-pro/text-to-video` |
| `klingai:6@1` | image-to-video | `kwaivgi/kling-v2.5-turbo-pro/image-to-video` |
| `klingai:kling-video@2.6-pro` | text-to-video | `kwaivgi/kling-v2.6-pro/text-to-video` |
| `klingai:kling-video@2.6-pro` | image-to-video | `kwaivgi/kling-v2.6-pro/image-to-video` |
| `klingai:kling-video@3-standard` | text-to-video | `kwaivgi/kling-v3.0-std/text-to-video` |
| `klingai:kling-video@3-standard` | image-to-video | `kwaivgi/kling-v3.0-std/image-to-video` |
| `klingai:kling-video@3-pro` | text-to-video | `kwaivgi/kling-v3.0-pro/text-to-video` |
| `klingai:kling-video@3-pro` | image-to-video | `kwaivgi/kling-v3.0-pro/image-to-video` |
| `klingai:kling-video@3-4k` | text-to-video | `kwaivgi/kling-v3.0-4k/text-to-video` |
| `klingai:kling-video@3-4k` | image-to-video | `kwaivgi/kling-v3.0-4k/image-to-video` |
| `klingai:kling-video@3.0-turbo` | text-to-video | `kwaivgi/kling-v3-turbo-std/text-to-video` |
| `klingai:kling-video@3.0-turbo` | image-to-video | `kwaivgi/kling-v3-turbo-std/image-to-video` |
| `klingai:kling-video@o3-standard` | text-to-video | `kwaivgi/kling-video-o3-std/text-to-video` |
| `klingai:kling-video@o3-standard` | image-to-video | `kwaivgi/kling-video-o3-std/image-to-video` |
| `klingai:kling-video@o3-pro` | text-to-video | `kwaivgi/kling-video-o3-pro/text-to-video` |
| `klingai:kling-video@o3-pro` | image-to-video | `kwaivgi/kling-video-o3-pro/image-to-video` |
| `klingai:kling-video@o3-4k` | text-to-video | `kwaivgi/kling-video-o3-4k/text-to-video` |
| `klingai:kling-video@o3-4k` | image-to-video | `kwaivgi/kling-video-o3-4k/image-to-video` |
| `klingai:kling@o1` | text-to-video | `kwaivgi/kling-video-o1/text-to-video` |
| `klingai:kling@o1` | image-to-video | `kwaivgi/kling-video-o1/image-to-video` |
| `klingai:kling@o1-standard` | text-to-video | `kwaivgi/kling-video-o1-std/text-to-video` |
| `klingai:kling@o1-standard` | image-to-video | `kwaivgi/kling-video-o1-std/image-to-video` |
| `bytedance:2@1` | text-to-video | `bytedance/seedance-v1-pro-t2v-1080p` |
| `bytedance:2@1` | image-to-video | `bytedance/seedance-v1-pro-i2v-1080p` |
| `bytedance:2@2` | text-to-video | `bytedance/seedance-v1-pro-fast/text-to-video` |
| `bytedance:2@2` | image-to-video | `bytedance/seedance-v1-pro-fast/image-to-video` |
| `bytedance:seedance@1.5-pro` | text-to-video | `bytedance/seedance-v1.5-pro/text-to-video` |
| `bytedance:seedance@1.5-pro` | image-to-video | `bytedance/seedance-v1.5-pro/image-to-video` |
| `bytedance:seedance@2.0` | text-to-video | `bytedance/seedance-2.0/text-to-video` |
| `bytedance:seedance@2.0` | image-to-video | `bytedance/seedance-2.0/image-to-video` |
| `bytedance:seedance@2.0-fast` | text-to-video | `bytedance/seedance-2.0-fast/text-to-video` |
| `bytedance:seedance@2.0-fast` | image-to-video | `bytedance/seedance-2.0-fast/image-to-video` |
| `bytedance:5@1` | image-to-video | `bytedance/avatar-omni-human` |
| `bytedance:5@2` | text-to-video | `bytedance/avatar-omni-human-1.5` |
| `bytedance:5@2` | image-to-video | `bytedance/avatar-omni-human-1.5` |
| `google:3@2` | text-to-video | `google/veo3.1/text-to-video` |
| `google:3@2` | image-to-video | `google/veo3.1/image-to-video` |
| `google:3@3` | text-to-video | `google/veo3.1-fast/text-to-video` |
| `google:3@3` | image-to-video | `google/veo3.1-fast/image-to-video` |
| `google:veo@3.1-lite` | text-to-video | `google/veo3.1-lite/text-to-video` |
| `google:veo@3.1-lite` | image-to-video | `google/veo3.1-lite/image-to-video` |
| `alibaba:wan@2.6` | text-to-video | `alibaba/wan-2.6/text-to-video` |
| `alibaba:wan@2.6` | image-to-video | `alibaba/wan-2.6/image-to-video` |
| `alibaba:wan@2.6-flash` | image-to-video | `alibaba/wan-2.6/image-to-video-flash` |
| `alibaba:wan@2.7` | text-to-video | `alibaba/wan-2.7/text-to-video` |
| `alibaba:wan@2.7` | image-to-video | `alibaba/wan-2.7/image-to-video` |
| `runware:201@1` | text-to-video | `alibaba/wan-2.5/text-to-video` |
| `runware:201@1` | image-to-video | `alibaba/wan-2.5/image-to-video` |
| `alibaba:happyhorse@1.0` | text-to-video | `alibaba/happyhorse-1.0/text-to-video` |
| `alibaba:happyhorse@1.0` | image-to-video | `alibaba/happyhorse-1.0/image-to-video` |
| `alibaba:happyhorse@1.1` | text-to-video | `alibaba/happyhorse-1.1/text-to-video` |
| `alibaba:happyhorse@1.1` | image-to-video | `alibaba/happyhorse-1.1/image-to-video` |
| `minimax:3@1` | text-to-video | `minimax/hailuo-02/t2v-pro` |
| `minimax:3@1` | image-to-video | `minimax/hailuo-02/i2v-pro` |
| `minimax:4@1` | text-to-video | `minimax/hailuo-2.3/t2v-pro` |
| `minimax:4@1` | image-to-video | `minimax/hailuo-2.3/i2v-pro` |
| `minimax:4@2` | image-to-video | `minimax/hailuo-2.3/fast` |
| `pixverse:1@3` | text-to-video | `pixverse/pixverse-v4.5-t2v` |
| `pixverse:1@3` | image-to-video | `pixverse/pixverse-v4.5-i2v` |
| `pixverse:1@5` | text-to-video | `pixverse/pixverse-v5-t2v` |
| `pixverse:1@5` | image-to-video | `pixverse/pixverse-v5-i2v` |
| `pixverse:1@6` | text-to-video | `pixverse/pixverse-v5.5/text-to-video` |
| `pixverse:1@8` | text-to-video | `pixverse/pixverse-v6/text-to-video` |
| `pixverse:1@8` | image-to-video | `pixverse/pixverse-v6/image-to-video` |
| `lightricks:ltx@2.3` | text-to-video | `wavespeed-ai/ltx-2.3/text-to-video` |
| `lightricks:ltx@2.3` | image-to-video | `wavespeed-ai/ltx-2.3/image-to-video` |
| `lightricks:ltx@2.3-fast` | image-to-video | `wavespeed-ai/ltx-2.3-spicy/image-to-video` |
| `vidu:1@1` | text-to-video | `vidu/text-to-video-q1` |
| `vidu:1@1` | image-to-video | `vidu/image-to-video-q1` |
| `vidu:2@0` | text-to-video | `vidu/text-to-video-2.0` |
| `vidu:2@0` | image-to-video | `vidu/image-to-video-2.0` |
| `vidu:3@2` | image-to-video | `vidu/image-to-video-q2-turbo` |
| `vidu:4@1` | text-to-video | `vidu/q3/text-to-video` |
| `vidu:4@1` | image-to-video | `vidu/q3/image-to-video` |
| `vidu:4@2` | image-to-video | `vidu/q3-turbo/image-to-video` |
| `luma:ray@3.2` | text-to-video | `luma/ray-3.2/text-to-video` |
| `luma:ray@3.2` | image-to-video | `luma/ray-3.2/image-to-video` |
| `xai:grok-imagine@video` | text-to-video | `x-ai/grok-imagine-video/text-to-video` |
| `xai:grok-imagine@video` | image-to-video | `x-ai/grok-imagine-video/image-to-video` |

Note: WaveSpeed splits far more models by operation (text-to-video vs. image-to-video, text-to-image vs. edit) than Fal or Replicate do at the ID level — always match on the specific operation the developer's call is doing, not just the model family, or you'll pick a slug that silently expects a different set of inputs.

---
name: video-upscale
description: >
  Increase a video's resolution and restore lost detail, up to 4K. Use when the
  user says "upscale this video", "make this clip higher-res / sharper", "enhance
  this footage", "remaster this old video", "4K this", "denoise this video", or
  "clean up this compressed clip". Covers video super-resolution and temporal
  detail restoration, not still images.
---

# Video upscale

Take a low-resolution or soft video and return a sharper, higher-resolution one, up to 4K, with detail recovered and compression noise reduced. The lever is a dedicated video-to-video model that upscales every frame while holding temporal consistency, so the result stays stable across frames instead of shimmering. This is a thin capability: only three models cover it today, so routing is simple but options are limited.

## Inputs to collect

- **The source video.** A URL or upload. (Ask only if none provided.)
- **Target resolution or factor.** A target like 1080p / 2K / 4K, or a 2x / 4x factor. What each model accepts differs, so confirm against the schema before promising one.
- **What needs fixing:** just too small/soft, or also noisy, compression-artifacted, or aliased. This nudges the model choice.
- **Frame rate**, if it should change (Topaz can resample fps). Otherwise keep the input's rate.

## Models

- **Default, highest quality: Topaz Starlight Precise 2.5** (`topazlabs:starlight-precise@2.5`). Diffusion-based enhancement that upscales, denoises, de-aliases, and sharpens with full temporal consistency and natural faces/skin. Best when detail recovery and a clean, photoreal result matter. Most expensive of the three.
- **Enterprise 4K restoration: ByteDance Video Upscaler** (`bytedance:50@1`). Boosts to 1080p, 2K, or 4K with denoising and motion enhancement, restores color, and cuts compression artifacts. Strong general pick for legacy films, UGC, and short clips. Cheapest per second.
- **2x / 4x with detail preservation: Bria Video Increase Resolution** (`bria:50@1`). Upscales 2x or 4x while preserving detail and transparency. Good for product footage and design workflows that need clean output without rework.
- All three are **video-to-video** (`op:upscale`), not still-image upscalers. For images, use `restore-and-upscale`.
- Confirm the live model + its schema via the `runware-models` and `runware-run` skills before calling. Never hardcode a stale choice.

## Workflow

1. **Pick the model** from the routing above based on quality bar, target resolution, and cost. Confirm it is `live` via `runware-models`.
2. **Resolve the schema** (`runware-run`) and confirm the input field and the resolution controls for that specific model. They differ (target width/height/fps on Topaz, `upscaleFactor` on Bria, server-chosen target on ByteDance).
3. **Upload the source** into `inputs.video` (required on all three).
4. **Run async and poll.** The taskType is `upscale`. Video upscaling is time-based, so it runs **asynchronously**: the task returns a `taskUUID`, then poll `getResponse` until terminal. Do not block a single sync call on a minutes-long job.
5. **Read the result.** It returns a `videoURL`. Inspect it at full resolution before returning.

## Technique

- **This is a one-pass operation, not a prompt.** None of the three takes a `positivePrompt`. You are not directing content, you are enhancing the existing frames. Pick the model and the target, run once, and judge the output. Don't try to steer it with text.
- **Don't chain upscalers.** Upscale once at the target you actually want. Running 2x then 2x again, or feeding one model's output into another, compounds artifacts and temporal flicker instead of improving sharpness.
- **Match the model to the footage.** Genuinely degraded or noisy capture where detail must be recovered and faces must stay natural, lean Topaz (diffusion recovers and sharpens detail). Legacy film / UGC / compressed clips where the goal is a clean 1080p-to-4K bump at low cost, lean ByteDance. A straightforward 2x or 4x on already-decent footage (product, design), Bria is the lean pick.
- **Temporal consistency is the whole point.** A good result stays stable frame to frame. If you see shimmer, crawling edges, or detail that pops in and out between frames, that's the model struggling with this footage, not a tuning knob you missed. Switch model class and rerun rather than re-running the same one.
- **Resolution is capped at 4K (3840x2160).** Asking for more than the model supports either clamps or fails. Confirm the ceiling per model before promising a target.

## Parameters that matter

- `inputs.video` is the source clip. Required on all three. Confirm the exact field against the live schema, never guess.
- **Topaz** (`topazlabs:starlight-precise@2.5`): `width` and `height` are the **target** output size and both are **required**, max **3840x2160**. Aspect ratio is preserved by computing the factor from the shorter edge. `fps` is optional, **15 to 120**, and must be at least the input's frame rate.
- **Bria** (`bria:50@1`): `upscaleFactor` is the only control, enum **2 or 4** (default 2). No width/height target.
- **ByteDance** (`bytedance:50@1`): no resolution parameters in the schema. The service targets 1080p / 2K / 4K from the input. Confirm how the target is selected via the live schema before promising a specific output size.
- `outputType` / `outputFormat` / `deliveryMethod` are video-output settings shared across all three. Defaults are fine for most runs.

## Quality bar

- Output is genuinely sharper and higher-resolution and still looks like the original footage. No plastic over-smoothing, no invented detail that wasn't there.
- **Temporal stability holds:** no shimmer, edge crawl, or flicker introduced by the upscale. If it appears, switch model class and rerun.
- The output hit the requested resolution/factor and the schema accepted the target (Topaz width/height within 3840x2160, Bria `upscaleFactor` in {2,4}).
- The job was run **async** and the `videoURL` was retrieved, not blocked on a sync call.

## Related skills

`runware-run`, `runware-models`, `runware-prompting`; `restore-and-upscale` (the still-image sibling: super-resolution and restoration for photos, where Topaz also appears as the video escape hatch).

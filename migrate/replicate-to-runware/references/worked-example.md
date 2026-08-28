# Worked examples: image and video

## Example 1 — image: `black-forest-labs/flux-1.1-pro` → `bfl:2@1`

### Before (Replicate)

```bash
curl -X POST https://api.replicate.com/v1/models/black-forest-labs/flux-1.1-pro/predictions \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: wait" \
  -d '{
    "input": {
      "prompt": "a photograph of an astronaut riding a horse",
      "aspect_ratio": "1:1",
      "seed": 42
    }
  }'
```

The actual completed prediction:

```json
{
  "id": "xwz6zpd3vnrmr0d072ssb1tqbr",
  "status": "succeeded",
  "output": "https://replicate.delivery/xezq/VT4iM0emMizgFyqeT6QmSsLK68ooWSsuHu0GKd06AZHfeLbcB/tmpatoamt96.webp",
  "metrics": { "predict_time": 3.436664102, "image_count": 1, "image_output_count": 1 }
}
```

Note `output` here is a **bare string**, not an array — a real `Prefer: wait` call to this model resolves synchronously in a few seconds.

List price: $0.04 per output image. Actual billed cost: $0.04 (this model bills per image, not per second, so `predict_time` doesn't factor into cost at all here — that's specific to video models, see Example 2).

### After (Runware)

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "<client-generated UUID v4>",
    "model": "bfl:2@1",
    "positivePrompt": "a photograph of an astronaut riding a horse",
    "width": 1024,
    "height": 1024,
    "seed": 42,
    "numberResults": 1,
    "includeCost": true,
    "deliveryMethod": "sync"
  }
]
```

`deliveryMethod` is set explicitly rather than left to default — `sync` happens to be `bfl:2@1`'s own schema default, but that default varies per model (Example 2 below uses a Kling model that's `async`-only), so state the choice rather than relying on an implicit default that won't hold if the model changes. `async` is the generally recommended choice outside of a deliberately verified fast-model case like this one — see the general integration skill.

The request body is always a JSON array of task objects, even for a single task — Runware doesn't accept a bare object at the top level.

(`aspect_ratio: "1:1"` is converted to explicit `width`/`height` here, following Runware's general convention — verify against `bfl:2@1`'s own schema whether it also accepts an `aspectRatio`-style field directly, since not every Runware model requires the width/height conversion.)

```json
{
  "data": [
    { "taskType": "imageInference", "taskUUID": "<uuid>", "imageURL": "https://im.runware.ai/...", "cost": 0.0013 }
  ]
}
```

### Compatibility report

**Maps cleanly**
- Auth scheme: `Authorization: Bearer <token>` on both sides — only the token value changes, unlike Fal's `Key` scheme.
- `prompt` → `positivePrompt`, `seed` → `seed`.

**Needs adjustment**
- Endpoint + envelope: per-model URL + `{"input": {...}}` wrapper → fixed endpoint + AIR in body + fields directly on the task object.
- `Prefer: wait` isn't a guarantee: this call resolved without needing a manual poll, but the header is a best-effort hint, not a deterministic sync mode like Runware's `deliveryMethod: "sync"` (where the target model's schema even offers `sync` — not all do). Keep a poll fallback regardless, and default to `deliveryMethod: "async"` rather than `sync` unless you've specifically verified the model supports it and you've chosen `sync` deliberately.
- Response shape: `output` is a bare URL string here. Other models return an array of URLs, or `{url: "..."}`, instead — the shape isn't consistent across Replicate's catalog. Runware is consistently `data[0].imageURL` regardless of model.

**No equivalent**
- Nothing to migrate for cost tracking — Runware gives you `cost` inline; Replicate gives you nothing, so there was never a "read the real cost" code path to carry over on this platform in the first place.

## Example 2 — video: `kwaivgi/kling-v2.1` (image-to-video) → Kling on Runware, and the billing gotcha

### Before (Replicate)

```bash
curl -X POST https://api.replicate.com/v1/models/kwaivgi/kling-v2.1/predictions \
  -H "Authorization: Bearer $REPLICATE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: wait" \
  -d '{
    "input": {
      "prompt": "a photograph of an astronaut riding a horse",
      "start_image": "https://assets.runware.ai/assets/inputs/25c6ba31-72c4-4abe-b46b-424451ca635c.jpg",
      "mode": "pro",
      "duration": 5
    }
  }'
```

Real completed prediction:

```json
{
  "id": "cc758zwt95rmw0czdfxa27gepw",
  "status": "succeeded",
  "output": "https://replicate.delivery/xezq/Mltxoznao14WNZnAzxjD2BMVeiRYQNVYHeQGTQsrKjdieYztA/tmpbs3919rd.mp4",
  "metrics": { "predict_time": 125.368277959 }
}
```

Note `output` here is a **bare string**, same as Example 1 for this platform — but other Replicate models return an array of URLs or `{url: "..."}` instead, so don't assume every model matches this shape.

List price: $0.05 per second of output video. The requested clip was 5 seconds.

**The gotcha, in real numbers**: `metrics.predict_time` was 125.37 seconds — how long Replicate's server actually took to render the request. If you compute cost as `predict_time × price_per_second`, you get **125.37 × $0.05 = $6.27**. The real billed cost was **$0.25** (`5 seconds requested × $0.05 = $0.25`) — a **~25x overcount** from using the wrong duration. This is a real, documented mistake, not a hypothetical edge case. If a developer's migration includes any cost-estimation logic reading `metrics.predict_time`, flag this explicitly before it ships.

### After (Runware)

```json
[
  {
    "taskType": "videoInference",
    "taskUUID": "<client-generated UUID v4>",
    "model": "klingai:kling-video@2.6-pro",
    "positivePrompt": "a photograph of an astronaut riding a horse",
    "duration": 5,
    "deliveryMethod": "async",
    "includeCost": true
  }
]
```

(Runware's Kling lineup has been renumbered since this example was captured — `klingai:5@2`, the AIR originally used here, is deprecated. `klingai:kling-video@2.6-pro` is shown as an illustrative live replacement, but don't take it on faith: confirm the current AIR for the Kling image-to-video tier you actually need against `model-map.md` or the live catalog before shipping this call. Separately, this example still needs an input image field for image-to-video — the exact Runware field name for that isn't confirmed either. Don't guess a field name here; check the model's own schema at `https://runware.ai/docs/models/index.json` before shipping this call.)

```json
{
  "data": [
    { "taskType": "videoInference", "taskUUID": "<uuid>", "videoURL": "https://vm.runware.ai/...", "cost": 0.xx }
  ]
}
```

### Compatibility report

**Maps cleanly**
- Auth scheme (`Bearer`, token swap only).
- `duration` → `duration` — same field name and semantics for this model family.

**Needs adjustment**
- Endpoint + envelope + async pattern, same as Example 1, plus: Replicate's `status` vocabulary (`starting`/`processing`/`succeeded`/`failed`/`canceled`) doesn't match Runware's (`processing`/`success`/`error`).
- Response `output` shape: bare string here, same as Example 1 — but other Replicate models return an array or `{url: "..."}` instead, so don't assume every model matches. Runware's `data[0].videoURL` is consistent regardless.
- The image-input field name for image-to-video isn't confirmed — verify before use rather than assume a name.

**No equivalent**
- Reading `metrics.predict_time` for any cost or duration purpose. Runware has no equivalent field to read because it doesn't need one — `cost` comes back correct, already computed, inline. Delete this logic rather than translate it, and specifically don't let `predict_time`-shaped thinking leak into how you validate or log Runware costs either.

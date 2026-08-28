# Worked example: `fal-ai/flux/dev` → `runware:101@1`

## Before (Fal)

```bash
curl -X POST https://queue.fal.run/fal-ai/flux/dev \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "a photograph of an astronaut riding a horse",
    "image_size": { "width": 1024, "height": 1024 },
    "num_inference_steps": 30,
    "seed": 42,
    "num_images": 1
  }'
```

This returns a `request_id` immediately (queue submit). The caller then polled `.../requests/{request_id}/status` until `COMPLETED`, then fetched `.../requests/{request_id}` for the result:

```json
{
  "images": [
    { "url": "https://v3b.fal.media/files/b/0a9f54fb/XmAz90S_w2W1i1T9GuZgA.jpg", "content_type": "image/jpeg" }
  ],
  "seed": 42
}
```

A separate call to Fal's billing-events endpoint confirmed the actual cost: **$0.025**.

## After (Runware)

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "<client-generated UUID v4>",
    "model": "runware:101@1",
    "positivePrompt": "a photograph of an astronaut riding a horse",
    "width": 1024,
    "height": 1024,
    "steps": 30,
    "seed": 42,
    "numberResults": 1,
    "outputType": "URL",
    "outputFormat": "JPG",
    "includeCost": true,
    "deliveryMethod": "sync"
  }
]
```

`deliveryMethod` is set explicitly here rather than left to default — `sync` happens to be `runware:101@1`'s own schema default, but that default varies per model (some models don't support `sync` at all), so relying on an implicit default instead of stating the choice is a good way to get surprised later when the model changes. See the general integration skill for why `async` is the recommended default in most other cases.

The request body is always a JSON array of task objects, even for a single task — Runware doesn't accept a bare object at the top level.

```json
{
  "data": [
    { "taskType": "imageInference", "taskUUID": "<the same UUID>", "imageURL": "https://im.runware.ai/...", "cost": 0.0013 }
  ]
}
```

(This is the shape for a direct/sync response or a `getResponse` poll — note there's no `errors` key at all on a clean success, only on failure, so don't assume both keys are always present. A webhook delivery for the same task would instead arrive flat — no `data` wrapper — since Runware's webhook payload and its direct-response payload use different envelopes. Don't assume whichever one you tested first is the only shape you'll see.)

One round trip, no separate poll or billing lookup needed. (Note: the $0.025 vs $0.0013 difference above is this specific pair of models' real list prices, not a general "Runware is Nx cheaper" claim — don't generalize a cost comparison from one model pair.)

## Compatibility report

**Maps cleanly**
- `prompt` → `positivePrompt` — same concept, rename only.
- `image_size.width` / `image_size.height` → `width` / `height` — same concept, unwrapped from the nested object into flat fields.
- `num_inference_steps` → `steps` — rename only.
- `seed` → `seed` — identical.
- `num_images` → `numberResults` — rename only.

**Needs adjustment**
- **Auth**: `Authorization: Key $FAL_KEY` → `Authorization: Bearer <RUNWARE_KEY>`. Different scheme keyword, not just a different key value.
- **Endpoint & model addressing**: Fal's model lives in the URL path (`/fal-ai/flux/dev`); Runware always calls `https://api.runware.ai/v1` and puts the model in the body as an AIR ID (`runware:101@1`). Any code that builds a URL per-model needs to become code that looks up an AIR ID per-model instead.
- **Request ID semantics**: Fal generates `request_id` server-side and hands it back to you. Runware expects you to generate `taskUUID` client-side *before* sending the request. If existing code assumes the ID only exists after the server responds, that assumption breaks.
- **Call shape collapses from 3 requests to 1 — but treat this as an illustration of `sync`, not a general recommendation**: Fal's queue pattern here required submit → poll status → fetch result (3 HTTP calls, because Fal's queue model applies even to fast image models). `deliveryMethod: "sync"` returns the image in the same request for this specific model, so the polling code for this specific call *could* be deleted, not just rewritten. That said: `sync` isn't available on every model (some models are `async`-only — check the target model's own schema before assuming), and even where it is available, the general integration skill recommends defaulting to `async` anyway, since a held-open sync connection is more exposed to timeouts under load than a poll loop is. Don't delete polling code on the strength of this one example — verify per model, and lean toward keeping it.
- **Response field name and shape**: `images[0].url` → `data[0].imageURL`. Still array-indexed on both sides — the win isn't "no more array indexing," it's that the key name and array structure become predictable instead of per-model. (And note: that's true for a direct/sync response. A webhook delivery for this same call would instead hand you a flat `imageURL` with no `data` wrapper at all — worth testing both paths if you use webhooks, not just whichever one you happened to test first.)

**No equivalent**
- Fal's billing-events lookup by request ID: Runware has no per-request equivalent. Delete that code path; request `includeCost: true` and read `cost` directly off the response instead — cost is known at response time here, not looked up separately. If the goal is auditing spend over time rather than one request's cost, Runware's account-management API (`getUsageActivity`, `getDetails`) covers that at the day/model/key level instead.

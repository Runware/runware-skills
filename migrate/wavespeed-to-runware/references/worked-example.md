# Worked example: `wavespeed-ai/flux-dev` → `runware:101@1`

## Before (WaveSpeed)

Submit:

```bash
curl -X POST https://api.wavespeed.ai/api/v3/wavespeed-ai/flux-dev \
  -H "Authorization: Bearer $WAVESPEED_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "a photograph of an astronaut riding a horse",
    "size": "1024*1024",
    "num_inference_steps": 28,
    "seed": 42,
    "num_images": 1,
    "output_format": "jpeg"
  }'
```

Real submit response:

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "ded09b4c9b104fb88a38feab74c9d54c",
    "model": "wavespeed-ai/flux-dev",
    "input": {
      "prompt": "a photograph of an astronaut riding a horse",
      "size": "1024*1024",
      "num_inference_steps": 28,
      "seed": 42,
      "num_images": 1,
      "output_format": "jpeg",
      "guidance_scale": 3.5,
      "strength": 0.8,
      "enable_base64_output": false,
      "enable_sync_mode": false
    },
    "outputs": [],
    "urls": { "get": "https://api.wavespeed.ai/api/v3/predictions/ded09b4c9b104fb88a38feab74c9d54c/result" },
    "status": "created",
    "created_at": "2026-08-25T00:49:21.479Z"
  }
}
```

Note WaveSpeed echoes back the fully-resolved input (including every default you didn't set explicitly) in the submit response — useful for confirming what actually ran.

Poll (`GET` the `urls.get` value from the submit response):

```bash
curl https://api.wavespeed.ai/api/v3/predictions/ded09b4c9b104fb88a38feab74c9d54c/result \
  -H "Authorization: Bearer $WAVESPEED_KEY"
```

Status progresses `created` → `processing` → `completed`. Final response:

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "ded09b4c9b104fb88a38feab74c9d54c",
    "model": "wavespeed-ai/flux-dev",
    "outputs": ["https://d2h7xmz5gqybh9.cloudfront.net/output/a900f7dd-4454-4490-8bc7-f67f3691a236-u2_080cffd4-ae89-493c-93f8-2619cdeffe70.jpeg"],
    "urls": { "get": "https://api.wavespeed.ai/api/v3/predictions/ded09b4c9b104fb88a38feab74c9d54c/result" },
    "status": "completed",
    "created_at": "2026-08-25T00:49:21.481196722Z",
    "timings": { "inference": 11076 }
  }
}
```

Real pricing lookup for this exact model:

```bash
curl -X POST https://api.wavespeed.ai/api/v3/model/pricing \
  -H "Authorization: Bearer $WAVESPEED_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "model_id": "wavespeed-ai/flux-dev", "inputs": {} }'
```

```json
{ "code": 200, "message": "success", "data": { "unit_price": 0.012, "currency": "USD" } }
```

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
    "steps": 28,
    "seed": 42,
    "numberResults": 1,
    "outputFormat": "JPG",
    "includeCost": true,
    "deliveryMethod": "sync"
  }
]
```

`deliveryMethod` is set explicitly here rather than left to default — `sync` happens to be `runware:101@1`'s own schema default, but that default (and whether `sync` is even offered) varies per model, so state the choice rather than relying on an implicit default that won't hold for a different model. `async` is the generally recommended choice outside of a deliberately verified fast-model case like this one.

Real response:

```json
{
  "data": [
    {
      "taskType": "imageInference",
      "imageUUID": "107fa9b6-baa9-4e44-8f18-7c3e0a0a8a4c",
      "taskUUID": "<uuid>",
      "cost": 0.0013,
      "seed": 42,
      "imageURL": "https://im.runware.ai/image/os/a08dlim3/ws/3/ii/107fa9b6-baa9-4e44-8f18-7c3e0a0a8a4c.jpg"
    }
  ]
}
```

Note there's no `errors` key at all here — it's only present when something fails, not as a permanent empty array alongside `data`.

(Cost figures aren't directly comparable in general — $0.012 and $0.0013 are this specific model pair's real prices, not a platform-wide claim. Don't generalize a cost comparison from one pair.)

## Compatibility report

**Maps cleanly**
- Auth scheme (`Bearer`, token swap only).
- `prompt` → `positivePrompt`, `seed` → `seed`, `num_inference_steps` → `steps`, `num_images` → `numberResults`.
- Response envelope concept: both platforms wrap results in a top-level object rather than returning the model output as the bare body — `data` on both sides, though the surrounding keys differ (see below).

**Needs adjustment**
- Endpoint: per-model URL (`/api/v3/{model_id}`) → fixed endpoint + AIR in body.
- Envelope shape: WaveSpeed wraps as `{"code", "message", "data": {...single object...}}`; Runware wraps as `{"data": [...array...]}` with no `errors` key on success. Don't assume either platform's `data` is the same shape (object vs. array) or that a same-named wrapper key means the same structure.
- `size: "1024*1024"` → separate `width`/`height` integers — parse the asterisk-delimited string, don't assume any other delimiter.
- `output_format` (`jpeg`/`png`/`webp`, lowercase) → Runware's `outputFormat`. Use the documented uppercase values (`JPG`/`PNG`/`WEBP`) — that's what the model schema specifies, even though direct testing shows the API also accepts lowercase without erroring. Don't rely on that leniency; write `"JPG"`, not `"jpg"`.
- Submit-then-poll pattern: WaveSpeed's submit response hands you the poll URL directly (`data.urls.get`); Runware's async pattern instead has you construct a fixed `{"taskType": "getResponse", "taskUUID": ...}` poll request yourself. Status vocabulary also differs (`created`/`processing`/`completed` vs. Runware's `processing`/`success`/`error`).
- Pre-flight cost lookup: WaveSpeed's dedicated `/model/pricing` endpoint computes exact cost before you submit. Runware's `cost` is only known from the generation response itself (`includeCost: true`) — there's no equivalent pre-flight call, so a developer showing a cost estimate before generating needs a different approach (e.g. a fixed price table) rather than a live pre-flight call.
- `enable_base64_output` (boolean) → `outputType` (enum: `"URL"` default, `"base64Data"`, `"dataURI"`) — a real equivalent exists, but it's a value-shape change, not a same-typed field swap: `enable_base64_output: true` becomes `outputType: "base64Data"` (or `"dataURI"` for the `data:`-prefixed form), not a boolean.

**No equivalent**
- WaveSpeed's `input` echo (the submit response repeats back every resolved parameter, including defaults) has no Runware equivalent — Runware's response doesn't echo the request back at all, only the output.

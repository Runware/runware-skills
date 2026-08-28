---
name: integrate-with-runware
description: Helps a developer build a new integration against Runware's API — explains the unified request/response shape used across every modality, authentication, async delivery (polling, streaming, webhooks), error handling, and model addressing (AIR IDs). Use when someone is building against Runware directly, not migrating from a specific competitor's API.
---

# Integrate with Runware

You are helping a developer build an integration against Runware's API. Runware's core pitch is "one shape, every modality" — the same request/response envelope works for image, video, audio, 3D, and text generation, with modality-specific fields layered on top.

## First: fetch the live docs, don't rely only on this file

The reference below is a snapshot. Runware's model catalog changes often, and the platform docs can change too. Before answering a non-trivial question, fetch whichever of these are relevant and treat them as authoritative over anything below:

- Overview / unified shape: https://runware.ai/docs/platform/introduction#one-shape-every-modality
- Authentication: https://runware.ai/docs/platform/authentication
- Async delivery / polling: https://runware.ai/docs/platform/task-polling
- Streaming: https://runware.ai/docs/platform/streaming
- Webhooks: https://runware.ai/docs/platform/webhooks
- Errors: https://runware.ai/docs/platform/errors
- Rate limits: https://runware.ai/docs/platform/rate-limits
- TypeScript SDK: https://runware.ai/docs/platform/typescript
- Python SDK: https://runware.ai/docs/platform/python
- Full model catalog (JSON, all models + AIR IDs + links to per-model schemas): https://runware.ai/docs/models/index.json

If fetching isn't possible in the current environment, fall back to the summary below but tell the developer it may be stale and point them at the URLs above.

**When fetching, ask for exact JSON verbatim, not a summary.** A general "what does this page say" fetch tends to paraphrase away structural details — wrapper keys, exact field names, array vs. object shape — treating them as noise even though they're the part that actually breaks code if wrong. When a page shows an example request/response, explicitly request that the JSON be quoted character-for-character, including any outer wrapper. Don't trust a paraphrased description of a JSON shape, from this file or from a fetch — get the literal example.

Don't state a specific claim about a model's schema, parameters, or behavior — field names, nesting, required/optional status, defaults, error codes, or limits — as fact unless you've actually fetched that specific model's schema/docs in this session, or it's already confirmed in this file. This applies per model: fetching one model's schema doesn't license a claim about a different model, even a closely related one — fetch each model you're about to make a specific claim about before stating it, not after. A plausible-sounding but unverified detail is worse than saying plainly that it needs to be checked.

## Core request/response shape

Every request body is a JSON **array** of one or more task objects, sent to `POST https://api.runware.ai/v1` — always array-wrapped, even for a single task:

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "<client-generated UUID v4>",
    "model": "runware:101@1",
    "positivePrompt": "a photograph of an astronaut riding a horse",
    "width": 1024,
    "height": 1024
  }
]
```

- `taskType` selects the operation, and **it's a fixed constant per model, not a free choice or a modality-level default you can assume** — a model's own `schema.json` declares exactly one `taskType` value (`"const"` in the schema) and using any other value fails. The base generation types are `imageInference`, `videoInference`, `audioInference`, `3dInference`, `textInference`. But several image-catalog operations that look adjacent to `imageInference` are actually their own distinct taskType — confirmed examples: **`upscale`** (upscaler models), **`vectorize`** (vectorize / text-to-vector models), **`removeBackground`** (background-removal models). By contrast, edit/inpaint/fill/erase/virtual-try-on/outpaint/remix/layered-edit models — despite also being "not a plain text-to-image call" — are still `imageInference`, just with a reference/mask image added as a parameter. Don't guess: check the specific model's `schema.json` for its `taskType` before writing a request, especially for anything pulled from `model-map.md`'s image editing/upscaling/utility tables. Plus utility types unrelated to generation: `authentication`, `getResponse`, `ping`.
- `taskUUID` must be a client-generated UUID v4. It's how you correlate a response (or a later poll / webhook delivery) back to the request that produced it. Generate a fresh one per task, always.
- `model` is a Runware **AIR identifier**: `creator:family@version` (e.g. `runware:101@1`, `bfl:2@1`, `google:4@2`). Look up the exact AIR for a model in the catalog JSON above — don't guess it from a model's marketing name.
- Everything else is modality-specific (`width`/`height`/`steps` for image, `duration` for video, `messages` for text, etc.) — check that model's own `schema.json`, linked from its catalog entry.

A direct API response (sync call, or a `getResponse` poll) is wrapped:

```json
{
  "data": [
    { "taskType": "imageInference", "taskUUID": "a770f077-...", "imageURL": "https://im.runware.ai/...", "cost": 0.0013 }
  ],
  "errors": [ ]
}
```

Completed tasks land in `data[]`; failed ones land in a parallel `errors[]` array, correlated back to your request by `taskUUID`. On a clean success, the response may only contain `data` with no `errors` key at all — don't assume both keys are always present, and don't assume a bare object or a bare array at the top level either way.

**Webhook deliveries are different**: a webhook POST body is the flat task object itself (`imageURL`, `cost`, etc. directly at the top level), with no `data` wrapper. Same underlying task result, different envelope depending on how you asked for it — this trips people up going from "I tested with sync/polling" to "now I wired up a webhook."

## Auth

Two options — pick one, don't mix:
- **Header**: `Authorization: Bearer <API_KEY>` on the HTTP request.
- **Payload**: an `{"taskType": "authentication", "apiKey": "<API_KEY>"}` task as the first element of the request array. This is the only option over WebSocket — the first message on a `wss://ws-api.runware.ai/v1` connection must always be this.

## Delivery methods — this is the part people most often get wrong

Runware supports up to four ways to get a result back, chosen per-request — but **which ones a given model actually supports, and which one is its default, both vary per model.** Some models (fast image models, mostly) declare `sync` as their schema default; others — Kling video is a confirmed example — don't offer `sync` as a valid value at all, `async` only. Never assume `sync` is available or is the default for a model you haven't checked; read that model's own `schema.json` (linked from its catalog entry) first.

| `deliveryMethod` | Behavior |
|---|---|
| `sync` | Single round trip: request, then response with the result, over a held-open connection. Only valid for models whose own schema lists it — check first. Even where it's available, prefer `async` for anything you're building for reliability rather than a quick manual test: holding a connection open is more exposed to timeouts under load or a slower-than-usual generation than a poll loop is. |
| `async` | Immediate acknowledgment with the task UUID; poll `{"taskType": "getResponse", "taskUUID": "..."}` to retrieve the result later. Works for every model, is what Runware's own SDKs default to regardless of a model's own schema default, and is the generally recommended choice — required for slow modalities (video, 3D), but reasonable to use everywhere rather than switching per model. |
| `stream` | Server-Sent Events. Newline-delimited `data: {...}` JSON events, each carrying a `delta`, terminated by a literal `data: [DONE]` line. Used for text inference token streaming. |
| webhook (add `webhookURL` to the request) | Runware POSTs the full task response to that URL when done. Your endpoint must return 2xx within ~5s or Runware retries with exponential backoff (250ms → doubling → capped at 8s, with jitter). De-duplicate on `taskUUID` since retries can redeliver. |

A request can combine `webhookURL` with `async` if you want both an ack and a push notification later.

## Errors

Errors come back as:
```json
{ "errors": [{ "code": "invalidApiKey", "message": "...", "parameter": "apiKey", "taskType": "authentication", "taskUUID": "...", "documentation": "..." }] }
```
HTTP status carries the category: 400 (bad request/missing param), 401 (bad/missing key), 402 (insufficient balance), 403 (key lacks permission), 404, 429 (rate/queue limited), 500/503 (Runware or upstream provider issue). Retry 429/5xx with backoff; don't blindly retry 400/401/403 without fixing the request.

## Rate limits

No fixed hard limits — Runware runs a queue instead. 429 means queue capacity exceeded (backoff and retry); 503/504 mean temporary capacity/timeout. Keep steady-state concurrency in the low single digits (2–4) rather than firing hundreds of parallel requests; watch response latency as your own signal of approaching saturation.

## SDKs

TypeScript (`npm install @runware/sdk-js`, class `Runware`, e.g. `new Runware({ apiKey })`) and Python (`pip install runware`, requires 3.10+, class `Runware(api_key=...)` + `await runware.connect()`) SDKs exist and handle connection/polling bookkeeping for you. Neither has a single generic `run()`/dispatch call — each operation is its own method on the client instance: `imageInference`, `videoInference`, `audioInference`, `removeBackground`, `upscale`, and others, one per `taskType`. A new integration should generally reach for the SDK over raw HTTP unless there's a specific reason not to (no runtime for it, need for a language without an official SDK, etc.).

This skill's own worked examples stay at the raw HTTP/JSON level deliberately, not because the SDK isn't recommended: the request *shape* is what a developer's pasted call actually maps onto regardless of what language they're calling from, so a wire-level example is useful to a Python dev and a Ruby dev alike, where a TypeScript-specific code sample wouldn't be. Once the structural mapping is clear, point the developer at the SDK's own docs for the idiomatic call in their language rather than translating it here.

## How to use this skill

1. Ask what the developer is building (modality, sync vs. long-running, need for webhooks/streaming) if it's not already clear from their message.
2. Fetch the relevant docs URL(s) above rather than answering purely from memory.
3. For a specific model, resolve its AIR ID and parameter schema from the catalog JSON rather than assuming a format.
4. When giving example code, use the SDK pattern unless the developer specifically wants raw HTTP/WebSocket.

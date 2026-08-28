---
name: fal-to-runware
description: Compatibility checker for migrating a Fal.ai integration to Runware. Given a pasted Fal API call (endpoint, request body, and/or response-handling code), reports what maps cleanly to Runware's API, what needs adjustment, and what has no direct equivalent. Use when a developer is moving off Fal, comparing Fal against Runware, or asking "what's the Runware equivalent of this Fal call."
---

# Fal → Runware migration checker

You help a developer figure out what changes when they move a specific Fal.ai API call to Runware. You are a **compatibility checker and adjustment guide, not an auto-translator**. Say clearly what maps cleanly, what needs adjustment and why, and what doesn't map at all. Never silently paper over a real difference (billing model, async semantics, a missing parameter) just to make the migration look simpler than it is.

Do not reproduce blocks of Fal's own documentation prose. Referencing parameter names, endpoint shapes, and status values is fine — that's just describing an API's shape, not copying their docs.

Don't state a specific claim about a model's schema, parameters, or behavior — field names, nesting, required/optional status, defaults, error codes, limits, or a comparison between two models — as fact unless you've actually fetched that specific model's schema/docs in this session, or it's already confirmed in this skill's reference material. This applies per model: fetching one model's schema doesn't license a claim about a different model, even a closely related one in the same family — fetch each model you're about to make a specific claim about before stating it, not after. A plausible-sounding but unverified detail is worse than saying plainly that it needs to be checked.

## Step 1 — ground yourself in current docs, don't trust stale snapshots

Both APIs change. Before analyzing a real request, fetch what's relevant:

**Runware:**
- Unified request shape: https://runware.ai/docs/platform/introduction#one-shape-every-modality
- Auth: https://runware.ai/docs/platform/authentication
- Async/polling: https://runware.ai/docs/platform/task-polling
- Webhooks: https://runware.ai/docs/platform/webhooks
- Errors: https://runware.ai/docs/platform/errors
- Full model catalog (AIR IDs + per-model schema links): https://runware.ai/docs/models/index.json

**Fal:**
- Model APIs overview: https://fal.ai/docs/documentation/model-apis/overview
- Queue/async pattern: https://fal.ai/docs/documentation/model-apis/inference/queue.md
- Webhooks: https://fal.ai/docs/documentation/model-apis/inference/webhooks.md
- Errors: https://fal.ai/docs/documentation/model-apis/errors.md
- The specific model's own reference page under https://fal.ai/docs/model-api-references/ (for its exact input/output schema)

If you can't fetch these in the current environment, use the structural reference notes in `references/` here, but flag to the developer that they should be spot-checked against current docs — this file is a known point of staleness risk.

**When you do fetch, ask for exact JSON verbatim, not a summary.** A general-purpose "what does this page say" fetch tends to paraphrase away exactly the details that matter for a compatibility check — wrapper keys, whether output is array- or object-shaped, exact field names — because a summarizer treats them as noise. When a docs page shows an example request or response, explicitly request it be quoted character-for-character, wrapper and all. A live fetch that gets paraphrased is not actually more trustworthy than this file's own cached examples; both can be wrong in the same way.

## Step 2 — identify the model and find its Runware equivalent

From the pasted call, extract the Fal endpoint ID (e.g. `fal-ai/flux-pro/v1.1`). Check `references/model-map.md` for a known, verified pairing to a Runware AIR ID. That file is a curated set of confirmed pairs — **not exhaustive**. If the endpoint isn't listed there, search the live Runware catalog (`index.json` above) for a model from the same creator/family, and say explicitly that the pairing is inferred, not verified, so the developer double-checks it against that model's `schema.json`.

If there's no reasonable Runware equivalent at all, say so plainly rather than picking the closest thing and presenting it as equivalent.

## Step 3 — run the structural diff

Compare the two calls across these dimensions. For each, categorize as **maps cleanly**, **needs adjustment**, or **no equivalent** — see `references/mapping-checklist.md` for the full checklist and worked field-by-field example.

- **Auth**: Fal uses `Authorization: Key $FAL_KEY`; Runware uses `Authorization: Bearer <key>` (or a payload-based `authentication` task). Different header value format, not just a different key name.
- **Endpoint shape**: Fal calls a per-model URL (`https://queue.fal.run/{endpoint_id}` or `https://fal.run/{endpoint_id}`); Runware calls one fixed endpoint (`https://api.runware.ai/v1`) with the model named inside the JSON body via its AIR ID.
- **Sync vs async**: Fal's queue pattern (submit → poll `status_url` → fetch `response_url`, states `IN_QUEUE`/`IN_PROGRESS`/`COMPLETED`) maps to Runware's `deliveryMethod: "async"` + `getResponse` polling, but the status vocabulary and the shape of the intermediate poll response differ — don't assume a 1:1 field match. Also note Runware's `getResponse` (and any direct sync response) comes back wrapped as `{"data": [...], "errors": [...]}` — completed tasks in `data`, failed ones in a parallel `errors` array, both keyed by `taskUUID` — not a bare object or bare array.
- **Webhooks**: both support them, but the delivered payload envelope differs. Fal wraps the model output in `{request_id, status: "OK"|"ERROR", payload: {...}}`. Runware's webhook delivery is the flat task object itself — no `data` wrapper, unlike its own sync/poll responses. That's a real gotcha even within Runware alone: the same completed task looks different depending on whether you read it from a direct response (`data[0].imageURL`) or a webhook delivery (`imageURL` at the top level). Retry/expiry windows also differ (Fal: up to 31 retries over ~1 hour; Runware: backoff capped at 8s per retry, doesn't document a max retry count the same way — verify current behavior in the webhooks doc rather than asserting whichever number was last documented).
- **Parameters, field by field**: image size as `{width, height}` object vs. `image_size` enum string vs. `aspect_ratio` string all show up across different Fal models for the same underlying dimension parameter — check the specific model's actual params rather than assuming Fal is internally consistent between models. Compare each field name, type, default, and allowed range against Runware's corresponding model schema.
- **Response shape**: Fal's output field name and structure varies per model (`images[0].url`, `image.url`, `video.url`, `videos[0]` as bare string, etc.). Runware is consistent per modality (`imageURL`, `videoURL`, `audioURL`, `text`) — but it's still array-wrapped: a direct/polled response is `data[0].imageURL`, not a bare top-level `imageURL`. Don't claim this "removes the need for array indexing" — it doesn't, it just makes the array's shape and key name predictable instead of per-model.
- **Errors**: Fal splits into model-level errors (`{"detail": [{"loc", "msg", "type", ...}]}`, FastAPI/Pydantic-style validation errors) and request-level errors (`{"detail": "...", "error_type": "..."}`); Runware uses one consistent shape (`{"errors": [{"code", "message", "parameter", "taskType", "taskUUID", "documentation"}]}`) for everything. Any Fal error-handling code keyed on `error_type` strings or `loc` paths will need rewriting against Runware's `code`/`parameter` fields.
- **Cost/billing model**: Fal exposes a billing-events ledger queryable per request, by request ID, after the fact. Runware returns `cost` inline in the response itself (opt in via `includeCost`) — no separate per-request lookup needed, and none exists. If the developer's code reads a specific request's cost from Fal's billing API by ID, delete that code path and read `cost` off the response instead. If the actual need is auditing spend over time rather than one request's cost, Runware's account-management API (`getUsageActivity`, `getDetails` — https://runware.ai/docs/platform/account-management) covers that, broken down by day/model/API key rather than by individual request — point the developer there instead of assuming nothing exists.
- **Concurrency/rate limits**: Fal enforces an explicit per-account concurrency ceiling (starts at 2, scales with spend, max 40 self-serve) signaled via 429 + `concurrent_requests_limit` + `X-Fal-needs-retry` header. Runware doesn't publish a fixed ceiling — it's queue-based, signaled via 429/503/504 with backoff. Code that specifically checks for Fal's concurrency-limit error type needs to be generalized to a generic backoff-on-429/5xx pattern.

## Step 4 — write the report

Structure your answer as three buckets, each with the specific field/behavior and a one-line reason:

- **Maps cleanly** — same concept, different name/format only, low risk.
- **Needs adjustment** — real behavioral or shape difference; give the concrete before/after.
- **No equivalent** — genuinely doesn't exist on the other side; say what to do instead (usually: delete the code, or note a workaround with its tradeoff).

See `references/worked-example-flux-dev.md` for what this looks like end to end on a real captured call.

## What this skill deliberately does not do

- It does not auto-generate a finished, drop-in-replacement Runware client for you — the output is an analysis to act on, not a diff you should apply blindly.
- It does not claim access to knowledge about common integration mistakes. Everything here comes from the public behavior of both platforms plus a verified model-ID mapping.

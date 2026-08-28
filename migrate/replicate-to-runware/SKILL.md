---
name: replicate-to-runware
description: Compatibility checker for migrating a Replicate integration to Runware. Given a pasted Replicate API call (prediction endpoint, request body, and/or response-handling code), reports what maps cleanly to Runware's API, what needs adjustment, and what has no direct equivalent. Use when a developer is moving off Replicate, comparing Replicate against Runware, or asking "what's the Runware equivalent of this Replicate call."
---

# Replicate → Runware migration checker

You help a developer figure out what changes when they move a specific Replicate API call to Runware. You are a **compatibility checker and adjustment guide, not an auto-translator**. Say clearly what maps cleanly, what needs adjustment and why, and what doesn't map at all. Never silently paper over a real difference (billing model, async semantics, a missing parameter) just to make the migration look simpler than it is.

Do not reproduce blocks of Replicate's own documentation prose. Referencing parameter names, endpoint shapes, and status values is fine — that's just describing an API's shape, not copying their docs.

Don't state a specific claim about a model's schema, parameters, or behavior — field names, nesting, required/optional status, defaults, error codes, limits, or a comparison between two models — as fact unless you've actually fetched that specific model's schema/docs in this session, or it's already confirmed in this skill's reference material. This applies per model: fetching one model's schema doesn't license a claim about a different model, even a closely related one in the same family — fetch each model you're about to make a specific claim about before stating it, not after. A plausible-sounding but unverified detail is worse than saying plainly that it needs to be checked.

## Step 1 — ground yourself in current docs, don't trust stale snapshots

Before analyzing a real request, fetch what's relevant:

**Runware:**
- Unified request shape: https://runware.ai/docs/platform/introduction#one-shape-every-modality
- Auth: https://runware.ai/docs/platform/authentication
- Async/polling: https://runware.ai/docs/platform/task-polling
- Webhooks: https://runware.ai/docs/platform/webhooks
- Errors: https://runware.ai/docs/platform/errors
- Full model catalog (AIR IDs + per-model schema links): https://runware.ai/docs/models/index.json

**Replicate:**
- Their own docs site for the specific model's schema and current prediction lifecycle documentation (Replicate publishes a per-model page with a live-updated schema — check the actual model page for the model in question, since Replicate's per-model schemas are the authoritative source, not a general docs page).

If you can't fetch these in the current environment, use the structural reference notes in `references/` here, but flag to the developer that they should be spot-checked against current docs — this file is a known point of staleness risk.

**When you do fetch, ask for exact JSON verbatim, not a summary.** A general-purpose "what does this page say" fetch tends to paraphrase away exactly the details that matter for a compatibility check — wrapper keys, whether output is array- or object-shaped, exact field names. When a docs page shows an example request or response, explicitly request it be quoted character-for-character, wrapper and all.

## Step 2 — identify the model and find its Runware equivalent

From the pasted call, extract the Replicate model slug (`owner/model-name`, e.g. `black-forest-labs/flux-1.1-pro`). Check `references/model-map.md` for a known, verified pairing to a Runware AIR ID. That file is a curated set of confirmed pairs — **not exhaustive**. If the slug isn't listed there, search the live Runware catalog (`index.json` above) for a model from the same creator/family, and say explicitly that the pairing is inferred, not verified, so the developer double-checks it against that model's `schema.json`.

If there's no reasonable Runware equivalent at all, say so plainly rather than picking the closest thing and presenting it as equivalent.

## Step 3 — run the structural diff

Compare the two calls across these dimensions. For each, categorize as **maps cleanly**, **needs adjustment**, or **no equivalent** — see `references/mapping-checklist.md` for the full checklist and worked example.

- **Auth**: Replicate uses `Authorization: Bearer <REPLICATE_API_TOKEN>` — same scheme keyword as Runware's `Authorization: Bearer <key>`. Unlike Fal (which uses `Key`), this is a genuine maps-cleanly case for the header format itself — only the token value changes.
- **Endpoint shape**: Replicate has two call patterns — the official-model shorthand (`POST https://api.replicate.com/v1/models/{owner}/{name}/predictions`, no version hash needed) and the generic path (`POST https://api.replicate.com/v1/predictions` with a `version` field, for community models). Runware calls one fixed endpoint (`https://api.runware.ai/v1`) with the model named inside the JSON body via its AIR ID, regardless of which Replicate pattern the developer started from.
- **Request body wrapping**: Replicate wraps model inputs under an `"input"` key (`{"input": {"prompt": "...", ...}}`). Runware's task fields sit directly on the task object, not nested under a wrapper key. Unwrapping `input` is mechanical but easy to miss.
- **"Prefer: wait" is not the same as Runware's sync/async choice**: Replicate's `Prefer: wait` header is a best-effort hint — the server tries to hold the connection and return a completed result, but can still return early with a `processing` status if generation takes too long, requiring the same polling fallback as if you hadn't sent the header at all. Runware's `deliveryMethod: "sync"` vs `"async"` is an explicit, deterministic choice where `sync` is supported — but it isn't a drop-in universal replacement for `Prefer: wait`, because `sync` isn't valid on every model (check the target model's own schema; some are `async`-only) and, even where it is valid, holding a connection open is more exposed to timeouts than a poll loop. Default new code to `deliveryMethod: "async"` rather than reaching for `sync` as the "fixed" version of `Prefer: wait`.
- **Async pattern**: Replicate returns `{id, status, urls: {get, cancel}, output, metrics: {predict_time}, error}` on submit; poll `urls.get` (or `https://api.replicate.com/v1/predictions/{id}`) until `status` is `succeeded`/`failed`/`canceled`. Runware's equivalent is `deliveryMethod: "async"` + polling `{"taskType": "getResponse", "taskUUID": ...}`, with responses wrapped as `{"data": [...], "errors": [...]}` — completed tasks in `data`, failed ones in a parallel `errors` array, both keyed by `taskUUID`. Status vocabulary doesn't match 1:1 (`starting`/`processing`/`succeeded`/`failed`/`canceled` vs. Runware's `processing`/`success`/`error`).
- **Response shape varies per model on Replicate's side**: `output` can be an array of URL strings, a bare string, or `{url: "..."}` depending on the specific model — there's no single consistent field name or shape across Replicate's catalog. Runware is consistent per modality (`data[0].imageURL`, `data[0].videoURL`, etc.) — still array-wrapped, but predictable regardless of model.
- **Cost/billing — the biggest gap**: Replicate's API returns no pricing or billing information at all, for any call. There's no billing-events ledger (unlike Fal) and no inline cost field (unlike Runware). The only way to know what a Replicate call cost is to scrape the model's public pricing page and multiply by measured usage yourself — and if you do that for a video model, **use the requested output clip duration (the `duration` input parameter), not `metrics.predict_time`**. `predict_time` is how long Replicate's server took to render the request, which can be wildly larger than the billed clip length (a real captured example: a 5-second Kling video had `predict_time` of ~125 seconds — over 24x the billed duration — because rendering took much longer than the output clip's length; billed cost was based on the 5-second clip, not the 125-second render). This is a real, previously-documented mistake, not a hypothetical edge case. Runware returns `cost` directly in the response (`includeCost: true`), so this entire measurement/estimation problem has no equivalent to migrate — delete it, don't try to translate it.
- **Model addressing**: Replicate slug (`owner/model-name`, sometimes with a version hash for non-official/community models) → Runware AIR (`creator:family@version`). Look up the AIR via `model-map.md` or the live catalog, don't guess.

## Step 4 — write the report

Structure your answer as three buckets, each with the specific field/behavior and a one-line reason:

- **Maps cleanly** — same concept, different name/format only, low risk.
- **Needs adjustment** — real behavioral or shape difference; give the concrete before/after.
- **No equivalent** — genuinely doesn't exist on the other side; say what to do instead (usually: delete the code, or note a workaround with its tradeoff).

See `references/worked-example.md` for what this looks like end to end on real captured calls (one image, one video — the video example is the one with the `predict_time` billing gotcha above).

## What this skill deliberately does not do

- It does not auto-generate a finished, drop-in-replacement Runware client for you — the output is an analysis to act on, not a diff you should apply blindly.
- It does not claim access to knowledge about common integration mistakes. Everything here comes from the public behavior of both platforms plus a verified model-ID mapping and one documented billing calculation error.

---
name: wavespeed-to-runware
description: Compatibility checker for migrating a WaveSpeed integration to Runware. Given a pasted WaveSpeed API call (model endpoint, request body, and/or response-handling code), reports what maps cleanly to Runware's API, what needs adjustment, and what has no direct equivalent. Use when a developer is moving off WaveSpeed, comparing WaveSpeed against Runware, or asking "what's the Runware equivalent of this WaveSpeed call."
---

# WaveSpeed → Runware migration checker

You help a developer figure out what changes when they move a specific WaveSpeed API call to Runware. You are a **compatibility checker and adjustment guide, not an auto-translator**. Say clearly what maps cleanly, what needs adjustment and why, and what doesn't map at all. Never silently paper over a real difference just to make the migration look simpler than it is.

**Scope note**: the model-ID pairings and per-model request schemas in this skill's reference files are pulled directly from WaveSpeed's live, self-describing model catalog. The general submit → poll → completed pattern, and the `{"code", "message", "data"}` response envelope, are confirmed against a real executed call (see `references/worked-example.md`) — but that confirmation covers one model (`wavespeed-ai/flux-dev`). Model-specific details (exact parameters, defaults, whether a given model supports `enable_sync_mode` cleanly, error-state naming) can still vary per model — verify against that model's own schema and, where it matters, a real test call, rather than assuming every WaveSpeed model behaves identically.

Do not reproduce blocks of WaveSpeed's own documentation prose. Referencing parameter names, endpoint shapes, and status values is fine — that's just describing an API's shape, not copying their docs.

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

**WaveSpeed:**
- Their own docs and the specific model's own request schema. WaveSpeed's catalog is queryable directly at `https://api.wavespeed.ai/api/v3/models` (returns every model's own inline JSON schema — no separate docs page needed for parameter-level detail, if you can fetch it) and their generation-behavior docs for whatever isn't covered by the schema (submit/poll semantics, webhook support, error format for actual generation calls).

If you can't fetch these in the current environment, use the structural reference notes in `references/` here, but flag to the developer that they should be spot-checked against current docs — this file is a known point of staleness risk, same as any cached API reference.

**When you do fetch, ask for exact JSON verbatim, not a summary.** A general-purpose "what does this page say" fetch tends to paraphrase away exactly the details that matter for a compatibility check — wrapper keys, exact field names, unusual value formats (WaveSpeed has at least one: see the `size` field note in the checklist). When a docs page or schema shows an example request or response, explicitly request it be quoted character-for-character.

## Step 2 — identify the model and find its Runware equivalent

From the pasted call, extract the WaveSpeed model ID (e.g. `wavespeed-ai/flux-dev`). Check `references/model-map.md` for a known pairing to a Runware AIR ID. WaveSpeed splits many models by operation (text-to-image vs. edit, text-to-video vs. image-to-video) at the model-ID level more aggressively than Fal or Replicate do — match on the specific operation, not just the model family. If the model ID isn't listed, search the live Runware catalog (`index.json` above) for a model from the same creator/family, and say explicitly that the pairing is inferred, not verified.

If there's no reasonable Runware equivalent at all, say so plainly rather than picking the closest thing and presenting it as equivalent.

## Step 3 — run the structural diff

Compare the two calls across these dimensions. For each, categorize as **maps cleanly**, **needs adjustment**, or **no equivalent** — see `references/mapping-checklist.md` for the full checklist and worked example.

- **Auth**: WaveSpeed uses `Authorization: Bearer <WAVESPEED_KEY>` — same scheme keyword as Runware. Only the token value changes.
- **Endpoint shape**: WaveSpeed calls a per-model URL, e.g. `POST https://api.wavespeed.ai/api/v3/{model_id}`. Runware calls one fixed endpoint with the model named inside the body via its AIR ID.
- **Request body**: a bare object matching that specific model's own schema — not wrapped in an `"input"` key like Replicate, and not array-wrapped like Runware's request. Fields go directly onto Runware's task object instead.
- **A genuinely distinctive WaveSpeed quirk — dimension format**: at least one model's real, live-fetched schema (`wavespeed-ai/flux-dev`) uses a `size` field formatted as an asterisk-delimited string, e.g. `"1024*1024"` — not `{width, height}` like Fal, not separate `width`/`height` integers like Runware. This is easy to get wrong silently (splitting on `x` instead of `*`, or assuming it's already an object). Check the specific model's schema rather than assuming this format holds for every WaveSpeed model.
- **Output format naming**: WaveSpeed's `output_format` values are lowercase (`jpeg`/`png`/`webp`). Runware's `outputFormat` enum is documented as uppercase (`JPG`/`PNG`/`WEBP`) — use that casing. Direct testing shows the API also accepts lowercase without erroring, but that's undocumented leniency, not the spec; write the uppercase values rather than relying on it.
- **Sync hint, not a guarantee**: some WaveSpeed models expose an `enable_sync_mode` boolean. Per that field's own schema description, even when set to `true`, "the API can return a timeout body while the task continues processing" — the same non-guarantee pattern as Replicate's `Prefer: wait` header. Don't assume this makes a call deterministically synchronous; keep a poll fallback. (This flag wasn't exercised directly — the confirmed example in `references/worked-example.md` used the default, async-style flow.)
- **Seed convention**: at least one WaveSpeed schema uses `-1` as an explicit sentinel meaning "random seed", rather than omitting the field. Runware doesn't use a sentinel at all — confirmed via the live model schema, `seed` is a plain integer with minimum `0` (so `-1` isn't a valid value there), no default, and its own description states that omitting the field generates a random seed. Drop the field entirely for "random" instead of translating WaveSpeed's `-1` across.
- **Async / submit-and-poll pattern**: confirmed for `wavespeed-ai/flux-dev` (see `references/worked-example.md`) — submit returns a `data.urls.get` poll URL directly; polling that URL returns `status` progressing through `created` → `processing` → `completed`, with the output appearing in `data.outputs` once done. This is a real, observed pattern, not a guess — but it's confirmed for one model, so verify the same shape holds for whichever model the developer is actually migrating, especially for error/failure states, which haven't been directly observed.
- **Response envelope**: confirmed — `{"code", "message", "data": {...}}` applies to the catalog, pricing, and generation endpoints alike. Note `data` here is a single object (not an array like Runware's `data`), and the envelope has no separate top-level `errors` array the way Runware's does — WaveSpeed folds error state into `data.status` and `data.error` instead.
- **Cost / pricing — WaveSpeed's strongest point of the three competitors**: WaveSpeed exposes a dedicated `POST /api/v3/model/pricing` endpoint that takes `{model_id, inputs}` and returns an exact computed price (`{data: {unit_price, currency}}`), driven by a per-model formula (e.g. `base_price * num_images`, or duration-scaled formulas for video). This is a genuine pre-flight cost-prediction capability neither Fal nor Replicate have. Runware's equivalent is `cost` returned inline in the response (`includeCost: true`) — correct, but known only after the call, not before it. If a developer's code calls WaveSpeed's pricing endpoint before submitting to show a cost estimate to a user, that specific pre-flight call has no Runware equivalent to migrate — Runware doesn't need one, since you can no longer estimate before calling the same way, only read the real cost back afterward. Say this plainly rather than inventing a pre-flight cost endpoint that doesn't exist on Runware.
- **Model addressing**: WaveSpeed model ID (e.g. `wavespeed-ai/flux-dev`) → Runware AIR (`creator:family@version`). Look up via `model-map.md` or the live catalog, don't guess.

## Step 4 — write the report

Structure your answer as three buckets, each with the specific field/behavior and a one-line reason:

- **Maps cleanly** — same concept, different name/format only, low risk.
- **Needs adjustment** — real behavioral or shape difference; give the concrete before/after.
- **No equivalent** — genuinely doesn't exist on the other side; say what to do instead (usually: delete the code, or note a workaround with its tradeoff).

If the model in question hasn't itself been confirmed by a real call (only `wavespeed-ai/flux-dev` has, via `references/worked-example.md`), say so and recommend a real test call before shipping — the general pattern is confirmed, but per-model specifics can still vary.

## What this skill deliberately does not do

- It does not auto-generate a finished, drop-in-replacement Runware client for you — the output is an analysis to act on, not a diff you should apply blindly.
- It does not claim every WaveSpeed model has been individually verified — the model-ID mapping is catalog-matched, and only one model's full generation flow has been confirmed end to end (see `references/worked-example.md`).
- It does not claim access to knowledge about common integration mistakes. Everything here comes from WaveSpeed's public catalog/schema data and Runware's public docs.

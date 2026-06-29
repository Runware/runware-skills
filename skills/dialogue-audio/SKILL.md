---
name: dialogue-audio
description: >
  Generate a two-speaker conversation as a single audio file. Use when the user says "make a
  podcast snippet", "two people talking", "a back-and-forth dialogue", "narrator and character",
  "interview audio", "an explainer with two voices", or wants speakers trading turns in one clip.
  One request, two voices, natural turn-taking. For a single narrator, use voiceover. To create or
  clone the voices themselves, use voice-cloning.
---

# Dialogue audio

Produce a multi-speaker conversation as one audio file: a podcast snippet, character dialogue, an interview, or an explainer back-and-forth. The lever is inline `<|speaker:N|>` tags in a single text input mapped to a `speech.voices` array, so both sides render in one call with automatic turn-taking. For one voice reading straight through, that is `voiceover`, not this.

## Inputs to collect

- **The script**, written as alternating turns for two speakers. (Ask for it if the user only gave a topic.)
- **The two voices** (voice model IDs) and which speaker is index `0` vs `1`. If none specified, pick two that contrast in pitch/cadence.
- **Per-turn emotion**, if any (excited, surprised, whispering). Optional, tag only the turns that need it.
- Optional: language (the model auto-detects and supports 80+) and output format.

## Models

- **Default: Fish Audio S2.1 Pro** (`fishaudio:s2.1@pro`) - the one model here that renders two speakers in a single `audioInference` call via inline speaker tags, with per-speaker emotion control. Capability `io:text-to-audio`, status `live`.
- This is a model-specific feature, not a generic TTS one. Other text-to-audio models do one voice per request and would need separate calls plus downstream stitching.
- Confirm the live model + its schema via the `runware-models` + `runware-run` skills before calling - never hardcode a stale choice.

## Workflow

1. Resolve the model schema (`runware-run`) and confirm `speech.text` and `speech.voices` (array) on the live schema.
2. Write the full conversation into one `speech.text` string, opening with `<|speaker:0|>` and switching voices with `<|speaker:N|>` at each turn.
3. Map voices: `speech.voices[0]` is `<|speaker:0|>`, `speech.voices[1]` is `<|speaker:1|>`. Use `speech.voices` (array), not `speech.voice` (singular) - they are mutually exclusive and sending both errors.
4. Run `audioInference` asynchronously and poll `getResponse` until terminal (audio is a time-based task, don't block a sync call).
5. Read the result from `audioURL` and check turn separation and emotion delivery.

## Technique

- **Tag at turn boundaries.** `<|speaker:N|>` switches the voice for everything that follows until the next tag. Place tags at the start of a new sentence or thought, never mid-clause, or the voice switch sounds unnatural.
- **Open with a tag.** The model defaults to speaker 0 if the text starts untagged. Lead with `<|speaker:0|>` explicitly to remove ambiguity.
- **Exactly two speakers.** Indices `0` and `1` only. The `speech.voices` array assigns one voice ID to each. There is no third speaker in a single call.
- **Emotion is per-speaker.** Bracket tags like `[excited]` or `[whispering]` apply to the current speaker only and do not bleed across the turn boundary. Tag only the speaker who needs it, the contrast makes the emotion read. Speaker tags and emotion tags are independent: one sets who talks, the other sets how. The full emotion catalog is a sibling concern (`emotion-and-expression` in the model guides).
- **Turn-taking is automatic.** The model paces pauses between speakers and adjusts pacing per speaker, so asymmetric turn lengths (a short question, a long answer) work without manual timing.
- **Patterns:** technical back-and-forth (no emotion tags when content carries it), podcast (longer multi-sentence turns), interview (short ask, long answer), narrator + character (one third-person voice, one first-person voice with emotion tags).

## Parameters that matter

- `speech.text` - the whole conversation in one string. Line breaks are optional (`\n` in JSON); the `<|speaker:N|>` tags are the only markers needed.
- `speech.voices` - array of two voice model IDs, index-mapped to the speaker tags. Mutually exclusive with `speech.voice`.
- `<|speaker:N|>` - `N` is `0` or `1` only; zero-based and matched positionally to `speech.voices`.
- `[emotion]` - inline bracket tag scoped to the current speaker; optional per turn.
- Confirm exact field names against the live schema (`runware-run`); never guess.

## Quality bar

- Two voices are clearly distinct; turns are correctly attributed (speaker 0 vs 1 not swapped).
- Voices contrast enough that the conversation does not sound like one person. Pick a lower-register and a higher-register voice if separation is weak.
- Turns are long enough (aim one full sentence each) to establish each voice; two-word turns blur the speakers.
- Emotion lands on the intended speaker only, with no bleed into the other's turn. Retry with clearer tags or contrasting voices if speakers merge.

## Related skills

`runware-run`, `runware-models`, `runware-prompting`; `voiceover` (single-voice narration), `voice-cloning` (custom voice IDs to drop into `speech.voices`).

---
name: music
description: >
  Generate music from a text description - full songs with vocals and lyrics,
  instrumental beds, jingles, intros, and background tracks. Use when the user
  says "write me a song", "make a beat", "background music for my video", "a
  jingle for my brand", "an instrumental in the style of lo-fi hip hop", or
  wants to restyle/cover an existing track in a new genre.
---

# Music

Produce original music from a text prompt: full vocal songs with your lyrics, instrumental beds, short jingles, or background tracks. You supply a style/genre description (and lyrics when you want vocals), the model composes and renders the audio. A separate cover variant restyles an existing track instead of writing one from scratch.

## Inputs to collect

- **Style / genre.** A short production-style description: genre, mood, tempo feel, instrumentation, era. This is required for every model.
- **Vocals or instrumental.** Whether the track should sing. If vocals, collect the **lyrics** (or ask the model to generate them from the prompt).
- **Length / use.** Full song, a 30-second bed, a short jingle. Affects model choice and `duration`.
- **For a cover/restyle:** the source audio (URL or UUID) and the target style. (Ask only if restyling.)
- Optional: BPM, musical key, vocal language, a seed for repeatability.

## Models

- **Full vocal song, promptable production:** **MiniMax Music 2.6** (`minimax:music@2.6`) - natural-language or production-style prompts, reliable BPM/key following, lyrics or instrumental via settings. Strong default for finished songs.
- **Open, controllable songs + longer tracks:** **ACE-Step v1.5 Base** (`runware:ace-step@v1.5-base`) - vocals, explicit lyrics, BPM, key, vocal language, and `duration` up to 5 minutes. Best when you need fine control or instrumental beds for video.
- **Fast iteration:** **ACE-Step v1.5 XL Turbo** (`runware:ace-step@v1.5-xl-turbo`) - accelerated 8-step inference for rapid drafts at higher capacity. Draft with this, finalize on Base or MiniMax.
- **Cover / restyle an existing track:** **MiniMax Music Cover** (`minimax:music@cover`) - keeps the original vocal melody, changes timbre, instrumentation, and genre from a text prompt. This is `io:audio-to-audio`, not text-to-music.

Confirm the live model and its schema via the `runware-models` + `runware-run` skills before calling. Models change weekly, do not hardcode a stale choice.

## Workflow

1. Resolve the model schema (`runware-run`) and confirm the field names and `duration` limits for the model you picked.
2. For text-to-music: send `taskType: 'audioInference'` with `positivePrompt` (the style) and, for vocals, `settings.lyrics`.
3. For a cover: upload the source track to `inputs.audio` and give the target style in `positivePrompt` (MiniMax Cover requires source audio between 6 seconds and 6 minutes).
4. **Run asynchronously and poll.** Audio is a time-based task. The call returns a `taskUUID`, poll `getResponse` until terminal. Don't block a sync call.
5. Read the result from the audio URL in the response. Listen back, then iterate on prompt, lyrics, BPM, or seed.

## Technique

- **Describe production, not vibes alone.** Name genre, instrumentation, tempo feel, and era ("upbeat synth-pop, 120 BPM, bright analog pads, 80s"). The more concrete the brief, the more controllable the result.
- **Lyrics drive the vocal.** On ACE-Step and MiniMax, pass the actual lyrics in `settings.lyrics`, formatted like a lyrics page. ACE-Step also reads structure tags (verse/chorus). Leave lyrics empty and the model improvises or stays instrumental.
- **Instrumental beds:** on MiniMax set `settings.instrumental: true` (this excludes lyrics, the two are mutually exclusive). On ACE-Step, omit lyrics and set `vocalLanguage: 'unknown'`. These beds pair well with `add-audio-to-video` for scoring a clip.
- **Lock musical feel** with `bpm` and `keyScale` (ACE-Step) or BPM/key phrasing in the MiniMax prompt. Fix `seed` to reproduce a take, vary it for alternates.
- **Jingles and intros:** short style prompt, a tight one or two line lyric, set a low `duration` on ACE-Step. Keep the hook simple so it reads in a few seconds.
- **Cover/restyle:** the source fixes melody and timing, the prompt directs only the new style and arrangement. Optionally edit lyrics via `settings.lyrics`. Don't re-describe the original song.
- Load `references/examples.md` for full worked requests (ACE-Step vocal song with BPM/key, MiniMax song, MiniMax instrumental bed).

## Parameters that matter

- `positivePrompt` - required everywhere, the style/genre brief. (ACE-Step up to 3000 chars, MiniMax 2.6 up to 2000, Cover 10-300 chars.)
- `settings.lyrics` - the vocal lyrics. ACE-Step 10-3000 chars, MiniMax 1-3500. Mutually exclusive with `instrumental: true` on MiniMax.
- `settings.instrumental` (MiniMax) - `true` for a vocal-free bed.
- `duration` (ACE-Step) - seconds, 30 to 300, default 60, step 0.1. Omitted/ignored when `inputs.audio` is supplied. MiniMax has no duration field, it returns a full song.
- `settings.bpm` (ACE-Step) - integer 30-300, auto if unset. `settings.keyScale` - `"C major"`, `"F# minor"`, etc.
- `settings.vocalLanguage` (ACE-Step) - ISO 639-1 code, `'unknown'` for instrumental/auto, default `en`.
- `inputs.audio` (Cover, and ACE-Step cover/repaint) - source track URL or UUID. Cover requires 6s-6min.
- `seed` - fix for repeatability, vary for alternates.
- Confirm exact field names and ranges against the live schema (`runware-run`), never guess.

## Quality bar

- The track matches the requested genre, tempo, and mood, and the vocal sings the supplied lyrics clearly (or is cleanly instrumental when asked).
- Length fits the use (full song vs short bed vs jingle), no abrupt cut-off mid-phrase.
- For a cover, the original melody is recognizable and only style/arrangement changed.
- Rights: do not imitate an identifiable artist, band, or specific existing song. Describe a style, not a named performer, and keep lyrics original.

## Related skills

`runware-run`, `runware-models`, `runware-prompting`; `add-audio-to-video` (score a video with a generated bed).

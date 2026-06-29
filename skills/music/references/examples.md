# Music worked recipes

Three end-to-end recipes for the `music` skill. Each shows the scenario, the real `audioInference` request, and the result shape. Adapt the style, lyrics, and settings. Confirm field names and ranges against the live schema (`runware-run`) before sending. Music is a time-based task, so run every request asynchronously, then poll `getResponse` until terminal and read `audioURL`.

Field reminders verified against the live request schemas:

- ACE-Step (`runware:ace-step@v1.5-base`) exposes structured controls: `duration` (seconds, 30 to 300, default 60, step 0.1), `settings.bpm` (integer 30 to 300), `settings.keyScale` (`"C major"`, `"F# minor"`, `"Bb major"`), `settings.timeSignature` (2, 3, 4, or 6), `settings.vocalLanguage` (ISO 639-1, or `unknown`), and `settings.lyrics` (10 to 3000 chars). The 300-second `duration` is the hard cap. Asking for a longer track fails validation. `duration` is ignored when you pass `inputs.audio`.
- MiniMax Music 2.6 (`minimax:music@2.6`) has no `duration`, `bpm`, or `keyScale` fields. It returns a full song and reads tempo and key from the prompt text. Control vocals with `settings.lyrics` (section tags like `[Intro]`, `[Verse]`, `[Chorus]`) or set `settings.instrumental: true`. The two are mutually exclusive. A request with both fails validation.
- Response is `audioInference` returning `audioUUID` and `audioURL`.
- Rights: describe a style, not a named artist, band, or specific existing song. Keep lyrics original. Do not imitate an identifiable performer.

## Recipe 1: Full vocal song with lyrics and explicit musical controls (ACE-Step v1.5 Base)

Scenario: an upbeat synth-pop song with a verse and a chorus. You want a fixed tempo and key, English vocals, and a 90-second length. ACE-Step takes the lyrics in `settings.lyrics` and reads `[verse]` / `[chorus]` structure tags. Lock `bpm` and `keyScale` so the take is reproducible alongside the `seed`.

Request:

```json
[
  {
    "taskType": "audioInference",
    "model": "runware:ace-step@v1.5-base",
    "positivePrompt": "upbeat synth-pop, bright analog pads, punchy four-on-the-floor drums, warm bass, 80s production, polished and energetic",
    "duration": 90,
    "seed": 42,
    "settings": {
      "bpm": 120,
      "keyScale": "C major",
      "vocalLanguage": "en",
      "lyrics": "[verse]\nCity lights are calling out my name\nNeon rivers running through the rain\nEvery heartbeat keeps a steady time\nFeel the rhythm, leave it all behind\n\n[chorus]\nWe are running through the night\nChasing every burning light\nHold on tight and don't let go\nThis is all we'll ever know"
    }
  }
]
```

Notes:

- `settings.lyrics` is the lyrics-vs-instrumental gate on ACE-Step. Pass lyrics and the model sings them. For an instrumental on ACE-Step instead, omit `lyrics` and set `vocalLanguage: "unknown"` (see the MiniMax bed in Recipe 3 for the alternate route).
- Format lyrics like a lyrics page. `[verse]` and `[chorus]` tags steer the song structure. Lyrics run 10 to 3000 chars.
- `duration` caps at 300 seconds (5 minutes). A larger value fails validation. Default is 60.
- `bpm` (30 to 300), `keyScale` (`"{Note}{Accidental} {Mode}"`), and `seed` together pin a reproducible take. Vary `seed` for alternates on the same brief.

Result:

```json
{
  "data": [
    {
      "taskType": "audioInference",
      "taskUUID": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "audioUUID": "f1e2d3c4-b5a6-7890-1234-567890abcdef",
      "audioURL": "https://am.runware.ai/audio/os/a14d18/ws/2/ai/f1e2d3c4-b5a6-7890-1234-567890abcdef.mp3",
      "seed": 42
    }
  ]
}
```

## Recipe 2: Full vocal song via MiniMax Music 2.6 (tempo and key in the prompt)

Scenario: a finished pop ballad with vocals. MiniMax has no `bpm` or `keyScale` field, so the tempo and key go in `positivePrompt`. Structure the lyrics with section tags. Leave `instrumental` unset, which keeps vocals on.

Request:

```json
[
  {
    "taskType": "audioInference",
    "model": "minimax:music@2.6",
    "positivePrompt": "emotional pop ballad, slow tempo around 70 BPM in A minor, intimate piano and strings, soft female lead vocal, cinematic build into the chorus",
    "settings": {
      "lyrics": "[Verse]\nQuiet morning, empty room\nShadows fading way too soon\nEvery word I left unsaid\nStill is echoing in my head\n\n[Chorus]\nAnd I would give it all to stay\nWatch the night turn into day\nHolding on to what we knew\nEvery road leads back to you"
    }
  }
]
```

Notes:

- MiniMax reads tempo and key from the prompt. Write "around 70 BPM in A minor" in `positivePrompt`, there is no separate `bpm` or `keyScale` field on this model.
- `settings.lyrics` uses section tags `[Intro]`, `[Verse]`, `[Chorus]`, `[Bridge]`, `[Outro]`, `[Inst]`. Lyrics run 1 to 3500 chars on MiniMax.
- Do not also set `settings.instrumental: true` here. Lyrics and `instrumental: true` are mutually exclusive and the request fails validation. MiniMax returns a full song, there is no `duration` field.
- `positivePrompt` runs up to 2000 chars on MiniMax.

Result:

```json
{
  "data": [
    {
      "taskType": "audioInference",
      "taskUUID": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
      "audioUUID": "e2d3c4b5-a6f7-8901-2345-67890abcdef1",
      "audioURL": "https://am.runware.ai/audio/os/a14d18/ws/2/ai/e2d3c4b5-a6f7-8901-2345-67890abcdef1.mp3"
    }
  ]
}
```

## Recipe 3: Instrumental bed for a video (MiniMax Music 2.6, instrumental true)

Scenario: a vocal-free background bed to score a product clip. Set `settings.instrumental: true` and describe the production fully in the prompt. No lyrics, since the two are mutually exclusive. Pair the result with `add-audio-to-video` to lay it under the footage.

Request:

```json
[
  {
    "taskType": "audioInference",
    "model": "minimax:music@2.6",
    "positivePrompt": "lo-fi hip hop instrumental, mellow and relaxed around 85 BPM, warm vinyl crackle, soft Rhodes piano, laid-back boom-bap drums, gentle bassline, no vocals",
    "settings": {
      "instrumental": true
    }
  }
]
```

Notes:

- `settings.instrumental: true` produces a bed with no vocals. Do not pass `settings.lyrics` alongside it, the request fails validation if you do.
- All the musical direction lives in `positivePrompt`. Name genre, mood, tempo feel, and instrumentation.
- For an ACE-Step instrumental instead, omit `settings.lyrics` and set `settings.vocalLanguage: "unknown"`, then set `duration` for the exact length you need (up to 300 seconds).

Result:

```json
{
  "data": [
    {
      "taskType": "audioInference",
      "taskUUID": "c3d4e5f6-a7b8-9012-cdef-123456789012",
      "audioUUID": "d3c4b5a6-f7e8-9012-3456-7890abcdef12",
      "audioURL": "https://am.runware.ai/audio/os/a14d18/ws/2/ai/d3c4b5a6-f7e8-9012-3456-7890abcdef12.mp3"
    }
  ]
}
```

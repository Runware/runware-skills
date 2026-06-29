# Product photography: worked recipes

Three end-to-end recipes. Each shows the scenario, the real request (taskType, model AIR, load-bearing params with concrete values), and the result shape. Confirm every model is `live` and re-resolve its schema via `runware-models` + `runware-run` before sending. Field names and dimension constraints below are verified against the live schemas at time of writing, but re-check rather than trust this file blind.

A request body is the array of task objects you pass to the API. Generate a fresh `taskUUID` per task. The result shape blocks below are abbreviated to the load-bearing fields.

---

## Recipe 1: Clean packshot on pure white (generate then cut)

**Scenario.** The user has no usable shot and wants a catalog-ready bottle on a seamless white background, transparent PNG for the storefront.

Two stages: generate a clean studio shot, then cut the background to alpha. A white-sweep generation plus a cutout beats trying to key out a busy scene.

**Stage 1, generate (FLUX.2 [max], `bfl:7@1`).** Brief it like a photographer: subject, surface, lighting, camera. Keep the background a plain white sweep so the cutout in stage 2 is clean.

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "a1111111-1111-1111-1111-111111111111",
    "model": "bfl:7@1",
    "positivePrompt": "A frosted glass serum bottle with a matte black pump, standing centered on a seamless white studio sweep, soft large softbox from the upper left, gentle fill from the right, clean reflection on the surface, shot at 100mm, product hero shot, premium skincare aesthetic",
    "width": 1024,
    "height": 1024
  }
]
```

Result shape (`imageInference`):

```json
{ "taskType": "imageInference", "imageUUID": "...", "imageURL": "https://im.runware.ai/image/ws/.../<uuid>.jpg", "seed": 123456 }
```

Carry `imageUUID` (or `imageURL`) into stage 2 as the input.

**Stage 2, cut to alpha (BiRefNet General, `runware:112@5`).** TaskType is `removeBackground`, not `imageInference`. The source goes in `inputs.image`. Request a PNG so the alpha channel survives.

```json
[
  {
    "taskType": "removeBackground",
    "taskUUID": "a2222222-2222-2222-2222-222222222222",
    "model": "runware:112@5",
    "inputs": { "image": "<imageUUID from stage 1>" },
    "outputFormat": "PNG"
  }
]
```

Result shape (`removeBackground`):

```json
{ "taskType": "removeBackground", "imageUUID": "...", "imageURL": "https://im.runware.ai/image/ws/.../<uuid>.png" }
```

The output PNG is the subject on transparency. For a hard white instead of transparent, composite it onto white downstream or skip the cutout and trust the white sweep. Pick the BiRefNet variant by subject: `runware:112@5` General for solid products, `runware:112@9` Matting for fine edges (hair, fur, mesh, translucent caps).

---

## Recipe 2: Lifestyle scene with grounded shadow (generate then relight)

**Scenario.** The user has a clean packshot of a sneaker and wants it sitting on a sunlit wooden bench in a real outdoor setting, looking photographed, not pasted.

Two stages: place the product in the new scene, then relight so the contact shadow and light direction match. FIBO Edit preserves the product's structure while changing its surroundings, which keeps the shoe on-brand.

**Stage 1, replace the background in place (Bria FIBO Edit, `bria:21@1`).** TaskType is `imageInference`. The product goes in `inputs.image`. The instruction is the `positivePrompt`. `CFGScale` is restricted to `3`, `4`, or `5`. `steps` runs `20` to `50`.

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "b1111111-1111-1111-1111-111111111111",
    "model": "bria:21@1",
    "inputs": { "image": "<the user's packshot UUID or URL>" },
    "positivePrompt": "Place the sneaker on a weathered wooden park bench outdoors, warm late afternoon sunlight from the right, soft green foliage blurred in the background, keep the shoe's shape, colors, and materials exactly as in the input",
    "CFGScale": 5,
    "steps": 40
  }
]
```

**Stage 2, match the shadow (Bria FIBO Edit again).** If the first pass left the shoe floating or the shadow pointing the wrong way, relight rather than re-rolling. Same model, an instruction that names the light direction and the contact shadow.

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "b2222222-2222-2222-2222-222222222222",
    "model": "bria:21@1",
    "inputs": { "image": "<imageUUID from stage 1>" },
    "positivePrompt": "Add a soft contact shadow under the sneaker falling to the left, consistent with warm sunlight coming from the right, ground it firmly on the bench surface, keep the shoe unchanged",
    "CFGScale": 4,
    "steps": 40
  }
]
```

Result shape (both stages, `imageInference`):

```json
{ "taskType": "imageInference", "imageUUID": "...", "imageURL": "https://im.runware.ai/image/ws/.../<uuid>.jpg", "seed": 123456 }
```

The giveaway of a composite is a missing or wrong-direction contact shadow. The relight step is what grounds the product. To target a region precisely, add `inputs.mask` (a mask image) so the edit only touches what you intend.

---

## Recipe 3: Palette-locked brand shot (Recraft, colors + background)

**Scenario.** A sports-drink can mockup for a brand whose guidelines specify an exact red and white, on a deep navy canvas. The output must hit the brand RGB, not approximate it.

One stage. Recraft V4.1 (`recraft:v4.1@0`) reads `settings.colors` as a hard palette constraint and `settings.backgroundColor` as the canvas behind the subject. Pass the exact RGB integers (0-255) from brand guidelines. Each color is `{ "rgb": [r, g, b] }`. Use 2-4 colors for a tight lock. Background color works best on simple, isolated-subject compositions, which is the product-shot case.

Recraft V4.1 only accepts a fixed set of dimension pairs. `1024x1024` (1:1), `1536x768` (2:1), `1280x832` (3:2), `1344x768` (16:9), `768x1344` (9:16), and a handful more. Confirm the allowed pair against the live schema before sending an arbitrary `width`/`height`.

```json
[
  {
    "taskType": "imageInference",
    "taskUUID": "c1111111-1111-1111-1111-111111111111",
    "model": "recraft:v4.1@0",
    "positivePrompt": "A sports drink aluminum can mockup, bold dynamic swoosh graphics, condensation droplets on the surface, studio lighting, centered product photography",
    "width": 1024,
    "height": 1024,
    "settings": {
      "colors": [
        { "rgb": [200, 30, 30] },
        { "rgb": [255, 255, 255] }
      ],
      "backgroundColor": { "rgb": [18, 22, 42] }
    }
  }
]
```

Result shape (`imageInference`):

```json
{ "taskType": "imageInference", "imageUUID": "...", "imageURL": "https://im.runware.ai/image/ws/.../<uuid>.jpg", "seed": 123456 }
```

To produce the same concept in multiple colorways, hold the `positivePrompt` and swap only `settings.colors`. For a flat, front-facing catalog asset rather than a creative interpretation, route to a Recraft Utility variant and keep the same `settings.colors`. The palette parameters work across the whole Recraft family.

---

## Cost note

A multi-stage pipeline multiplies per-image cost. Before a batch, set `includeCost` or dry-run a single image via `runware-run` to confirm the price, since recipe 1 (generate plus cut) and recipe 2 (place plus relight) each bill two stages per output.

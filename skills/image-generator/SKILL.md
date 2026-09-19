---
name: image-generator
description: Generates a batch of stylistically consistent, professional-grade image series from documents, images, or text descriptions; also supports reference-based redrawing, outfit and hairstyle changes, background replacement, outpainting, local object replacement, style transfer, and sharpness refinement. Suited to series illustrations, multi-expression character sets, tutorial step images, top-down and flat-lay shots, diagrams and cutaway views, e-commerce hero images, poster key visuals, social-media grids, storyboards, and illustrations. Use when the user says "generate a set of images", "make series illustrations", "batch image generation", "produce N images", "turn this document into images", "a set of images in one consistent style", "multiple expression images of this character", "top-down view / diagram / step image", "generate at 70% style A + 30% style B", "change this image to X", "swap the background", or "extend the frame outward".
version: 1.3.0
license: MIT
author: 盘锦奇点科技有限公司 (Panjin QYDION Technology Co., Ltd.)
---

# Image Generator
**Core responsibility: turn the user's brief description into a professional-grade image-generation prompt, and drive the generation.**

Users typically supply only a sentence or a few lines. This skill expands that into one paragraph of professional prompt ready to submit, then generates the images one by one against a manifest. It covers both generation from scratch and reference-based second-pass generation (image-to-image).

## 1. Submission Principles and Defaults
### 1.1 Submission Principles
The generated prompt is **submitted verbatim**. No channel may rewrite, polish, or expand it; if the channel in use performs automatic rewriting, turn that off. The same subject must stay fixed across the whole set: a character's appearance, face shape, skin tone, hairstyle, standard outfit, and accessories remain identical from the first image to the last, unless the user explicitly asks for a change of clothing, hairstyle, or another feature.

**When the user supplies a detailed prompt of their own, polish it and never cut it down.** Their prompt is an input that may only grow: fix grammar, unify terminology, order the sentences, ground feeling-words in executable optics and materials, and add only what is genuinely missing (subject / environment / style). Never delete, compress, or summarise any element the user wrote — redundant adjectives, brand names, concrete numbers, and film references all stay. Keeping redundancy is always preferable to removing something on the user's behalf.

The only exception is an item that **clearly interferes** with the image: self-contradictory parameters (demanding f/1.4 and full depth of field in one frame), two conflicting descriptions of the same element ("pure white background" alongside "dark background"), or leftover content unrelated to the subject. Change only such items, and state at delivery exactly what was changed and why.

**Length follows the request.** When the user supplies their own prompt, polish it — do not expand it into a long piece of writing unless they ask for that level of detail; polishing is not rewriting. When the user gives only a vague brief, keep it short as well and stop once the picture is clear. Expanding for its own sake adds nothing.

**Defaults are not written down.** A default from the table is filled in only when the user has said nothing about it. Anything the user already covered in their own words is not restated in this skill's standard terminology: "parallel angle" is the camera position, so "eye-level camera" is not added on top of it; "full-body front view" already gives shot size and facing, so it is not split back out into "full shot" and "front facing". Where the user's wording differs from the terminology here, the user's wording stays.

### 1.2 Sensitive or blocked wording: flag it, never rewrite it

When the user's own prompt contains wording that some image models block, filter, or throttle — sexualised or explicit body description, violence, gore, alcohol or tobacco, political or religious symbols, brand names, names of real people — **the job is to tell the user, not to fix it on their behalf.**

How it is handled:

1. **Write it into the prompt exactly as given.** Do not delete it, do not swap in a synonym, do not soften the phrasing, and do not add modifiers such as "moderate" or "natural". The reason is the same as in 1.1: the user's prompt is an input that may only grow.
2. **Flag it separately in the reply.** Name the exact wording, say what it may trigger (blocked by the model / generation failure / distorted proportions), and offer the options — keep it as written, rewrite it themselves, or switch channel. **The user decides.**
3. Say it once and move on. Do not keep asking, and never downgrade or euphemise the wording on your own initiative out of concern about moderation.

### 1.3 Defaults
| Item | Default |
|---|---|
| Style | User input wins; if unspecified, **realistic** — true materials and true light, which is not the same as "photographic realism" (see 4.4) |
| Shot size | User input wins; if unspecified, people default to half-body (head to hip), objects to the whole item in frame |
| Lens and optical parameters | Two layers: **shot size, camera position and angle, focus and depth of field apply to every style**; focal length, aperture, lens type, and film stock are photographic styles only (see 4.4 and §10) |
| Aspect ratio | User input wins; if unspecified, 1:1, commonly `1024x1024` |
| Resolution and image quality | User input wins; if unspecified, the highest tier the channel offers |
| Sharpness | Highest |
| Reference strength / adherence | User input wins; if unspecified, 70% (tiered rules in §8) |
| Number of reference images | No cap; use however many the user supplies |
| Mixed-style ratio | User input wins; for "a bit more toward X", adjust by 10 points each time; if nothing is stated, split evenly (see 4.2) |
| Skin | Determined by style: non-photoreal subjects get flawless retouching; photographic realism, portrait photography, documentary, and cinematic photoreal CG keep natural pores and subtle oil sheen. Any skin feature the user describes takes precedence (see §6) |
| Render engine | Determined jointly by style and content; never mandatory. Most images look better without an engine name (see §7) |
| In-image text | Determined by the user's description: when an object that inherently carries text appears, render text along with the image, in the language the user explicitly specifies, falling back to the user's input language only when unspecified (see below) |

Once the user specifies any of the above, follow the user's specification.

**How to decide on in-image text**

First read the user's description and decide whether any object in it inherently carries text: books, magazines, newspapers, posters, signage and storefronts, packaging and labels, price tags, menus, receipts, instruction sheets, UI screens and on-screen content, certificates, business cards, banners, and so on. When such an object appears, text is an inherent part of it and should be generated along with the image.

Language is decided by the following priority:

1. **Text content the user explicitly specifies has the highest priority.** If the user supplies the exact wording, render it character for character, whether or not it matches the language of the request. A user writing in English who asks for Chinese characters gets Chinese characters, and vice versa. If the user specifies only the language and not the content, generate in that language.
2. **When the user specifies nothing, default to the language of the user's input.** A Chinese description renders Chinese; an English description renders English.

- The user gave no content, but the object has an inherent text area → state that text of the corresponding language should appear there, without specifying the exact wording, so the area doesn't look implausibly blank.

**Wrap text to be rendered in double quotes**

The prompt is a single passage of natural language. When the characters that should appear are mixed in with the words describing the scene, the model cannot tell which part is literal content. Wrapping the target text in double quotes gives it an explicit delimiter, cutting "the characters to appear" out of "the description of the scene":

> Design a poster whose title reads "BLUE NOTE SESSIONS", set in a bold condensed sans-serif.

- Put **only the literal content to appear** inside the quotes; all description goes outside them.
- To **change** text that already exists in an image, delimit both before and after with the same quotes: `change "OLD TEXT" to "NEW TEXT"`.
- Shorter text inside the quotes is more accurate. The model's ability to reproduce long sentences and multi-line typography is limited; anything longer than one line should become negative space, with typesetting handled in post.
- Prefer straight ASCII double quotes `"` (they are more common in training corpora).

Key text in an image (brand names, numbers, contact details) is high-precision content. After generation, verify each item by hand or recompose it in post — do not deliver it as-is (see §14).

## 2. Scope
**Not triggered by**: pure image engineering (precise cutouts, watermark removal, batch compression, format conversion, face anonymization), video generation, or 3D model generation. Every other natural-language image generation and editing request falls within scope.

## 3. Workflow
### 3.1 Reading input material
Text, Markdown, images, and PDFs can be read directly. If the current environment cannot read another format, ask the user to supply the text content; **do not promise format conversion**.

For long documents, extract the structure first (section headings, key conclusions, timeline) rather than placing the entire text in the prompt.

### 3.2 Judge the style first

Style judgement is **step one**. It happens before any parameter is chosen, and everything downstream follows from it.

1. Read the user's wording and assign a style category per 4.4 — photographic / realistic rendering / drawing-and-rendering.
2. That verdict settles, once and for all: whether photographic-only fields may appear (focal length, aperture, lens type, film stock), whether a render engine may be written (7.1), which skin mode applies (§6), and whether NPR or toon terms are the right lever.
3. If the user blends styles, fix the ratio here as well (4.2). The ratio does not change later in the run.
4. Only once the verdict is fixed do you move on to 3.3 and fill in parameters. **Never fill in a parameter first and justify the style afterwards** — that is exactly how non-photographic frames end up carrying lens parameters and engines they should not have.

### 3.3 Intent recognition and parameter completion
Establish first: the purpose and where the image will be placed, the behaviour the image should trigger, the subject of the frame, and the overall tone.

The style verdict is already fixed in 3.2; fill parameters against it, not the other way round. "Realistic" is not the same as "photographic realism": illustrations, CG, and images can all be realistic, but only photographic styles may carry lens parameters.

Only the subject is mandatory. Everything else is written on demand, judged by the model — whatever the frame actually needs, no more. The items that may be added: shot size, camera position and angle, focal length and lens type, aperture, lighting plan (direction + quality), composition rule, colour and tonality, material and detail density, and reserved copy space. Focal length, lens type, and aperture are photographic-only fields — skip them entirely for every other style. Skin strategy, in-image text, and render engine are decided at this step as well. When information is missing, ask once in a single pass; aspect ratio is not part of the question.

The basis for completion is in §10; the structure for assembling the finished prompt is in 10.3.

**Translating vague language into technical terms**

What users give you is usually a feeling, not a parameter. Copying "premium feel" straight into the prompt is the same as writing nothing — it points to no executable difference in the image. First ground it in optics, lighting, colour, and composition; then assemble.

| User says | Concrete prompt terms |
|---|---|
| Premium / high-end feel | Low saturation, medium-low contrast, modelled by side or side-back light, name the material concretely (glaze speckle / brushed metal / frosted glass), restrained post-processing |
| Atmosphere / moody | State the light quality and colour temperature (e.g. 3200K warm, low key), add an atmospheric medium (thin mist / floating dust / water vapour), darken the background |
| Cinematic / epic | 2.39:1 widescreen, anamorphic, strong rim light, high dynamic range with restrained post-processing, low angle or over-the-shoulder |
| Clean / minimalist | Large areas of negative space, a single dominant colour, no clutter, frontal softbox light to eliminate messy shadows |
| Instagram style | High brightness, low saturation, natural light, generous negative space, subject occupying ≤ 1/3 of frame, light film grain |
| Soothing / healing | High-key warm tones, soft light, rounded forms and soft materials, shallow depth of field, no strong contrast |
| Retro | Low saturation leaning yellow-green, film grain with slight light leaks, soft focus, swirly bokeh from an old lens |
| Tech | Cool colour temperature, hard or edge light, metal and glass materials, dark background with clean highlights |
| Warm | Colour temperature 3200–4000K, warm dominant colour, soft light, detail retained in the shadows |
| Cool / refreshing | Colour temperature above 5500K, cyan-blue palette, high-key lighting, clean highlights |
| Crisp / airy | Back or side-back light, clean atmospheric medium, low contrast at high brightness, highlights not blown |
| Heavy / weighty | Low-key lighting, large shadow area, high contrast, dark materials with coarse texture |
| Lived-in / everyday | Available light, slight handheld feel, household clutter in frame, warm colour temperature, imperfect composition |
| Commercial / e-commerce | Seamless background, softbox with black flags on both sides, true materials, clean edges, subject ≥ 70% of frame |
| Editorial / magazine | Strong conceptual composition, negative space and a title area, one light direction, tightly unified colour |
| Girly / youthful | High-key pink tones, soft light with slight overexposure, rounded forms, low contrast |
| Handcrafted | Visible handmade marks (paper grain / brush strokes / deckled edges), natural light, warm white balance |
| Hard-edged | High contrast, a single hard light, cool or neutral colour temperature, industrial materials, low angle |

A single feeling word usually lands on 3–4 dimensions at once (light quality, colour temperature, saturation, composition density) — it is not one word for one word. Colour temperature, saturation, and contrast stay under the rule in 10.2 — written only when the user asks, otherwise not written at all — so they are not written merely to fill out the dimension count. When the user stacks synonyms ("premium and clean"), take the intersection rather than listing them side by side.

Where a row's mapping involves camera or film terms (widescreen ratio, anamorphic, film grain), apply that part only when the style is photographic per 4.4; non-photographic styles take the light quality, colour temperature, saturation, and composition parts only.

### 3.4 Plan table
Output the table directly and proceed, **without blocking for confirmation**: `# / source or purpose / scene description / shot size / focal length and aperture / lighting / reference strength / notes (whether space is reserved for copy)`.

The count follows user input; if the user does not specify, default to **1 image** and do not ask. Leave the focal-length-and-aperture column empty for non-photographic styles; the shot-size column is filled when the frame needs it.

### 3.5 Generation and delivery
Before generating, build the task manifest and execute one by one, writing status back:

| # | Scene description | Reference strength | Status | Output |
|---|---|---|---|---|
| 1 | … | 70% | pending / generated / failed | actual file path |

- Call one at a time. Record the **actual output path** and status immediately after each image; do not rename files.
- A single failure **must not abort the batch**: record the index and reason, then continue.
- **Resuming**: when re-running the same task, read the manifest and the output directory first, skip what already succeeded, and do not spend credits twice.
- Do not change style or aspect ratio mid-run; if one image is unsatisfactory, regenerate only that image.
- After everything finishes, report: **N images total, X completed, Y failed**. List each failure with its index and reason; if only partially complete, say so plainly.
- Task acceptance, progress, and intermediate state do not mean the image is finished.

On delivery, present all images at once in manifest order, stating the aspect ratio used, image quality, count, output path, and how to use the result; verify that the files on disk match the manifest, and troubleshoot common issues per §12.

## 4. Style System
### 4.1 Locking a single style
The whole set shares one style string, covering the art medium, brushwork or rendering method, colour strategy, lighting logic, and post-processing texture. Once set, it must not change.

Example: `Overall 3D cartoon rendering, rounded forms, soft global illumination, rubber-like material feel, bright cream palette, slight depth of field, clean digital post with no noise.`

### 4.2 Style blending
Blending can be written in any of these ways — all are valid:

- With a plus sign: `70% cyberpunk + 30% CG animation`
- Without a plus sign: `cyberpunk blended with CG animation, leaning cyberpunk`
- Names listed side by side: `watercolour + Chinese ink wash`

**Steer the user toward giving explicit ratios**, for example "seven parts A, three parts B". When the user gives ratios, follow them exactly in the prompt.

When no ratio is given, infer from how the user ranks the two:

| User's wording | Ratio |
|---|---|
| "a bit more toward X" | Start from an even split, move X up 10 points |
| "a bit more still" / "more X" | Move another 10 points from the previous result |
| No ranking expressed | Split evenly: 50% each for 2 styles, 33.3% each for 3 |

Each "a bit more toward X" is fixed at 10 points: two styles start at 50/50, one request gives 60/40, another gives 70/30, and so on; three styles start at 33.3% each and step by the same amount. "Half and half" or "fifty-fifty" counts as an even split, but note that when the shares are equal the two styles tend to dilute each other's features.

State the resulting ratio at delivery.

Blend wording (substitute the shares from the table above; the example below is 70/30):

> `Predominantly A (70%), with B as a secondary influence (30%) blended into <dimension: background brushwork / material rendering / edge outlines>`

One sentence. Do not go on to explain how the two blend.

For an even split, rephrase as:

> `A and B blended in equal measure, each contributing <dimension split>`

For strongly conflicting combinations (photographic realism × flat vector), split by layer instead: subject in A and background in B, or material in A and contour in B; alternatively generate in A first, then convert with image-to-image in style B.

Requirements for the whole set: consistent blend ratio, light direction, and focal length.

### 4.3 Common style terms
**Modern image-generation staples**: photographic realism | cinematic CG | 3D cartoon rendering | Pixar style | Japanese cel shading | **Chinese animation / pure 2D manhua** | flat vector | Chinese ink wash | cyberpunk neon | pixel art | low-poly | Ghibli | American / Japanese comics | black-and-white film

**AI short-drama / manhua styles** (the 2026 mainstream; the words below go straight into the prompt):

- Chinese animation / 2D anime-series: clean lines, bright colour, refined colouring, strong character-sheet feel, clear panel framing
- AI simulated human, live-action realism: close to live-action texture, true skin texture, natural soft light, low smoothing, short-drama cinematic grading
- Realistic Chinese animation, cinematic rendering: between 2D anime and live action, refined features, cinematic lighting, reads like a CG film still
- 2D Chinese-style costume / ancient-romance fantasy: Eastern aesthetic, ink diffusion, gilded cloud light, exquisite costume detail, cool high-end tonality
- 3D Chinese animation, next-gen cel shading: volumetric 3D, refined modelling, outlines and layered shading, 3D-to-2D
- AI ink wash: Chinese ink wash translucency, layered ink diffusion, 3D ink-wash modelling

This group is drawing and rendering, not photography (see 4.4): shot size and camera angle are fine; focal length, aperture, and lens type are not written.

**Traditional media — needs brushwork or paper detail added to look good**: impasto oil painting | hand-painted watercolour | pencil line art | retro pop

### 4.4 Style Judgement and the Parameter Gate
Judge the style category before filling in any parameter — never match parameters wholesale. "Realistic" is not "photographic realism": illustrations, CG, and images can all be realistic. The gate covers photographic-only fields (focal length / aperture / lens type / film stock) only; shot size, camera position and angle, and focus and depth of field are framing language and are not style-restricted.

| What the user gives | Verdict | Photographic-only fields (focal length / aperture / lens type / film stock) | Render engine |
|---|---|---|---|
| No style given, just "generate an image of X" | Realistic (true materials and light), not photographic | No | No |
| A person, no style given | Realistic, defaulting to half-body | No | No |
| Photo / shoot / photography / documentary / product photography / portrait photography / cinematic live action | Photographic | Yes | No |
| Realistic / photoreal CG / realism-leaning 3D | Realistic rendering | No | Optional (see 7.1) |
| Illustration / hand-drawn / watercolour / oil painting / pencil / anime / cartoon / Pixar / Chinese animation / realistic Chinese animation / 2D costume fantasy / simulated-human realism / 3D Chinese animation cel shading / AI ink wash / Chinese style / vector / pixel / low-poly | Drawing and rendering, not photographic | No | No (3D Chinese animation and 3D-to-2D may take NPR terms) |

- Redrawing from an original image or reference image is not photographic by default and carries no lens parameters; treat it as photographic only when the source is itself a photograph and the user wants a photograph back.
- If the user gives both a style and camera direction ("illustration style, but shot on an 85mm"), follow the user.
- If the style changes within a request, the verdict changes with it — never carry the previous image's lens parameters over.
- Shot size, camera position and angle, focus and depth of field — plus lighting, composition, and colour — are outside this gate and are not style-restricted.

## 5. Character Consistency
### 5.1 Character sheet
**Fixed items**: gender and age range, face shape and features, skin tone, hairstyle / hair colour / length, build, standard outfit (**top + bottom + footwear, each item specified by cut, material, and colour**), signature accessories, makeup intensity, and skin treatment.

The bottom and footwear are worth writing when the set has to stay consistent. Leave them out and the model invents its own — differently every time, so the same character's trousers and shoes keep changing across a series.

**Variable items**: expression (broken down into specific changes of brow, eyes, and mouth), body pose and gestures, camera position and shot size, scene, and light direction.

Execution notes:

1. Generate one half-body reference portrait first, to serve as the anchor.
2. Pass the anchor image as reference input for every subsequent image, at the default reference strength of 70%. Too high locks the expression; too low lets the face drift.
3. In multi-character scenes, assign each person a codename and use it throughout.
4. When ageing the same character, keep the signature features and change only the age-related parts.
5. Image-to-image inherits the reference image's shot size. To change shot size, describe it as a hard constraint in the prompt (share of frame, crop position at the edges), or generate the anchor at the target shot size; if it still will not change, deliver at the original shot size and note that it can be cropped. Cropping costs no credits; regenerating does.
6. Between adjacent images in a series, change only one variable. Keep light direction and focal length constant throughout, so the subject does not jump around in the sequence.

## 6. Skin Strategy
Determined by style and purpose; any skin feature the user describes takes the highest priority.

| Condition | Mode | Notes |
|---|---|---|
| CG animation / anime / cartoon rendering / flat illustration / product commercial / beauty / fashion / social media | Retouched | Finished-makeup look: porcelain-smooth, even and luminous complexion, refined base makeup |
| Photographic realism / portrait photography / documentary / cinematic photoreal CG / realism-leaning 3D rendering | Realistic | Keeps natural pores and subtle oil sheen, clean skin tone, minor blemishes |
| Sci-fi / cyberpunk / semi-realistic CG / illustration with real optics (shallow depth of field, real lenses) | Realistic | A fictional subject does not mean the skin should be sanded flat. Whenever the frame calls for real materials and real optics, use Realistic |
| Mixed styles | Follow the larger share | E.g. 70% CG animation + 30% cyberpunk → Retouched |
| User describes skin features explicitly | Follow the user | Write it into the character sheet and keep it consistent across the set |

Empirical note: if cyberpunk or semi-realistic characters are wrongly set to Retouched, the skin turns plastic and the true texture of leather and fabric is sanded away along with it. Prefer Realistic for these subjects.

**Retouched mode string**

> Skin: porcelain-smooth and translucent, even luminous complexion, plump and hydrated, commercial beauty-retouch finish, soft refined highlight transitions, clean and clear makeup.
> `flawless porcelain-smooth skin, even luminous complexion, high-end beauty retouching, soft natural shading, naturally refined`

**Realistic mode string**

> Skin: true portrait skin texture, keeping natural pores and the subtle oil sheen on the nose wings and forehead, clean complexion, minor and naturally distributed blemishes, soft light-to-shade transitions preserved.
> `natural skin texture with fine pores, subtle natural oil sheen, healthy even skin tone, light retouching, realistic portrait skin`

In both modes, lashes, lip texture, and individual hair strands are never over-smoothed. The English strings above are prompt text submitted verbatim to the model — copy them as they are.

### 6.1 Only wanted content is written — where unwanted content goes depends on the channel

**The positive prompt writes only what should appear in the frame.** Where unwanted content goes is decided by the channel's schema, checked per 11.2:

- **No negative-prompt field** → unwanted content is simply not written: not as a negation ("no freckles", "no watermark"), and never named in any form at all. If it is not wanted, it is not written.
- **A negative-prompt field exists** → unwanted content is written there, in the negative prompt. It still stays out of the positive prompt entirely.

The reason the positive prompt must stay clean either way is practical: a concrete noun named in the positive prompt, even in order to be denied, becomes more likely to appear in the frame. Putting it in the negative prompt is what makes naming it safe — that field exists precisely to be denied.

Where the wanted result still does not appear even with a negative prompt, go to targeted second-pass refinement (see 6.2 and 10.4) rather than piling more exclusions on.

This governs generation from scratch. In image-to-image, removing something is often the task itself: the user asks to take out text, a watermark, a logo, or clutter, or asks for a quality raise or a redraw. Those are written as the task and are not withheld by this rule (see §8).

### 6.2 When positive description is not enough
Switch to targeted second-pass refinement (see 10.4): use that image as reference input at a high reference strength (80% or above), write only the skin delta in the prompt (even out skin tone, strengthen smoothing, make the makeup finish even and settled), and state that everything else stays unchanged. In practice this works on stubborn priors such as "the model tends to give young women freckles".

## 7. Render Engines and Physical Simulation
### 7.1 Whether to include one
The engine and solver are **determined jointly by style and content** — the same verdict used by the 4.4 gate — and are never mandatory. Photorealism, realism-leaning CG, and 3D rendering subjects may include one; 2D, illustration, flat, pixel art, and line art should not, or the frame will drift toward realism.

**Most images look better without an engine name.** Include one only when the frame clearly depends on a specific rendering behaviour (subsurface scattering on skin, metal caustics, ray-traced reflections, volumetric media). Specify one engine per image; never stack them.

### 7.2 Common engines
| Engine | Character | Suited to |
|---|---|---|
| Arnold | Mature subsurface scattering on skin, hair, and cloth | Cinematic characters, photoreal people |
| RenderMan (RIS) | Film-industry standard, hair and volumetric rendering | High-spec CG shots, animated features |
| V-Ray | Broad general purpose, large material library | Architecture, product, film — general |
| Corona | Fast output, soft natural light | Architectural interiors, product still life |
| Cycles | Open-source offline, feature complete | CG shorts, concept scenes |
| Redshift | GPU, film-grade, supports SSS and hair | Characters, product, advertising |
| Octane | GPU spectral rendering, clean highlights on metal and glass | Product, industrial parts, beauty |
| Unreal Engine 5 | Lumen global illumination, Nanite, path tracer | Game-look frames, virtual scenes, realistic Chinese animation (needs toon shading / NPR) |
| Keyshot | Industrial-grade materials and colour | Product design, material review |
| Houdini Karma | Particles, fluids, pyro, volumetric media | VFX shots |
| Blender EEVEE + toon shading / UE5 NPR | Controllable outlines and layered shading | Japanese animation, 3D-to-2D |

For 3D-to-2D subjects, an engine name is usually unnecessary; `NPR toon rendering, cel-shaded layered shading, clean outlines, flat highlights` steers better.

**First choice by subject**: product / industrial parts / e-commerce hero → Octane, Keyshot | film-grade characters → Arnold, Redshift | interiors and architecture → Corona, V-Ray | real-time feel and virtual scenes → UE5 | CG shorts → Cycles | particles and fluids → Houdini Karma | anime and cel shading → EEVEE + toon shading | 3D Chinese animation and 3D-to-2D → UE5 + toon shading (NPR).

### 7.3 Companion optics and shading terms
Global illumination (GI), HDRI environment light, subsurface scattering (SSS — essential for skin, candles, jade, and dairy), ambient occlusion (AO), ray-traced reflection and refraction, caustics (glass, water, metal), volumetric media and volumetric fog, depth of field (DOF), sampled denoising, path tracing, micro-poly displacement, ACES tone mapping.

These terms steer the image more effectively than the name of a renderer does; prefer them whenever you need to control how the frame is rendered.

### 7.4 Physics simulation and solving
When cloth, hair, liquid, smoke, fracture, or particle crowds appear in the frame, whether their form reads as plausible depends on dynamics solving. Such subjects require two things to be stated together: **which solver**, and **the form it produces**. A software name alone has limited effect; the shape description matters more.

| Simulated object | Common solvers | Shape description |
|---|---|---|
| Clothing, cloth, drape | Marvelous Designer, CLO 3D, Houdini Vellum, Maya nCloth, Blender cloth | `natural drape, realistic fold structure, fabric tension at stress points, soft irregular folds` |
| Hair, groom | XGen, Ornatrix, Yeti, Houdini hair | `natural gravity fall, individual strand detail, subtle frizz, layered groom` |
| Liquids, splashes | RealFlow, Houdini FLIP, Phoenix FD, Blender Mantaflow, LiquiGen | `surface tension, droplet breakup, thin sheet and ribbon forms, splash crown` |
| Fire, smoke, explosions | Phoenix FD, Houdini Pyro, FumeFX, EmberGen | `volumetric smoke with wispy detail, self-shadowing, buoyancy-driven plumes` |
| Rigid bodies, fracture, soft bodies | Houdini RBD, Bullet, PhysX / NVIDIA Flex, Blender MPM | `Voronoi fracture pattern, secondary debris, plausible mass and stacking` |
| Particles, crowds | Houdini POP, X-Particles, nParticles | `non-uniform distribution, size variation, clumping, motion trails` |
| Vegetation, ecology | SpeedTree | `species-accurate branching, wind response, seasonal variation` |
| Crowds, traffic | Massive, Golaem | `varied gait and spacing, individually distinct silhouettes` |

Boundary: do not itemise these into a still image. Mention them only when the frame **clearly depends** on a particular dynamic form — a fluttering hem, splashing liquid, falling hair, a smoke explosion, the instant of fracture. The whole set must share one set of dynamics assumptions; for example, hair and hem must agree in direction under the same gust of wind.

Common combinations: Houdini for solving + Redshift / Arnold / Karma for rendering; Marvelous Designer for garment patterns → export → Keyshot / Octane for rendering.

### 7.5 Consistency and boundaries
The whole set uses the same engine, the same tone mapping, and the same GI settings, so material reflections stay consistent from image to image, and the engine is never switched mid-set — changing it changes the style. If the user specifies an engine, follow it.

Writing an engine name steers the rendering style; **it does not mean that pipeline is actually invoked, and you must not promise the user that it is.**

## 8. Reference-Based Second-Pass Generation
| Task | Reference strength | Treatment |
|---|---|---|
| Whole-image redraw | 40–50% | Pass in the original and write the target result in the prompt; lower the reference strength for larger changes |
| Change of outfit, hairstyle, or colour scheme | 70% | Write only the changes in the prompt, and state that the rest stays unchanged |
| Background replacement | 70% | With an existing image, use image-to-image and change only the background; without one, generate from scratch (see below) |
| Outpainting (frame extension) | 75–85% | Keep the original camera position, focal length, light direction, and colour temperature; describe what fills the extension. Perspective mismatch may appear at the original image's edges — say so in advance |
| Local object replacement | 70% | State the object to replace and the area to keep; split complex changes into steps |
| Removal of an element (text, watermark, logo, clutter) | 70–85% | Name what is removed and what should fill its place; this is the task itself and is not restricted by 6.1 |
| Style transfer | 40–50% | Pass in the original, describe the target style and its share; may need several rounds of fine-tuning |
| Sharpness refinement | 75–85% | See 10.4 |
| Character-consistency anchor pass | 70% | See 5.1 |

**Targeted environment and background changes (e.g. "change the background to night")**

The criterion is "has the existing image already reached a reusable standard", not "is there an existing image":

- **A usable existing image** → use it as reference input for image-to-image at 70% reference strength, write only the background delta in the prompt (change to night, with night light sources and atmosphere), and state that the subject, camera position, and composition stay unchanged.
- **No existing image** (the user gave only a description) → generate from scratch to the new brief, writing the full day-to-night difference into the prompt.
- **An existing image whose subject or composition is itself unsatisfactory** → do not use it as reference; generate from scratch to the target, so that the unsatisfactory parts are not locked in along with it.

Environment changes (day/night, weather, season, scene replacement) follow the same rule.

**Reference strength and adherence are the same concept under different names in different channels**; treat them as a single value.

Map to the channel in three tiers: a channel that accepts a continuous value → pass the corresponding value; a channel that offers only tiers → map 70% to the nearest high tier; a channel with no such parameter → state the degree of preservation in the prompt.

State explicitly in the prompt which parts **stay unchanged** and which parts **change**.

## 9. Scene Recipes
The "Lens and camera" column: the camera position, viewpoint, and shot-size parts apply to every style; only focal length and lens type are photographic-only and are omitted elsewhere (see 4.4 and §10).

| Need | Lens and camera | Lighting | Composition | Watch out for |
|---|---|---|---|---|
| Tutorial step images | 45° tilted top-down or straight top-down, fixed across the set | Soft and even, avoiding hard shadows from hands | Subject centred or slightly left, annotation area on the right | Props, background, and angle identical at every step; change only one variable between adjacent images |
| Top-down flat lay / knolling | Straight top-down at 90°, lens perpendicular to the surface | Broad soft light, consistent projection or shadowless | Objects arranged in parallel with even spacing | State that items do not overlap, or stack by rule; align to an axis |
| Diagrams, schematics | Isometric or front-on | No strong shadows, even brightness | Aligned elements, clear connectors | Consistent line weight and palette, negative space reserved for labels |
| Cutaway, exploded view | Front-on or 45° isometric | Even soft light | Parts laid out along an axis at even spacing | State the number and arrangement of parts; avoid freely added parts |
| Product hero, e-commerce | 50–85mm, f/5.6–8, subject ≥ 70% | Softbox with black flags on both sides to define form | Centred or rule of thirds | Clean edges, seamless background, true materials |
| Poster KV, cover | Subject occupies most of the frame | Backlit silhouette or a strong colour-block background | Title area top and bottom | Most channels do not support native 16:9 or 9:16 (see 11.3) |
| Social-media 3×3 grid | Single-image subject stands out | One colour grade across the set | Slightly above centre | Consistent colour system across the set; each image complete on its own |
| Storyboard, shot breakdown | Wide → medium → close-up progression | Same light source | Keep the axis consistent between adjacent shots | Note the shot size and camera movement in each panel |
| Illustration, picture book | One consistent medium and brushwork | Lighting logic inherent to the style | Generous negative space | The style string never changes across the set |
| UI, icons, assets | Front-on, solid background | No lighting | Centred, even margins all round | State the negative-space dimensions and background requirements |

## 10. Framing and Optical Parameters
Two layers — do not mix them (verdict per 4.4):

- **Universal layer, not style-restricted**: shot size, camera position and angle, focus and depth of field — see 10.0. This is framing language, not photographic property; illustration, anime, Chinese animation, and AI short-drama styles use it too.
- **Photographic-only layer, written for photographic styles alone**: focal length, lens type, aperture, film stock — see 10.1–10.2. Photo / live action / photography / product photography / portrait photography / documentary may take them; illustration, redraw-from-original, anime, cartoon, 3D rendering, photoreal CG, vector, pixel, 3D-to-2D, and AI short-drama styles take none of them. Wrong parameters are not merely useless; they drag a non-photographic frame toward photographic texture.
- Lighting, composition, and colour are not gated.

### 10.0 Shot size, camera position, and focus (not style-restricted)
**Shot size**: extreme wide / wide (environment leads, establishes scale) | full shot (whole figure in context) | medium shot (half-body to knee, the narrative workhorse) | medium close-up (chest up, emotion) | close-up (face or object detail, emphasis) | extreme close-up (single eye, single material detail) | aerial top-down (whole scene from above) | macro (object texture). With no specification, people default to half-body and objects to the whole item in frame.

**Camera position and angle**: eye level (conversational) | low angle / looking up (authority, grandeur) | high angle / looking down (smallness, order) | straight top-down 90° (inventory, flat lay) | 45° tilted top-down (volume and information together — the common choice) | over-the-shoulder (immersion) | Dutch angle (unease, use sparingly) | aerial / bird's-eye | POV first person. With nothing said about it, treat it as eye level and write nothing; once the user has covered the angle in any wording, no camera-position term is added (see 1.1). Avoid three consecutive frames from the same angle.

**Subject facing**: full front / three-quarter / profile / back three-quarter / back / looking back over the shoulder. Written when it matters — leave it out and the model picks a facing at random.

**Focus and depth of field**: shallow depth of field (subject separated, background blurred) | deep depth of field (everything sharp, environment established) | foreground blur (layering) | focus on a specific point (eyes, product detail) | focus pull (narrative). When focus is written, state where it sits and how much is blurred — this is framing, not a photographic parameter.

### 10.1 Focal length and lens type
**Focal length**: 14–24mm ultra-wide (spatial tension, architecture) | 35mm (narrative, environmental portrait, documentary) | 50mm (standard view, product and everyday) | 70–85mm (portrait workhorse) | 100–135mm (headshots, food, jewellery) | 90–105mm macro (material texture) | 200mm and beyond telephoto (spatial compression, voyeuristic angle).

**Lens type** — specify one per image, never stacked:

| Type | Visual character | Suited to |
|---|---|---|
| Portrait prime (85mm class, f/1.2–1.4) | Flattering facial perspective, creamy bokeh | Character close-ups, expert portraits |
| Standard prime (50mm class) | Close to human vision, high micro-contrast, sharp edges | Product, everyday scenes, wherever realism matters |
| Macro (90–105mm, 1:1 or greater) | Extremely clear texture | Ingredients, jewellery, fabric, components, print detail |
| Probe / specialty | Wide-angle bug-eye view, large depth of field even up close, fits into narrow gaps | Industrial inspection, food fly-through shots, product interiors |
| Tilt-shift | Perspective correction, verticals stay vertical, plane of focus can be tilted | Architecture, interiors, merchandise, plated dishes (can produce a miniature depth of field) |
| Cine lens | Breathing corrected, rounded bokeh, rich gradation | Film-grade frames, narrative CG |
| Anamorphic | Oval edge flares, horizontal blue streaks, 2× squeeze | Cinematic posters, cyberpunk imagery |
| Medium format | Extremely high detail, fine skin, shallow depth of field | High-end commercial, fashion, large-format output |
| Telephoto (200mm and beyond) | Spatial compression, background reduced to colour blocks | Sports, wildlife, city skylines, suspense |
| Aerial / action camera | Fixed aperture, deep depth of field, first-person motion feel, barrel distortion and strong stabilisation | Aerial top-downs, extreme sports, POV angles, real-estate aerials |
| Fisheye | Near or beyond 180°, extreme edge distortion | Skateboarding, extreme sports, installation views, high-impact KV |
| Vintage lens | Swirly bokeh, edge smearing, soft nostalgic focus | Retro subjects, atmosphere-driven portraits |
| Smartphone main camera | Equivalent 24–28mm, mild compression, authentic casual feel | UGC, authentic social-media look, lifestyle content |

Brand and specific model steer the image far less than focal length and aperture values do, so they are not required fields.

### 10.2 Aperture, lighting, composition, colour, texture
**Aperture (photographic only)**: f/1.2–1.8 isolates the subject — at most one such image per set | f/2.8–4 general purpose | f/5.6–8 fully sharp for product and group shots | f/8–11 architecture, interiors, deep depth of field. Every set should include at least one f/8 wide shot to establish the environment.

**Lighting**: default to three-point lighting (key to model form, fill so the shadows do not block up, rim to separate subject from background). Options include softbox, 45° side-top light, Rembrandt lighting, butterfly lighting, split lighting, rim / back light, window light, golden hour, blue hour, and low-key lighting with a single hard source. For each, state the light direction and whether the light is hard or soft.

**Colour temperature, colour tone, saturation, and contrast — written only when the user asks for them. Otherwise they are never written at all.** Not even the model's own judgement opens an exception, and they are not added to make the frame feel more complete. A style that already carries its own look — cyberpunk, ink wash, Chinese costume fantasy — needs none of them written for it. When the user does ask, use a Kelvin value for photographic styles, and plain colour wording for illustration, CG, and every other non-photographic style (warm tones / cool tones).

> **Common misreading — do not make it.** This rule is *not* "middle values are not written". "Medium saturation, medium contrast" is indeed never written, but the reason is that the user did not ask, not that the value sits in the middle. There is exactly one test: **did the user ask for it?** If not, write nothing.

**Composition**: rule of thirds (default) | central symmetry | leading lines | framing | negative space (use when copy must be placed) | golden ratio | knolling.

**Colour**: saturation and contrast follow the rule above. Two points specific to this entry: middle values such as "medium saturation, medium contrast" are not written even when colour is wanted, because a middle value says nothing and only takes up words; and dominant, secondary, and accent colours are written only when the user asks for a palette, or when a multi-image set has to stay consistent — not for a single image. Avoid mixing warm and cool, several dominant colours at once, and overdone teal-and-orange.

**Texture (photographic styles only)**: Kodak Portra (soft warm) | Velvia (high-saturation landscape) | CineStill (warm halation in night scenes) | black-and-white film (emphasises structure) | clean digital with no grain (the default for business, e-commerce, and tech). Write `natural dynamic range, restrained post-processing` rather than any HDR-style description.

### 10.3 Prompt structure
**Write it as one passage of natural-language description, comma-separated.** Put down whatever the frame needs, in the user's own plain wording — subject, clothing, materials, lighting, framing, style. Use the fewest words that still make the picture clear. Comma-separated descriptive phrases are the normal way to write this and have always been fine; what matters is that every phrase carries concrete, executable information, and that the whole thing reads as one continuous description of the picture.

Two things to avoid: a bare keyword list with no descriptive content (`coffee, tasty, premium`), and a literary paragraph of prose. This is a description of a picture, not a story — do not write flowing narrative sentences, do not pad it out, and do not turn it into several paragraphs of composition. More words do not make a better image.

The items below may be covered, written out in the order given — they are the options and their order, not a checklist to fill in. **Only the subject is mandatory.** A single image normally draws on four to six of them; skip the rest:

1. **Subject and features** — who or what it is: appearance, material, colour, state.
2. **Action or state** — what the subject is doing, or the state it is in.
3. **Environment** — from near to far: what is in the foreground, the plane the subject occupies, what the background is.
4. **Style** — medium, brushwork or rendering method, colour strategy, lighting logic, post-processing texture; state the ratio when blending.
5. **Rendering and physics** (judge per 7.1) — only when the frame clearly depends on a particular rendering behaviour.
6. **Character sheet** (including skin strategy, see §6) — for a set that must stay consistent, write the fixed features and the variable items into the same passage; for a single image it is on demand, not a reason to describe every part of the body.
7. **Framing and lens** — shot size, camera position and angle, focus and depth of field (not style-restricted); focal length, lens type, aperture (photographic styles only). Skip this item when the user has already covered angle or shot size.
8. **Lighting** — light direction and hard or soft quality; colour temperature and colour tone only when the user asks (see 10.2).
9. **Composition and palette** — composition rule, negative-space placement, texture and post-processing; dominant/secondary/accent colours, saturation and contrast only when the user asks (see 10.2).
10. **Material and detail density** — whether to add detail terms is the model's judgement (see 10.4).

**The same thing is never written twice.** What is already there is not written again in other words, and nothing is added just because the list has an entry for it. Keep it to one passage and stop once the frame is described. When information is thin, add only what is needed to make the frame unambiguous.

Example (original request: "a coffee product shot" — a photographic style, so lens parameters are written):

> A freshly brewed pour-over coffee in a matte ceramic cup, fine steam rising from the rim, a few beans and a paper filter on a light oak table as foreground, against a softly blurred warm wood wall. Clean commercial photographic realism. Camera at a 45° tilted top-down, 90mm tilt-shift lens at f/5.6. A softbox upper left as side-back light tracing the steam, a reflector on the right holding down the shadows. Rule-of-thirds composition, empty space upper right for copy. The cup keeps fine glaze speckle and wheel-thrown texture. `sharp focus, fine detail, crisp clean edges, natural texture`

Polishing example — the user supplies their own prompt, so it is polished, not rewritten: keep what is already there, add only what removes ambiguity, add nothing because the list has an entry for it.

> A cyberpunk fox-girl, white background, full-body front view at a parallel angle, nine thick fluffy white tails spread behind her (four on each side, one above the head, tips glowing neon blue).

Contrast (the default when no style is given): the request "draw a cat sitting on a windowsill" is judged realistic, not photographic — no focal length, aperture, lens type, or film stock; only true materials and light, light direction, and composition. People default to half-body.

Counter-example: `coffee, looks tasty, premium feel`. It contains no executable information and no relationships between elements.

### 10.4 Sharpness, detail terms, and second-pass refinement
Image-quality parameters take the user's specified values; when unspecified, use the highest tier the channel offers. Sharpness defaults to highest.

**Detail terms** — `sharp focus, fine detail, high micro-contrast, crisp clean edges, professional retouching, natural texture`, and others of that kind. Whether they are needed, and where they go, is the model's judgement, style by style. Worth keeping in mind: they describe objective attributes of the frame — where the focus sits, how much detail there is, whether the edges are clean, whether the materials read true — rather than asking for a grade to be given to the image.

> `sharp focus, fine detail, high micro-contrast, crisp clean edges, professional retouching, natural texture`

For images with people, determine the skin mode per §6 and do not let the generic `natural texture` decide skin. Retouched mode adds `high-end beauty retouching, skin smoothing, even skin tone, subtle glow`.

**Second-pass refinement**: pass the previous version as a reference image at 75–85% reference strength to preserve the composition, and write only the delta in the prompt (raise material clarity, clean up edge fringing, reduce noise; skin direction per the mode in play). This step consumes extra credits and must be stated in advance.

## 11. Invocation Rules
### 11.1 Parameter validity
Before calling, check the parameter documentation actually returned, especially the valid range of `size` and the tiers of `quality`. `1024x1024` and `high` are common values, not guaranteed ones.

Never pass parameters the channel does not support.

The prompt is submitted verbatim; do not accept channel rewriting (see 1.1).

### 11.2 Negative prompt — gated by the channel schema

**A negative prompt is never written by default.** Before assembling the prompt, read the target channel's input schema through the tool metadata interface and check whether it exposes a negative-prompt field — commonly `negative_prompt` or `negative_prompt_2`; some channels name it differently but expose the same capability.

- **The field exists** → write a negative prompt for that channel.
- **The field does not exist** → write none. Do not smuggle "what must not appear" into the positive prompt as a negation (6.1). A channel with no such field simply gets no negative prompt; the wanted result is pursued through positive description and, if that is not enough, second-pass refinement (6.2 / 10.4).

When one is written, it follows the same discipline as the positive prompt: concrete nouns, no padding, and nothing that contradicts what the positive prompt asks for. It is **never** used to carry a sensitive-wording rewrite — see 1.2.

What belongs in it: the artifacts and failure modes this channel is prone to and this frame can actually suffer from — malformed hands, fused or extra fingers, extra limbs, garbled text, warped background geometry, duplicated tails or ears when a count was specified, over-smoothed plastic skin. Keep it short; do not list everything.

### 11.3 Aspect-ratio mapping
| User's wording | Orientation | Notes |
|---|---|---|
| Unspecified / square / 1:1 | Square | `1024x1024` |
| Landscape, cover, presentation image, 16:9 | Largest landscape size | Commonly `1536x1024` (3:2), not native 16:9 — must be stated |
| Portrait, phone poster, 9:16 | Largest portrait size | Commonly `1024x1536` (2:3), not native 9:16 — must be stated |
| 4:5, 21:9, and others | Nearest supported size | State that it has been approximated, and offer cropping as an option |

The user's stated ratio carries the highest weight. An unsupported ratio must never be substituted silently; there are two paths: generate at the nearest ratio, or generate first and crop afterwards. Cropping consumes no extra credits.

### 11.4 Output
Output to `generated-images/<task-name>/` inside the workspace, with a short slug for the task name. Filenames are produced by the generation channel and are not renamed. For series images, keep the order clear. On delivery, present all images at once in manifest order.

### 11.5 Credits
Before batch generation, state the number of images and the estimated consumption, **without blocking execution**. On resume, skip the images that already succeeded so credits are not spent twice.

## 12. Common Issues and Prevention
| Issue | Prevention |
|---|---|
| Framing dragged into a bust shot by the reference image | Generate the anchor directly at the target shot size; do not produce a half-body first and hope to tighten it later |
| Malformed hands | Keep hands out of prominent positions, have the subject hold something or place the hands at the edge, and write `natural anatomy` |
| Freckles and coarse texture in a Retouched subject | Mount the positive skin string from §6; the positive prompt names no unwanted feature at all, and unwanted features go in the negative prompt when the channel exposes the field (6.1 / 11.2); if that still fails, go to targeted second-pass refinement per 10.4 |
| Garbled text in the image | Decide language and content per 1.3, and verify key text by hand or composite it in post after generation |
| Sensitive wording in the user's own prompt rewritten without asking | Keep it verbatim per 1.2 and flag it in the reply; the user decides what to do |
| Tilted horizon or building verticals | Write `level horizon, vertical lines corrected` |
| Over-processing | Write `natural dynamic range, restrained post-processing` |
| The same character inconsistent across images | Character sheet + anchor image + image-to-image pass-back, reference strength 70% |
| Style drift | The style string stays identical across the set, and a blended style keeps the same ratio throughout; confirm the prompt was not rewritten by the channel |
| Subject jumping between images in a series | Change only one variable between adjacent images, keeping light position and focal length constant throughout |
| Realistic subjects smoothed too far | Use the Realistic string from §6 |
| Image-to-image drifting too far from the reference | Raise the reference strength, and list in the prompt the parts that must stay unchanged |
| An illustration or redraw request acquiring focal length, aperture, and film stock | Judge the style per 4.4; lens parameters are for photographic styles only |
| The user's own prompt being cut down | Polish only, never delete; remove only self-contradictory noise, and report each change |
| One failed generation aborting the whole batch | Record the failure separately and continue; report partial completion at the end |

## 13. Prompt Validation

Run the finished prompt through this check **before the generation call**. A failure caught here costs nothing; the same failure caught after generation costs credits.

1. **Style verdict** — does the verdict from 3.2 match the parameters actually written? A drawing-and-rendering or illustration frame carrying focal length, aperture, lens type, or film stock fails. Shot size, camera position and angle, and focus and depth of field are framing language and are always allowed.
2. **Engine** — at most one engine, and only where 7.1 allows it. Naming two fails.
3. **Gated colour fields** — colour temperature, colour tone, saturation, and contrast appear only if the user asked for them (10.2); dominant, secondary, and accent colours only if the user asked for a palette or the set must stay consistent.
4. **Writing rules (10.3)** — one continuous comma-separated passage, neither a bare keyword list nor a literary paragraph; the same thing is not written twice; nothing is in there merely because the list in 10.3 has an entry for it; the passage stops once the frame is clear.
5. **Nothing cut** — every element of the user's own prompt is still present and every change is reported item by item (1.1); sensitive wording stays verbatim (1.2).
6. **Hands and limbs** — where a person is in frame, a hand clause is present and `natural anatomy` is written (§12).
7. **Negative prompt** — present only when the channel exposes the field (11.2), and never contradicting the positive prompt.

Fix every failure inside the prompt, then proceed. If a failure cannot be fixed without cutting something the user wrote, stop and say so rather than cutting it silently.

## 14. Delivery Checklist
The prompt passed the validation in §13; files on disk match the task manifest; aspect ratio matches what was agreed, and any unsupported ratio was stated in advance; image quality takes the user's specified value or the channel's highest tier; the style string is identical across the set and any blend ratio has been restated; the style verdict matches the parameters actually used (no focal length, aperture, lens type, or film stock in a non-photographic frame; shot size, camera position, and focus are not style-restricted); no element of a user-written prompt was cut, and every change was reported item by item; characters are consistent across images; no garbled text and no malformed limbs (text generated per 1.3 has been verified by hand); sensitive wording from the user's own prompt was kept verbatim and flagged in the reply rather than rewritten (see 1.2); colour temperature, colour tone, saturation, and contrast were written only if the user asked for them (see 10.2); sequence order is correct; the prompt was not rewritten by the channel.

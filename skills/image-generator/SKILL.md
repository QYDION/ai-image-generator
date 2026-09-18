---
name: image-generator
description: Generates a batch of stylistically consistent, professional-grade image series from documents, images, or text descriptions; also supports reference-based redrawing, outfit and hairstyle changes, background replacement, outpainting, local object replacement, style transfer, and sharpness refinement. Suited to series illustrations, multi-expression character sets, tutorial step images, top-down and flat-lay shots, diagrams and cutaway views, e-commerce hero images, poster key visuals, social-media grids, storyboards, and illustrations. Use when the user says "generate a set of images", "make series illustrations", "batch image generation", "produce N images", "turn this document into images", "a set of images in one consistent style", "multiple expression images of this character", "top-down view / diagram / step image", "generate at 70% style A + 30% style B", "change this image to X", "swap the background", or "extend the frame outward".
version: 1.0.0
license: MIT
author: 盘锦奇点科技有限公司 (Panjin QYDION Technology Co., Ltd.)
---

# Image Generator

> [中文](SKILL.zh-CN.md) | English

**Core responsibility: turn the user's brief description into a professional-grade image-generation prompt, and drive the generation.**

Users typically supply only a sentence or a few lines. This skill expands that into one paragraph of professional prompt ready to submit, then generates the images one by one against a manifest. It covers both generation from scratch and reference-based second-pass generation (image-to-image).

---

## 1. Submission Principles and Defaults

### 1.1 Submission Principles

The generated prompt is **submitted verbatim**. No channel may rewrite, polish, or expand it; if the channel in use performs automatic rewriting, turn that off. The same subject must stay fixed across the whole set: a character's appearance, face shape, skin tone, hairstyle, standard outfit, and accessories remain identical from the first image to the last, unless the user explicitly asks for a change of clothing, hairstyle, or another feature.

### 1.2 Defaults

| Item | Default |
|---|---|
| Aspect ratio | User input wins; if unspecified, 1:1, commonly `1024x1024` |
| Resolution and image quality | User input wins; if unspecified, the highest tier the channel offers |
| Sharpness | Highest |
| Reference strength / adherence | User input wins; if unspecified, 70% (tiered rules in §8) |
| Number of reference images | No cap; use however many the user supplies |
| Mixed-style ratio | User input wins; for "a bit more toward X", adjust by 10 points each time; if nothing is stated, split evenly (see 4.2) |
| Skin | Determined by style: non-photoreal subjects get flawless retouching; photographic realism, portrait photography, documentary, and cinematic photoreal CG keep natural pores and subtle oil sheen. Any skin feature the user describes takes precedence (see §6) |
| Render engine | Determined jointly by style and content; never mandatory. Most images look better without an engine name (see §7) |
| In-image text | Determined by the user's description: when an object that inherently carries text appears, render text along with the image, in the language the user explicitly specifies, falling back to the user's input language only when unspecified; otherwise generate no text, watermark, or logo, and use negative space where copy is needed (see below) |

Once the user specifies any of the above, follow the user's specification.

**How to decide on in-image text**

First read the user's description and decide whether any object in it inherently carries text: books, magazines, newspapers, posters, signage and storefronts, packaging and labels, price tags, menus, receipts, instruction sheets, UI screens and on-screen content, certificates, business cards, banners, and so on. When such an object appears, text is an inherent part of it and should be generated along with the image.

Language is decided by the following priority:

1. **Text content the user explicitly specifies has the highest priority.** If the user supplies the exact wording, render it character for character, whether or not it matches the language of the request. A user writing in English who asks for Chinese characters gets Chinese characters, and vice versa. If the user specifies only the language and not the content, generate in that language.
2. **When the user specifies nothing, default to the language of the user's input.** A Chinese description renders Chinese; an English description renders English.

- The user gave no content, but the object has an inherent text area → state that text of the corresponding language should appear there, without specifying the exact wording, so the area doesn't look implausibly blank.
- The object carries no text of its own → generate no text, watermark, or logo, and state by default that the frame is clean with generous negative space.

**Wrap text to be rendered in double quotes**

The prompt is a single passage of natural language. When the characters that should appear are mixed in with the words describing the scene, the model cannot tell which part is literal content. Wrapping the target text in double quotes gives it an explicit delimiter, cutting "the characters to appear" out of "the description of the scene":

> Design a poster whose title reads "BLUE NOTE SESSIONS", set in a bold condensed sans-serif.

- Put **only the literal content to appear** inside the quotes; all description goes outside them.
- To **change** text that already exists in an image, delimit both before and after with the same quotes: `change "OLD TEXT" to "NEW TEXT"`.
- Shorter text inside the quotes is more accurate. The model's ability to reproduce long sentences and multi-line typography is limited; anything longer than one line should become negative space, with typesetting handled in post.
- Prefer straight ASCII double quotes `"` (they are more common in training corpora).

Key text in an image (brand names, numbers, contact details) is high-precision content. After generation, verify each item by hand or recompose it in post — do not deliver it as-is (see §13).

---

## 2. Scope

**Not triggered by**: pure image engineering (precise cutouts, watermark removal, batch compression, format conversion, face anonymization), video generation, or 3D model generation. Every other natural-language image generation and editing request falls within scope.

---

## 3. Workflow

### 3.1 Reading input material

Text, Markdown, images, and PDFs can be read directly. If the current environment cannot read another format, ask the user to supply the text content; **do not promise format conversion**.

For long documents, extract the structure first (section headings, key conclusions, timeline) rather than placing the entire text in the prompt.

### 3.2 Intent recognition and parameter completion

Establish first: the purpose and where the image will be placed, the behaviour the image should trigger, the subject of the frame, and the overall tone.

Then fill in nine parameters: shot size, camera position and angle, focal length and lens type, aperture, lighting plan (direction + quality + colour temperature), composition rule, colour and tonality, material and detail density, and text and negative space. Skin strategy, in-image text, and render engine are decided at this step as well. When information is missing, ask once in a single pass; aspect ratio is not part of the question.

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

A single feeling word usually lands on 3–4 dimensions at once (light quality, colour temperature, saturation, composition density) — it is not one word for one word. When the user stacks synonyms ("premium and clean"), take the intersection rather than listing them side by side.

### 3.3 Plan table

Output the table directly and proceed, **without blocking for confirmation**: `# / source or purpose / scene description / shot size / focal length and aperture / lighting / reference strength / notes (whether space is reserved for copy)`.

The count follows user input; if the user does not specify, default to **1 image** and do not ask.

### 3.4 Generation and delivery

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

---

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

> `Predominantly A (70%), with B as a secondary influence (30%) blended into <dimension: background brushwork / material rendering / edge outlines / colour bias>; the two blend naturally, with consistent light direction, line weight, and saturation.`

For an even split, rephrase as:

> `A and B blended in equal measure, each contributing <dimension split>; consistent light direction, line weight, and saturation, avoiding mutual dilution of features.`

For strongly conflicting combinations (photographic realism × flat vector), split by layer instead: subject in A and background in B, or material in A and contour in B; alternatively generate in A first, then convert with image-to-image in style B.

Requirements for the whole set: consistent blend ratio, light direction, colour temperature, and focal length.

### 4.3 Common style terms

**Modern image-generation staples**: photographic realism | cinematic CG | 3D cartoon rendering | Pixar style | Japanese cel shading | flat vector | Chinese ink wash | cyberpunk neon | pixel art | low-poly | Ghibli | American / Japanese comics | black-and-white film

**Traditional media — needs brushwork or paper detail added to look good**: impasto oil painting | hand-painted watercolour | pencil line art | retro pop

---

## 5. Character Consistency

### 5.1 Character sheet

**Fixed items**: gender and age range, face shape and features, skin tone, hairstyle / hair colour / length, build, standard outfit (**top + bottom + footwear, each item specified by cut, material, and colour**), signature accessories, makeup intensity, and skin treatment.

The bottom and footwear must be written. Without them, the model invents its own — and invents them differently every time, so the same character's trousers and shoes change repeatedly across a series.

**Variable items**: expression (broken down into specific changes of brow, eyes, and mouth), body pose and gestures, camera position and shot size, scene, and light direction.

Execution notes:

1. Generate one half-body reference portrait first, to serve as the anchor.
2. Pass the anchor image as reference input for every subsequent image, at the default reference strength of 70%. Too high locks the expression; too low lets the face drift.
3. In multi-character scenes, assign each person a codename and use it throughout.
4. When ageing the same character, keep the signature features and change only the age-related parts.
5. Image-to-image inherits the reference image's shot size. To change shot size, describe it as a hard constraint in the prompt (share of frame, crop position at the edges), or generate the anchor at the target shot size; if it still will not change, deliver at the original shot size and note that it can be cropped. Cropping costs no credits; regenerating does.
6. Between adjacent images in a series, change only one variable. Keep light direction and focal length constant throughout, so the subject does not jump around in the sequence.

---

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

In both modes, lashes, lip texture, and individual hair strands are never over-smoothed.

### 6.1 Do not list concrete blemishes as negations

Writing "no freckles, no acne marks, no visible pores" pulls all three concepts into the prompt, and in practice makes freckles *more* likely to appear — text encoders handle negation poorly. Always switch to positive description (porcelain-smooth, even luminous complexion, refined base makeup) and avoid naming any blemish at all. When the channel offers a dedicated exclusion field, these items are more effective written there; the positive-description approach in this section applies when the channel has no such field (see 11.1).

The same principle applies to scene elements: when the object itself carries no text and text, watermarks, or logos must be kept out of the frame, prefer the positive "clean frame, generous negative space" over a negation such as "no text" (objects that inherently carry text are handled per 1.2 and are exempt from this rule).

Negating abstract textural adjectives (such as "not soft enough") has less impact than negating concrete nouns, but is equally limited in effect and is not a default tool.

### 6.2 When positive description is not enough

Switch to targeted second-pass refinement (see 10.4): use that image as reference input at a high reference strength (80% or above), write only the skin delta in the prompt (even out skin tone, strengthen smoothing, make the makeup finish even and settled), and state that everything else stays unchanged. In practice this works on stubborn priors such as "the model tends to give young women freckles".

---

## 7. Render Engines and Physical Simulation

### 7.1 Whether to include one

The engine and solver are **determined jointly by style and content**, and are never mandatory. Photorealism, realism-leaning CG, and 3D rendering subjects may include one; 2D, illustration, flat, pixel art, and line art should not, or the frame will drift toward realism.

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
| Unreal Engine 5 | Lumen global illumination, Nanite, path tracer | Game-look frames, virtual scenes |
| Keyshot | Industrial-grade materials and colour | Product design, material review |
| Houdini Karma | Particles, fluids, pyro, volumetric media | VFX shots |
| Blender EEVEE + toon shading / UE5 NPR | Controllable outlines and layered shading | Japanese animation, 3D-to-2D |

For 3D-to-2D subjects, an engine name is usually unnecessary; `NPR toon rendering, cel-shaded layered shading, clean outlines, flat highlights` steers better.

### 7.3 Subject mapping

| Subject | First choice | Alternative |
|---|---|---|
| Product, industrial parts, e-commerce hero | Octane, Keyshot | V-Ray, Corona |
| Film-grade characters (skin, hair, cloth) | Arnold, Redshift | RenderMan, V-Ray |
| Interiors, architecture, spaces | Corona, V-Ray | — |
| Real-time feel, game look, virtual scenes | Unreal Engine 5 | — |
| CG shorts, concept scenes | Blender Cycles | Arnold, Redshift |
| Large-scale scenes, particles, fluids | Houdini Karma | — |
| Anime, cel shading | Blender EEVEE + toon shading | UE5 + NPR |

### 7.4 Companion optics and shading terms

Global illumination (GI), HDRI environment light, subsurface scattering (SSS — essential for skin, candles, jade, and dairy), ambient occlusion (AO), ray-traced reflection and refraction, caustics (glass, water, metal), volumetric media and volumetric fog, depth of field (DOF), sampled denoising, path tracing, micro-poly displacement, ACES tone mapping.

These terms steer the image more effectively than the name of a renderer does; prefer them whenever you need to control how the frame is rendered.

### 7.5 Physics simulation and solving

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

### 7.6 Consistency and boundaries

The whole set uses the same engine, the same tone mapping, and the same GI settings, so material reflections stay consistent from image to image. If the user specifies an engine, follow it.

Writing an engine name steers the rendering style; **it does not mean that pipeline is actually invoked, and you must not promise the user that it is.**

---

## 8. Reference-Based Second-Pass Generation

| Task | Reference strength | Treatment |
|---|---|---|
| Whole-image redraw | 40–50% | Pass in the original and write the target result in the prompt; lower the reference strength for larger changes |
| Change of outfit, hairstyle, or colour scheme | 70% | Write only the changes in the prompt, and state that the rest stays unchanged |
| Background replacement | 70% | With an existing image, use image-to-image and change only the background; without one, generate from scratch (see below) |
| Outpainting (frame extension) | 75–85% | Keep the original camera position, focal length, light direction, and colour temperature; describe what fills the extension. Perspective mismatch may appear at the original image's edges — say so in advance |
| Local object replacement | 70% | State the object to replace and the area to keep; split complex changes into steps |
| Style transfer | 40–50% | Pass in the original, describe the target style and its share; may need several rounds of fine-tuning |
| Sharpness refinement | 75–85% | See 10.4 |
| Character-consistency anchor pass | 70% | See 5.1 |

**Targeted environment and background changes (e.g. "change the background to night")**

The criterion is "has the existing image already reached a reusable standard", not "is there an existing image":

- **A usable existing image** → use it as reference input for image-to-image at 70% reference strength, write only the background delta in the prompt (change to night, night light sources, colour temperature and atmosphere), and state that the subject, camera position, and composition stay unchanged.
- **No existing image** (the user gave only a description) → generate from scratch to the new brief, writing the full day-to-night difference into the prompt.
- **An existing image whose subject or composition is itself unsatisfactory** → do not use it as reference; generate from scratch to the target, so that the unsatisfactory parts are not locked in along with it.

Environment changes (day/night, weather, season, scene replacement) follow the same rule.

**Reference strength and adherence are the same concept under different names in different channels**; treat them as a single value.

Map to the channel in three tiers: a channel that accepts a continuous value → pass the corresponding value; a channel that offers only tiers → map 70% to the nearest high tier; a channel with no such parameter → state the degree of preservation in the prompt.

State explicitly in the prompt which parts **stay unchanged** and which parts **change**.

---

## 9. Scene Recipes

| Need | Lens and camera | Lighting | Composition | Watch out for |
|---|---|---|---|---|
| Tutorial step images | 45° tilted top-down or straight top-down, fixed across the set | Soft and even, avoiding hard shadows from hands | Subject centred or slightly left, annotation area on the right | Props, background, and angle identical at every step; change only one variable between adjacent images |
| Top-down flat lay / knolling | Straight top-down at 90°, lens perpendicular to the surface | Broad soft light, consistent projection or shadowless | Objects arranged in parallel with even spacing | State that items do not overlap, or stack by rule; align to an axis |
| Diagrams, schematics | Isometric or front-on | No strong shadows, even brightness | Aligned elements, clear connectors | Consistent line weight and palette, negative space reserved for labels |
| Cutaway, exploded view | Front-on or 45° isometric | Even soft light | Parts laid out along an axis at even spacing | State the number and arrangement of parts; avoid freely added parts |
| Product hero, e-commerce | 50–85mm, f/5.6–8, subject ≥ 70% | Softbox with black flags on both sides to define form | Centred or rule of thirds | Clean edges, seamless background, true materials |
| Poster KV, cover | Subject occupies most of the frame | Backlit silhouette or a strong colour-block background | Title area top and bottom | Most channels do not support native 16:9 or 9:16 (see 11.2) |
| Social-media 3×3 grid | Single-image subject stands out | One colour grade across the set | Slightly above centre | Consistent colour system across the set; each image complete on its own |
| Storyboard, shot breakdown | Wide → medium → close-up progression | Same light source | Keep the axis consistent between adjacent shots | Note the shot size and camera movement in each panel |
| Illustration, picture book | One consistent medium and brushwork | Lighting logic inherent to the style | Generous negative space | The style string never changes across the set |
| UI, icons, assets | Front-on, solid background | No lighting | Centred, even margins all round | State the negative-space dimensions and background requirements |

---

## 10. Camera and Optical Parameters

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

### 10.2 Aperture, angle, lighting, composition, colour, texture

**Aperture**: f/1.2–1.8 isolates the subject — at most one such image per set | f/2.8–4 general purpose | f/5.6–8 fully sharp for product and group shots | f/8–11 architecture, interiors, deep depth of field. Every set should include at least one f/8 wide shot to establish the environment.

**Angle**: eye level (default, person-to-person communication) | low angle (authority, pressure) | straight top-down (order, inventory) | 45° tilted top-down (three-dimensionality and information together — the common choice) | over-the-shoulder (immersion in narrative) | Dutch angle (unease, use sparingly). Avoid three consecutive images from the same angle.

**Lighting**: default to three-point lighting (key to model form, fill so the shadows do not block up, rim to separate subject from background). Options include softbox, 45° side-top light, Rembrandt lighting, butterfly lighting, split lighting, rim / back light, window light, golden hour, blue hour, and low-key lighting with a single hard source. For each, state the light direction, hard or soft quality, and colour temperature.

**Composition**: rule of thirds (default) | central symmetry | leading lines | framing | negative space (use when copy must be placed) | golden ratio | knolling.

**Colour**: set colour temperature first, then saturation and contrast. Low saturation with medium-low contrast reads as restrained; high saturation with high contrast reads as energetic. Dominant colour 60%, secondary 30%, accent 10%, consistent across the set. Avoid mixing warm and cool, several dominant colours at once, and overdone teal-and-orange.

**Texture**: Kodak Portra (soft warm) | Velvia (high-saturation landscape) | CineStill (warm halation in night scenes) | black-and-white film (emphasises structure) | clean digital with no grain (the default for business, e-commerce, and tech). Write `natural dynamic range, restrained post-processing` rather than any HDR-style description.

### 10.3 Prompt structure

**Write it as one coherent passage of natural language.** Do not use `[tag] + [tag]` stacking, and do not break the elements into a comma-separated keyword list. Today's mainstream image models read meaning from sentences; a passage of prose can express the relationships between elements — who is doing what, where the light comes from, how subject and background separate — whereas isolated tags can only assert themselves individually and cannot convey relationships.

A complete prompt unfolds naturally in the order below, with each item woven into the sentence rather than labelled:

1. **Subject and features** — who or what it is: appearance, material, colour, state.
2. **Action or state** — what the subject is doing, or the state it is in.
3. **Environment** — from near to far: what is in the foreground, the plane the subject occupies, what the background is.
4. **Style** — medium, brushwork or rendering method, colour strategy, lighting logic, post-processing texture; state the ratio when blending.
5. **Rendering and physics** (judge per 7.1) — only when the frame clearly depends on a particular rendering behaviour.
6. **Character sheet** (including skin strategy, see §6) — when people are present, write the fixed features and the variable items into the same passage.
7. **Lens** — camera position, focal length, lens type, aperture.
8. **Lighting** — light direction, hard or soft quality, colour temperature.
9. **Composition and palette** — composition rule, negative-space placement, dominant/secondary/accent colour ratios, texture and post-processing.
10. **Material and detail density**, closing with **quality terms** (see 10.4) and **constraints** (clean frame, generous negative space, level horizon). The exception is quality terms the user wrote themselves — those go at the very front (see 10.4).

One or a few connected sentences is enough; let them flow. When information is thin, fill it in on the principle of minimum sufficiency — do not pile up synonyms in order to "write everything".

Example (original request: "make a coffee product image"):

> A freshly brewed pour-over coffee in a matte ceramic cup, with fine steam rising naturally from the rim, a few coffee beans and a paper filter scattered across a light oak table as foreground depth, against a softly blurred warm wood wall. Overall clean, fine commercial photographic realism, low-saturation warm tones, terracotta brown as the dominant colour with deep brown accents, clean tonality and restrained post-processing with no grain. Camera at a 45° tilted top-down, shot on a 90mm tilt-shift lens at f/5.6. A softbox upper left as side-back light, tracing the outline of the steam at the rim, with a reflector on the right filling to hold down the shadows, colour temperature around 3800K leaning warm. Rule-of-thirds composition, with generous empty space in the upper right for copy to be placed in post. The terracotta body retains fine glaze speckle and wheel-thrown texture, and the liquid surface carries a clean specular reflection. `sharp focus, fine detail, high micro-contrast, crisp clean edges, professional retouching, natural texture`. Clean frame, generous negative space, level horizon.

Counter-example: `coffee, looks tasty, premium feel, 8k, ultra HD, masterpiece`. It contains no executable information and no relationships between elements.

### 10.4 Sharpness, quality, and second-pass refinement

Image-quality parameters take the user's specified values; when unspecified, use the highest tier the channel offers. Sharpness defaults to highest.

**Quality terms** go at the end of the prompt:

> `sharp focus, fine detail, high micro-contrast, crisp clean edges, professional retouching, natural texture`

Do not use `8k, ultra HD, masterpiece, trending, best quality`; the usual result is over-sharpening and a plastic texture.

The difference between the two is not "whether to use quality terms" but **describing objective attributes versus demanding a quality grade**. The string above speaks of focus, detail density, clean edges, and truthful materials — true for models of any generation. The banned group asks the model for a "quality tier", and current-generation models are already at the highest tier by default, so writing it adds no quality and only over-sharpening and plastic sheen. Quality terms remain necessary in SD-family workflows and are harmless redundancy on current-generation models, which is why they are kept.

When the user writes quality terms themselves (`8k`, `masterpiece`, `ultra HD`, and the like), treat it as user input and give it priority: keep their wording, substitute nothing, and **move it to the very beginning of the prompt** — earlier positions carry more weight, and only at the front is it guaranteed to take effect. The banned-terms rule above governs only what this skill appends on its own initiative; it does not govern terms the user specifies.

For images with people, determine the skin mode per §6 and do not let the generic `natural texture` decide skin. Retouched mode adds `high-end beauty retouching, skin smoothing, even skin tone, subtle glow`.

**Second-pass refinement**: pass the previous version as a reference image at 75–85% reference strength to preserve the composition, and write only the delta in the prompt (raise material clarity, clean up edge fringing, reduce noise, unify colour temperature; skin direction per the mode in play). This step consumes extra credits and must be stated in advance.

---

## 11. Invocation Rules

### 11.1 Parameter validity

Before calling, check the parameter documentation actually returned, especially the valid range of `size`, the tiers of `quality`, and whether a dedicated exclusion field exists. `1024x1024` and `high` are common values, not guaranteed ones.

Never pass parameters the channel does not support. Exclusions follow one of two paths depending on what the channel can actually do: if the channel provides a dedicated exclusion field, write the excluded content into that field; if it has no such field, write it as a declarative sentence or a positive description placed in the prompt, without negating instructions (for the reasoning, see 6.1). The presence of a field does not mean it takes effect — aggregation platforms often normalise every model onto one set of parameters (`prompt` / `negative_prompt` / `steps` / `guidance`), and the irrelevant fields may be silently ignored; if writing to the field has no effect, fall back to positive description rather than retrying repeatedly.

The prompt is submitted verbatim; do not accept channel rewriting (see 1.1).

### 11.2 Aspect-ratio mapping

| User's wording | Orientation | Notes |
|---|---|---|
| Unspecified / square / 1:1 | Square | `1024x1024` |
| Landscape, cover, presentation image, 16:9 | Largest landscape size | Commonly `1536x1024` (3:2), not native 16:9 — must be stated |
| Portrait, phone poster, 9:16 | Largest portrait size | Commonly `1024x1536` (2:3), not native 9:16 — must be stated |
| 4:5, 21:9, and others | Nearest supported size | State that it has been approximated, and offer cropping as an option |

The user's stated ratio carries the highest weight. An unsupported ratio must never be substituted silently; there are two paths: generate at the nearest ratio, or generate first and crop afterwards. Cropping consumes no extra credits.

### 11.3 Output

Output to `generated-images/<task-name>/` inside the workspace, with a short slug for the task name. Filenames are produced by the generation channel and are not renamed. For series images, keep the order clear. On delivery, present all images at once in manifest order.

### 11.4 Credits

Before batch generation, state the number of images and the estimated consumption, **without blocking execution**. On resume, skip the images that already succeeded so credits are not spent twice.

---

## 12. Common Issues and Prevention

| Issue | Prevention |
|---|---|
| A blemish appears *because* it was negated | Negating a concrete noun brings the concept into the frame. Switch to positive description, or go straight to second-pass refinement |
| Exclusion written to the field has no effect | The field may be a shell normalised in by an aggregation platform. Fall back to positive description and do not retry repeatedly |
| Framing dragged into a bust shot by the reference image | Generate the anchor directly at the target shot size; do not produce a half-body first and hope to tighten it later |
| Malformed hands | Keep hands out of prominent positions, have the subject hold something or place the hands at the edge, and write `natural anatomy` |
| Freckles and coarse texture in a Retouched subject | Mount the positive skin string from §6 and write no negations at all; if that still fails, go to targeted second-pass refinement per 10.4 |
| Garbled text in the image | When the object carries no text, state a clean frame with generous negative space; when it does, decide language and content per 1.2, and verify key text by hand or composite it in post after generation |
| Tilted horizon or building verticals | Write `level horizon, vertical lines corrected` |
| Over-processing | Write `natural dynamic range, restrained post-processing` |
| The same character inconsistent across images | Character sheet + anchor image + image-to-image pass-back, reference strength 70% |
| Style drift | The style string stays identical across the set, and a blended style keeps the same ratio throughout; confirm the prompt was not rewritten by the channel |
| Subject jumping between images in a series | Change only one variable between adjacent images, keeping light position and focal length constant throughout |
| Realistic subjects smoothed too far | Use the Realistic string from §6 |
| Image-to-image drifting too far from the reference | Raise the reference strength, and list in the prompt the parts that must stay unchanged |
| One failed generation aborting the whole batch | Record the failure separately and continue; report partial completion at the end |

---

## 13. Delivery Checklist

Files on disk match the task manifest; aspect ratio matches what was agreed, and any unsupported ratio was stated in advance; image quality takes the user's specified value or the channel's highest tier; the style string is identical across the set and any blend ratio has been restated; characters are consistent across images; no garbled text and no malformed limbs (text generated per 1.2 has been verified by hand); sequence order is correct; the prompt was not rewritten by the channel.

<a href="https://www.qydion.com"><img src="https://qydion.com/brand/16-horizontal-full-deepspace.png" alt="QYDION" width="240"></a>

# Image Generator

> [中文](README.md) | English

v1.0.0 · MIT License · 盘锦奇点科技有限公司 (Panjin QYDION Technology Co., Ltd.)

Expands a single sentence into a professional image-generation prompt you can use as-is, then generates the images one by one against a manifest.

## Features

- **Prose, not keyword soup**: the output is one connected passage of description rather than stacked keywords. Shot size, camera position, focal length, aperture, light direction and colour temperature, composition, colour ratios, materials, the style string, and quality terms are each pinned down.
- **Feeling words translated**: vague terms such as "premium feel", "moody", or "crisp" are grounded in concrete parameters — light quality, colour temperature, saturation, composition density.
- **Two modes**: building from scratch, and reference-based second-pass generation (outfit swap, background change, outpainting, local replacement, style transfer, sharpness refinement).
- **Consistent, no drift**: a character sheet plus anchor-image pass-back keeps the same face and the same style consistent across the entire batch.
- **Style blending**: blend by ratio, e.g. "70% cyberpunk + 30% impasto CG". "A bit more toward one side" steps by 10 points; if nothing is said, the split is even.
- **Controllable in-image text**: objects that inherently carry text — books, signage, packaging, UI — get text along with the image, in the language of your input; give exact wording and it is rendered character for character, wrapped in double quotes.
- **Batch delivery that closes the loop**: accounted for image by image, a single failure does not drag down the batch, and a re-run automatically skips what is already done.

## Usage

Just describe what you want in natural language:

- `A coffee product shot`
- `5 illustrations for this article, watercolour style`
- `6 expression images of this character`
- `70% cyberpunk + 30% impasto CG, an 18-year-old fox girl, silver dreadlocks, mechanical cat ears`
- `Turn this image into a night scene`
- `Swap the background to a beach, keep the person unchanged`

An unspecified count defaults to 1 image; aspect ratio defaults to 1:1; image quality and sharpness default to the highest tier. Every reference image you supply is used, with no cap, at a default reference strength of 70%.

## Installation

A **universal** Agent Skill with no client binding; it runs on any common agent (Claude Code, Codex, Cursor, OpenCode, Gemini CLI, GitHub Copilot, Windsurf, Cline, Trae, Qoder and 80+ others).

**Option 1: skills CLI**

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

**Option 2: manual**

Download this repository and copy the entire `skills/image-generator` folder into your client's skills directory (the location differs per client — check that client's documentation), then restart once.

Either way, just describe what you want in natural language afterwards — no special commands.

The skill itself is a single Markdown file: no scripts, no external dependencies, no machine-specific paths, and no binding to any image-generation provider. The generation channel is configured by the user.

## Compatibility

Not tied to any model or vendor; behaviour adapts to what the channel actually exposes. If a dedicated exclusion field exists, exclusions are written into it; if not, they become positive descriptions. If a quality-tier parameter exists, it is set; if not, quality terms go into the prompt.

By default no evaluative quality terms such as `8k` or `masterpiece` are written — current-generation models are already at their highest tier, so such terms add nothing. SD-family models still rely on them; add them yourself when using an SD-family model. Note also that a `negative_prompt` field appearing in the parameter list does not mean the model actually honours it — an aggregation platform may simply have normalised it in.

## License

MIT License · Copyright (c) 2026 盘锦奇点科技有限公司 (Panjin QYDION Technology Co., Ltd.)

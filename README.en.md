<a href="https://www.qydion.com" target="_blank" rel="noopener"><img src="https://qydion.com/brand/16-horizontal-full-deepspace.png" alt="QYDION" width="240"></a>

# ai-image-generator

> [中文](README.md) | English

## Included skill

| Skill | Description |
|---|---|
| `image-generator` | Turns a user's brief description into a professional-grade image-generation prompt and drives the generation. Supports series illustrations, document-to-image, reference-based second-pass generation, outfit and background changes, and consistent or blended styles. |

## Installation

**Option 1: skills CLI**

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

**Option 2: manual**

Download this repository and copy the entire `skills/image-generator` folder into your client's skills directory (the location differs per client — check that client's documentation), then restart once.

Either way, just describe what you want in natural language afterwards — no special commands and no client-specific invocation format.

## Repository layout

```
.
├── LICENSE
├── README.md
├── README.en.md
└── skills/
    └── image-generator/
        └── SKILL.md
```

## Dependencies

None. `SKILL.md` is a single Markdown file: no scripts, no external dependencies, no machine-specific paths, and no binding to any image-generation provider. The generation channel is configured by the user; the skill only uses image-generation capabilities already available in the current environment.

## License

[MIT](LICENSE)

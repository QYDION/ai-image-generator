<a href="https://www.qydion.com"><img src="https://qydion.com/brand/16-horizontal-full-deepspace.png" alt="QYDION" width="240"></a>

# ai-image-generator

> [中文](README.md) | English

An open-source Agent Skill from Panjin QYDION Technology Co., Ltd.: expands a brief request into a professional image-generation prompt you can submit as-is, then drives generation image by image.

## Included skill

| Skill | Description |
|---|---|
| `image-generator` | Turns a user's brief description into a professional-grade image-generation prompt and drives the generation. Supports series illustrations, document-to-image, reference-based second-pass generation, outfit and background changes, and consistent or blended styles. |

Full details: [`skills/image-generator/README.en.md`](skills/image-generator/README.en.md).

## Installation

This is a **universal** Agent Skill with no client binding. It installs and runs on any common agent that follows the Agent Skills spec — Claude Code, Codex, Cursor, OpenCode, Gemini CLI, GitHub Copilot, Windsurf, Cline, Trae, Qoder and 80+ other clients.

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

Installs into your client's skills directory; restart once to activate. After that, just describe what you want in natural language — no special commands and no client-specific invocation format.

## Repository layout

```
.
├── LICENSE
├── README.md                     Chinese documentation
├── README.en.md                  This file
└── skills/
    └── image-generator/
        ├── SKILL.md              The skill itself (the only required file)
        ├── README.md             Chinese usage guide
        ├── README.en.md          This usage guide
        └── LICENSE
```

## Dependencies

None. `SKILL.md` is a single Markdown file: no scripts, no external dependencies, no machine-specific paths, and no binding to any image-generation provider. The generation channel is configured by the user; the skill only uses image-generation capabilities already available in the current environment.

## License

[MIT](LICENSE)

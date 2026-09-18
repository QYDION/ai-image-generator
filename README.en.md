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

**Option 1: skills CLI** (works with Claude Code, Codex, Cursor and 70+ other clients)

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

**Option 2: Claude Code plugin marketplace**

```
/plugin marketplace add QYDION/ai-image-generator
/plugin install image-generator@ai-image-generator
```

**Option 3: manual**

Copy the entire `skills/image-generator` folder into your client's skills directory (for example `~/.claude/skills/`).

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json          Claude Code marketplace manifest
├── LICENSE
├── README.md                     Chinese documentation
├── README.en.md                  This file
└── skills/
    └── image-generator/
        ├── SKILL.md              The skill itself (English; the only required file)
        ├── SKILL.zh-CN.md        Chinese version of the skill
        ├── README.md             Chinese usage guide
        ├── README.en.md          This usage guide
        └── LICENSE
```

## Dependencies

None. `SKILL.md` is a single Markdown file: no scripts, no external dependencies, no machine-specific paths, and no binding to any image-generation provider. The generation channel is configured by the user; the skill only uses image-generation capabilities already available in the current environment.

## License

[MIT](LICENSE)

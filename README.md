<a href="https://www.qydion.com"><img src="skills/image-generator/brand/qydion-logo.svg" alt="QYDION" width="240"></a>

# ai-image-generator

> 中文 | [English](README.en.md)

盘锦奇点科技有限公司开源的 Agent Skill：把一句简单的需求补成一段可直接提交的专业生图提示词，并驱动逐张出图。

## 包含的技能

| 技能 | 说明 |
|---|---|
| `image-generator`（图片生成器） | 把用户的简单描述转成专业级生图提示词并驱动生成。支持系列配图、文档转图、参考图二次生成、换装换背景、画风统一与混合画风。 |

技能详情见 [`skills/image-generator/README.md`](skills/image-generator/README.md)。

## 安装

**方式一：skills CLI**（支持 Claude Code、Codex、Cursor 等 70+ 客户端）

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

**方式二：Claude Code 插件市场**

```
/plugin marketplace add QYDION/ai-image-generator
/plugin install image-generator@ai-image-generator
```

**方式三：手动**

把 `skills/image-generator` 整个文件夹复制到所用客户端的技能目录（如 `~/.claude/skills/`）。

## 目录结构

```
.
├── .claude-plugin/
│   └── marketplace.json          Claude Code 插件市场清单
├── LICENSE
├── README.md                     本文件（中文）
├── README.en.md                  英文版说明
└── skills/
    └── image-generator/
        ├── SKILL.md              技能本体（英文，唯一必需文件）
        ├── SKILL.zh-CN.md        技能本体中文版
        ├── README.md             使用说明（中文）
        ├── README.en.md          使用说明英文版
        ├── brand/                文档展示用公司 logo
        └── LICENSE
```

## 依赖

无。`SKILL.md` 是单个 Markdown 文件，不含脚本、不含外部依赖、不含机器相关路径，也不与任何生图服务商绑定。生图通道由使用者自行配置，技能只使用当前环境中已有的生图能力。

## 许可

[MIT](LICENSE)

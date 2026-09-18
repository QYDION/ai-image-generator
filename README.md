<a href="https://www.qydion.com"><img src="https://qydion.com/brand/16-horizontal-full-deepspace.png" alt="QYDION" width="240"></a>

# ai-image-generator

> 中文 | [English](README.en.md)

## 包含的技能

| 技能 | 说明 |
|---|---|
| `image-generator`（图片生成器） | 把用户的简单描述转成专业级生图提示词并驱动生成。支持系列配图、文档转图、参考图二次生成、换装换背景、画风统一与混合画风。 |

技能本体见 [`skills/image-generator/SKILL.md`](skills/image-generator/SKILL.md)。

## 安装

**方式一：skills CLI**

```bash
npx skills add QYDION/ai-image-generator --skill image-generator
```

**方式二：手动**

下载本仓库，把 `skills/image-generator` 整个文件夹复制到所用客户端的技能目录（各客户端的技能目录位置不同，以该客户端文档为准），重开一次生效。

两种方式装好后，在对话里直接用自然语言提需求即可，不需要任何特殊指令或专用调用格式。

## 目录结构

```
.
├── LICENSE
├── README.md
├── README.en.md
└── skills/
    └── image-generator/
        └── SKILL.md
```

## 依赖

无。`SKILL.md` 是单个 Markdown 文件，不含脚本、不含外部依赖、不含机器相关路径，也不与任何生图服务商绑定。生图通道由使用者自行配置，技能只使用当前环境中已有的生图能力。

## 许可

[MIT](LICENSE)

# 在 OpenClaw 中编写自定义技能 🦞

![customSkill](img/2604061410_openclaw-custom-skills.png)

## 从想法到实现

我最近帮助一位朋友理清思路，他经营几家便利店，想利用 AI 提高生产效率。我的想法是在 OpenClaw 中构建自己的自定义技能，而不是依赖预构建的功能呢？事实证明，这完全是可行的。

本指南将带你完整了解整个过程——从明确你想要构建的内容，到编写技能、测试，最后将其部署到 ClawHub。无论你是想自动化个人工作流，还是为开源社区贡献工具，这个过程都既直观又灵活。

在开始之前，一个关键的建议是：**在写任何代码前，先坐下来用一份 markdown 文档明确表述你想要技能做什么**。这份文档应该包括输入、处理过程、需要保留的元素、输出格式，甚至视觉风格。我通常会用 Perplexity 或 Claude Code 来分析这份需求文档，确保我的思路清晰。

例如，如果你想构建一个技能来润色技术博客文章，你可能会这样记录：

- **输入：** 关于技术主题的原始 markdown 草稿（例如 Linux 文件同步工作流）
- **处理：** 将草稿重写为 1,200-1,400 字的润色 zh-CN 简体中文，分为 4-5 个逻辑段落
- **保留：** 精确保留所有技术术语、代码块、文件路径和命令示例
- **输出：** 一份清晰的 markdown 文件和一张 hero image，在视觉上总结整篇文章
- **风格：** 使用一致的视觉语言（例如"清洁平面矢量插图，极简等距"）
- **分辨率：** 生成 16:9 宽高比的图像，适合作为博客头部

## 配置你的模型

在编写技能之前，确保你的语言和图像模型已正确配置。大多数情况下，OpenClaw 使用可以处理文本和图像的多模态模型。但如果你想生成高质量的图像，应该明确设置 `imageGenerationModel`。在2026.4.5 版本之前，这个参数为 `imageModel`。

首先列出你当前的模型：

```sh
openclaw models list
```

你应该看到类似这样的输出：

```sh
🦞 OpenClaw 2026.4.5 (3e72c03)
   Your task has been queued; your dignity has been deprecated.

Model                                      Input      Ctx      Local Auth  Tags
anthropic/claude-haiku-4-5-20251001        text+image 195k     no    yes   default,configured
openai/gpt-5.1-codex                       text+image 391k     no    yes   configured,alias:GPT
openai/gpt-5-mini                          text+image 391k     no    yes   configured
google/gemini-3-flash-preview              text       195k     no    yes   configured,alias:gemini-flash
moonshot/kimi-k2.5                         text+image 250k     no    yes   configured,alias:Kimi
```

在我的情况下，`openai/gpt-image-1` 被定义为 `agents.defaults.imageGenerationModel`。将你的 API 密钥存储在 `~/.openclaw/.env` 中——像对待信用卡一样对待这个文件。

```json
openclaw config get agents.defaults.imageGenerationModel

🦞 OpenClaw 2026.4.5 (3e72c03)
   I don't sleep, I just enter low-power mode and dream of clean diffs.

{
  "primary": "openai/gpt-image-1",
  "fallbacks": [
    "google/gemini-3-pro-image-preview",
    "fal/fal-ai/flux/dev"
  ]
}
```

## 编写技能定义

现在是时候编写实际的技能定义了。这是你定义输入、输出和工作流程的地方。

一个典型的 `SKILL.md` 包含两部分：YAML 前置元数据和 markdown 文档。这是一个真实的例子：

```yaml
---
name: blog-polish-eng-single-image
description: Polish a technical blog draft into a 1000–1200 word, 4–5 section en-US article, preserve technical terms/code, and generate one consistent hero image prompt.
author: Jeff Yang
version: 1.0.5
tags: [openclaw, clawhub, blog, polish, translate, markdown, images, prompts]
triggers: ["polish blog", "technical blog images", "blog draft images"]
metadata:
  openclaw:
    requires: []
    platforms: ["linux", "darwin"]
    env: []
inputSchema:
  type: object
  properties:
    draftPath:
      type: string
      description: Path to the draft markdown. Defaults to ~/.openclaw/workspace/contentDraft/latestDraft.md
    outputDir:
      type: string
      description: Directory to save outputs. Defaults to ~/.openclaw/workspace/contentPolished/
    subject:
      type: string
      description: Short subject slug used in output filename (e.g. openclaw-skills). If omitted, infer from the draft title.
    style:
      type: string
      description: Visual style phrase reused for the hero image (e.g. "clean flat vector illustration, minimal isometric").
    background:
      type: string
      description: Background phrase reused for the hero image (e.g. "white background with subtle grid").
    aspectRatioHero:
      type: string
      description: Aspect ratio for hero image (e.g. "16:9 horizontal").
  required: []
outputSchema:
  type: object
  properties:
    polishedPath:
      type: string
      description: Path to the final polished markdown file.
    imagePath:
      type: string
      description: Path of the generated hero image, or intended filename if only a prompt was produced.
    imagePrompt:
      type: string
      description: Single-line prompt for the hero image.
---

# Blog Polish Skill

This skill rewrites a technical blog draft into a polished English article and generates exactly one hero image as a PNG, using one matching hero prompt. It is intended for drafts that already contain technical content, code, commands, or product details that should be preserved while improving clarity, structure, and reading flow.

## Purpose

Use this skill when you want to turn a rough technical draft into a publishable article without losing domain-specific detail. The output should read naturally in en-US English, stay faithful to the original meaning, and keep technical terms, identifiers, code blocks, file paths, and command examples intact unless a correction is clearly needed.

This skill generates one hero image only. It does not create per-section images. The single hero image should summarize the whole article visually at a high level and should be consistent with the article’s subject and tone.

## When to use

Use this skill when the source draft is one of the following:

- A technical blog post that needs editing for clarity and flow.
- A translated draft that should be rewritten into natural en-US English.
- A markdown article that needs a better structure and cleaner sectioning.
- A post that should include one matching hero image prompt for later image generation.

Do not use this skill for short notes, changelogs, marketing copy, or posts that do not need technical preservation.

## Editing behavior

The rewrite should preserve the author’s intent while improving readability. Prefer shorter paragraphs, clearer transitions, and section headings that guide the reader through the main idea.

Rewrite the article in spoken, unofficial English that feels natural, clear, and conversational, while still preserving technical accuracy.

The skill should:

- Keep technical terms, product names, API names, file names, and command syntax accurate.
- Preserve code blocks, inline code, quoted commands, and URLs unless they are obviously wrong.
- Improve grammar, sentence flow, and article structure.
- Expand thin or fragmented notes into a coherent article when the source material supports it.
- Avoid inventing facts, results, benchmarks, or claims that are not present in the draft.

The skill should not:

- Rewrite code into prose.
- Remove essential technical details.
- Add unnecessary marketing language.
- Split the article into section images or multiple image prompts.

## Input fields

`draftPath` points to the source markdown draft. If omitted, the skill reads the default latest draft file from the workspace. This should contain the original article text, headings, and any code samples that need preservation.

`outputDir` sets where the polished markdown file and image filename should be saved. If omitted, the skill uses the default polished-content directory.

`subject` is used to build the output filename. If not provided, the skill should infer a short slug from the article title.

`style` defines the visual language for the hero image. Use one style phrase consistently so the image matches the article’s mood.

`background` defines the backdrop for the hero image. Keep it simple and reusable across posts for consistency.

`aspectRatioHero` controls the hero image shape. Typical values are `16:9 horizontal` or similar wide formats suitable for blog headers.

## Output

The skill produces one polished markdown file and one hero image prompt.

The polished file should contain:

- A cleaned-up title.
- A strong introduction.
- 3 to 5 content sections, depending on the source material.
- A concise closing section if appropriate.
- Preserved technical content where relevant.

The image output should contain:

- The image file must be written in the same directory as the markdown file and must use the same basename with a .png extension.
- One hero image filename or intended image path.
- One single-line hero prompt.
- A high-level visual summary of the article, not a section-by-section breakdown.

## Image policy

This skill intentionally generates one image only.

The image should be:

- A hero image for the whole article.
- Visually aligned with the topic and style.
- Broad enough to represent the subject without depending on individual section contents.
- Consistent with the same visual style and background settings used across posts.

The image should not be:

- A separate illustration for each section.
- A collage of unrelated concepts.
- Overly literal if the topic is abstract.
- Packed with too many technical labels or small details.

## Workflow

1. Resolve the draft and output paths.
2. Read the markdown draft.
3. Extract the title and basic structure.
4. Rewrite the article into polished en-US prose.
5. Save the polished markdown file.
6. Create one hero image prompt only.
7. Return the final file path, hero image path, and image prompt.

## Constraints

Maintain the meaning of the original draft. If the source contains code snippets, commands, paths, or configuration examples, keep them intact and formatted correctly. If the draft is sparse, improve clarity and organization, but do not fabricate missing technical content.

Keep the article focused and practical. Prefer specific explanations over generic filler. If the article has a narrow technical subject, the hero image should stay broad and conceptual rather than trying to depict every detail.

## Example usage

A draft about a Linux file synchronization workflow might be polished into a clear article with headings such as introduction, setup, common pitfalls, and conclusion. The hero image prompt could describe a clean technical illustration showing a laptop, file paths, and subtle sync arrows, but only as one overall image for the post.

## Implementation notes

The workflow should emit a single structured output object with these fields:

- `polishedPath`
- `imagePath`
- `imagePrompt`

The image must be written as a PNG file. It must use the same basename as the markdown file and be saved in the same directory as the markdown file. The skill should not emit arrays of images or prompts. It should not reference per-section image generation in the description, schema, or workflow.

Generate the image using OpenClaw’s default image model (`agents.defaults.imageModel`) unless an explicit image generation model is provided by the environment.
```

结构很简洁直观：名称、描述、版本、输入参数和输出结构。关于你的技能做什么，要保持清晰而具体。

## 验证和部署

在发布之前，使用 Perplexity 或 Claude 之类的 AI 工具根据 OpenClaw 和 ClawHub 标准审核你的 `SKILL.md`。给它这个提示：

```
审核这个 SKILL.md 是否符合 OpenClaw 和 ClawHub 标准。
检查 JSON schema、描述和工作流逻辑。
建议对清晰度或结构进行任何改进。
```

迭代几次，直到你确信技能是没有瑕疵的。现在的小错误会节省以后的调试时间。

一旦你对 `SKILL.md` 感到满意，安装它到本地：

```sh
cd ~/.openclaw/workspace/skills
clawhub install your-skill-name
```

列出已安装的技能来确认：

```sh
clawhub list
```

你应该在输出中看到你的技能。现在从你配置的频道（Discord、WhatsApp、Telegram 等）测试它：

```
trigger "your-skill-name"
```

在你的工作区文件夹中观看输出，检查润色的 markdown 和生成的图像是否出现在你期望的位置。

当你确信你的技能有效时，将其上传到 ClawHub：

```sh
clawhub publish ~/.openclaw/workspace/skills/your-skill-name
```

ClawHub 将验证 `SKILL.md` 并使其对社区可用。不要担心如果它将 shell 脚本标记为"Suspicious" —— 这只是一个警告。只要你信任你自己的代码，就没有问题。

发布后，任何人都可以用以下命令安装你的技能：

```sh
clawhub install your-username/your-skill-name
```

## 总结

在 OpenClaw 中构建自定义技能一旦你理解了工作流就很直观：清楚地思考你的目标，编写一个与标准相匹配的稳健 `SKILL.md`，验证它，本地测试，当你准备好时发布。整个过程，从想法到社区共享工具，通常需要不到一小时。

你的技能成为你的生产力工具包的一部分——如果它们很好，它们也成为社区工具包的一部分。

---

tag: #openclaw #ai #opensource #linux #clawhub #skill

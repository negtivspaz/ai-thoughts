# 怎么从 GitHub 安装自定义 Hermes Skill

![hermes-skill](../imgs/260429-hermes-skill.png)

本指南按端到端流程说明（已实际验证）：把 GitHub 上的自定义 `SKILL.md` 复制到 `~/.hermes/skills`，确认已列出，再运行，并可按需自动化。采用这种目录布局时，Hermes 不需要单独的 register 或 install 命令。

## 前置条件

- 已配置至少一个频道的 Hermes
- 已接入的模型（本文示例使用通过 OpenRouter 的 Nvidia Nemotron **免费**版；任意兼容模型均可）

## 背景

我在 GitHub 上发布了一份自定义 `SKILL.md`，并希望有一条可靠的 Hermes 安装路径。读过若干文档后仍未找到一条连贯的可操作说明，于是与 AI 助手一起跑通流程，并把结果整理成文。

本文是该次会话的净化版，便于你按相同步骤操作，减少试错。

下文 shell 示例已在真实环境中验证；请将仓库、路径、分支与本地技能 slug 换成你自己的。

## 对话上下文（为什么会有这篇文档）

最初向助手提出的请求：

> i've written a custom SKILL.md that has been published at my Github account. Please guide me how to install it either through chat here or in command line

会话涵盖内容：

- 明确输入项：仓库 URL、技能名、`SKILL.md` 路径。
- 示例 URL：`https://github.com/j3ffyang/ai-custom-skills/tree/main/hermes/ai-newsletter-prompt`。
- 拉取并审阅 raw `SKILL.md`，再复制到 `~/.hermes/skills`。
- 将技能安装为 `ai-newsletter-daily`，并用 `hermes skills list` 确认。
- 确认输出：`newsletter_items`、`markdown_newsletter`、`json_newsletter`；持久化与投递仍由你的工作流负责（`write_file`、`send_message`、cron 等）。

源仓库：https://github.com/j3ffyang/ai-custom-skills

---

## 1. 从 GitHub 获取自定义 SKILL.md

复制到 `~/.hermes/skills` 之前，你需要两样东西：

1. **`SKILL.md` 的 raw URL**。最简单：在 GitHub 打开 `SKILL.md`，点 **Raw**，复制地址栏。也可自行拼  
   `https://raw.githubusercontent.com/<owner>/<repo>/<ref>/<path-to-SKILL.md>`  
   （`<ref>` 一般为 `main` 或 `master`；若要固定版本可用标签或提交 SHA）。

2. **本地技能 slug** — 小写标识（连字符或下划线），对应路径 `~/.hermes/skills/<slug>/SKILL.md`。slug **由你决定**，不必与仓库里的文件夹名一致；本文仓库路径为 `hermes/ai-newsletter-prompt/`，安装目录名为 `ai-newsletter-daily`。

下面示例使用我本人维护的一份自定义 `SKILL.md`。

| 项目 | 示例（本文） |
|------|----------------------|
| 所有者 / 仓库 | `j3ffyang/ai-custom-skills` |
| 仓库内 `SKILL.md` 路径 | `hermes/ai-newsletter-prompt/SKILL.md` |
| 分支 / 标签 / 提交 | `main`（默认） |
| 本地 slug | `ai-newsletter-daily` |

先下载到临时目录，再执行 §2 安装：

```sh
mkdir -p /tmp/hermes-skill
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/j3ffyang/ai-custom-skills/main/hermes/ai-newsletter-prompt/SKILL.md"
```

**克隆备选：** 若使用 `git clone`，可用 `cp` 从工作区复制 `SKILL.md`，代替 `curl` — 适合私有仓库或需要旁邻文件时。

## 2. 在本地安装技能

将 `SKILL.md` 复制到 `~/.hermes/skills/<skill-name>/SKILL.md`。这就是完整安装：Hermes 从此目录加载技能，本地自定义技能不需要额外的 CLI 注册步骤。本例中 slug 为 `ai-newsletter-daily`，故路径为 `~/.hermes/skills/ai-newsletter-daily/SKILL.md`。

## 3. 验证安装

执行：

```sh
hermes skills list
```

自定义技能应出现在输出中。

```sh
hermes skills list
                                   Installed Skills                                   
┏━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━┓
┃ Name                    ┃ Category             ┃ Source  ┃ Trust   ┃ Status  ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━┩
│ ai-newsletter-daily     │                      │ local   │ local   │ enabled │
```

## 4. 环境变量（示例：ai-newsletter-daily）

若仅通过复制到 `~/.hermes/skills` 安装技能，`hermes skills inspect` 不会覆盖该本地技能，因此无法用 inspect 核对 `required_environment_variables` 或相关元数据。

本文使用的示例 `SKILL.md` 在文件中未定义 `required_environment_variables`。运行时所需的密钥仍取决于技能实际调用的接口；请根据 `SKILL.md` 正文、README 或你自己的笔记来设置。

`ai-newsletter-daily` 需要：

| 变量 | 用途 | 如何设置 |
|----------|---------|------------|
| `BRAVE_API_KEY` | 通过 Brave 进行网页搜索 | `export BRAVE_API_KEY=your_brave_key` |
| `FIRECRAWL_API_KEY` | 通过 Firecrawl 抓取页面 | `export FIRECRAWL_API_KEY=your_firecrawl_key` |

请写入 shell 配置文件（`~/.bashrc`、`~/.zshrc` 等）持久化，或在运行 Hermes 的会话中 export。

```bash
export BRAVE_API_KEY=xxxxxxxxxxxxxxxxxxxx
export FIRECRAWL_API_KEY=yyyyyyyyyyyyyyyyyyyy
```

## 6. 使用 skill

Hermes 会根据用户意图，在终端 TUI 与桥接的聊天应用中路由到技能。除非你的部署或桥接另有约定，否则不需要特殊语法。

**Hermes TUI**

- 用自然语言描述任务（例如技能与代码审查相关时，可说：“审查我当前目录里的代码”）。
- 若 `SKILL.md` 定义了激活短语或关键词，使用它们以便路由命中技能的触发条件。
- 依赖文件或上下文的技能，通常在询问当前打开的缓冲区或你关心的路径时会有响应。

**消息通道（如 WhatsApp、Telegram）**

- 在群组中，桥接有时需要 @ 提及（例如 `@Hermes …`），代理才会处理该消息。
- 在私聊中，与技能目的相符的纯文本通常即可。
- 部分桥接使用命令前缀（`!`、`/` 等）；请在你的桥接与 Hermes 频道设置中确认。

**若技能未运行**

- 将你的措辞与 `SKILL.md` 中的 **Triggers**（或等价章节）对照。
- 发送测试消息时观察 Hermes 日志中的入站文本与意图路由。

## 6. 用 Cron 自动化（可选）

若要按固定节奏生成并投递输出（例如每日简报），可配置 Hermes 的 cron 任务，或使用外部调度器调用与你交互时相同的工作流。具体参数取决于 Hermes 版本与频道配置；cron 相关语法请用 `hermes --help` 及安装文档核对。

## 7. 日后更新 Skill

重新获取 raw `SKILL.md`（§1），替换 `~/.hermes/skills/<skill-name>/SKILL.md` 下的文件，再用 `hermes skills list` 确认。

## 8. 速查备忘

| 步骤 | 操作 |
|------|--------|
| 1 | Raw URL（GitHub **Raw** 或 `raw.githubusercontent.com`…）+ 选定本地 `<slug>`；`mkdir` + `curl` 或从 `git clone` 复制 |
| 2 | 复制到 `~/.hermes/skills/<slug>/SKILL.md` |
| 3 | `hermes skills list` 验证 |
| 4 | 导出技能所需环境变量（依据 `SKILL.md`/文档；本地复制无法用 `inspect`） |
| 5 | 通过 TUI 或频道调用；措辞与触发条件对齐 |
| 6 | 可选：cron 或调度器 |
| 7 | 更新：重复 §1 下载并替换文件 |

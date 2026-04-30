# 怎么从 GitHub 安装自定义 Hermes Skill

这篇是一个已经跑通、可复现的完整流程：把 GitHub 上的自定义 `SKILL.md` 装进 Hermes，验证安装结果，运行它，最后还能做自动化。

## 前置条件

- 你已经有 Hermes，并且 channel 配好了
- 你这边已经接了 Nvidia Nemotron **免费**模型（我是通过 OpenRouter 接的，你也可以换成别的模型）

## 背景

我之前把一个自定义 `SKILL.md` 发到了 GitHub，然后需要一条稳定、可落地的安装路径把它装到 Hermes 里。看了不少文档，还是觉得流程不够顺，所以我就直接用一轮 AI 助手对话，把整套步骤一步一步跑通了。

这篇文章就是那次会话整理后的技术版。目的很简单：让你少踩坑、少试错，直接按步骤就能复现。

这里所有技术命令都在真实环境里测过，确认可用。文中的代码块保持和验证时完全一致。

## 对话上下文（为什么会有这篇文档）

我当时给助手的原始提问：

> i've written a custom SKILL.md that has been published at my Github account. Please guide me how to install it either through chat here or in command line

对话结论大概是这样：

- 先把关键输入信息确认清楚：仓库 URL、技能名、`SKILL.md` 在仓库里的路径。
- 我提供的 URL 是：`https://github.com/j3ffyang/ai-custom-skills/tree/main/hermes/ai-newsletter-prompt`。
- 然后把原始 `SKILL.md` 拉下来，检查内容，再在本地注册。
- skill 以 `ai-newsletter-daily` 的名字安装成功，并且能通过查看/列表命令验证。
- 输出行为也确认了：这个 skill 会返回 `newsletter_items`、`markdown_newsletter`、`json_newsletter`；至于存哪里、怎么发（`write_file`、`send_message`、cron 等），由你自己的工作流决定。

---

## 1. 先准备必要信息

| 项目 | 你需要准备什么 | 你的实际示例 |
|------|---------------|------------------------|
| GitHub 仓库 | Owner / repo 名（或完整 URL） | j3ffyang/ai-custom-skills |
| SKILL.md 路径 | 仓库内相对路径（不在根目录时） | hermes/ai-newsletter-prompt/SKILL.md |
| Skill 名称 | 本地要用的小写标识（支持连字符/下划线） | ai-newsletter-daily |
| 分支 / tag / commit（可选） | 默认 main/master；不同的话要指定 | main（默认） |


## 2. 拉取原始 SKILL.md 文件

用 `curl`（或者 `wget`）从 GitHub 下载 raw 文件。把占位符换成你自己的值就行。

```bash
# Create a temporary directory (optional but tidy)
mkdir -p /tmp/hermes-skill

# Download the raw SKILL.md
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/<OWNER>/<REPO>/<BRANCH>/<PATH_TO_SKILL>"
```

你的实际示例：

```bash
mkdir -p /tmp/hermes-skill
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/j3ffyang/ai-custom-skills/main/hermes/ai-newsletter-prompt/SKILL.md"
```


可选：验证下载内容

```bash
head -20 /tmp/hermes-skill/SKILL.md   # should show the YAML front‑matter
```

## 3. 在 Hermes 里注册 Skill

Hermes 提供了 `skill_manage` 工具。如果这个 skill 还不存在，用 create；如果已经有了并且要更新，用 patch 或 edit。

```bash
# Install (create) the skill
hermes skill manage create \
  --name <SKILL_NAME> \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

示例

```bash
hermes skill manage create \
  --name ai-newsletter-daily \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

你应该会看到类似这样的成功提示：

```sh
Success: Skill 'ai-newsletter-daily' created.
```

## 4. 验证安装结果

```bash
hermes skill view <SKILL_NAME>
```

示例

```bash
hermes skill view ai-newsletter-daily
```

输出里会包含 YAML front‑matter、描述、标签和完整流程，能用来确认 skill 已经正确加载。


## 5. 设置必需环境变量（如果有）

很多 Hermes skill 会在 metadata 里声明需要的环境变量。你可以在 `skill view` 输出里找 `required_environment_variables` 这类字段。

`ai-newsletter-daily` 需要这两个：

| 变量 | 用途 | 怎么设置 |
|----------|---------|------------|
| BRAVE_API_KEY | 通过 Brave 做网页搜索 | export BRAVE_API_KEY=your_brave_key |
| FIRECRAWL_API_KEY | 通过 Firecrawl 抓网页内容 | export FIRECRAWL_API_KEY=your_firecrawl_key |

把这些 export 写到你的 shell profile（`~/.bashrc`、`~/.zshrc` 等），或者在当前会话里临时导出也行。

```bash
export BRAVE_API_KEY=xxxxxxxxxxxxxxxxxxxx
export FIRECRAWL_API_KEY=yyyyyyyyyyyyyyyyyyyy
```


## 6. 使用这个 Skill

这个 skill 运行后会返回 3 个值：

- newsletter_items – 文章对象列表  
- markdown_newsletter – 可直接发送的 Markdown newsletter
- json_newsletter – 给下游流程用的 JSON payload

示例：交互式运行 skill 并保存 markdown

```bash
# Run the skill (you can pass overrides if desired)
result=$(hermes skill run ai-newsletter-daily \
          --set target_news_count=15 \
          --set search_query="latest AI news today")

# Extract the markdown part (assuming the skill returns a JSON map)
markdown=$(echo "$result" | jq -r .markdown_newsletter)

# Save to a dated file
hermes write_file \
  --path "~/ai-newsletters/$(date +%F)_newsletter.md" \
  --content "$markdown"
```


示例：通过 WhatsApp 发送 newsletter（或者发到别的平台）

```bash
hermes send_message \
  --target whatsapp \
  --message "$markdown"
```

## 7. 用 Cron 自动化（可选）

如果你想定时生成并发送 newsletter，可以建一个 Hermes cron job。

先创建 cron job 定义（YAML）

```yaml
# ~/.hermes/cron/jobs/daily-ai-newsletter.yaml
action: create
name: daily-ai-newsletter
schedule: "0 8 * * *"          # 08:00 every day
prompt: |
  # Run the skill and capture outputs
  result=$(hermes skill run ai-newsletter-daily \
                --set target_news_count=15 \
                --set search_query="latest AI news today")
  markdown=$(echo "$result" | jq -r .markdown_newsletter)

  # Save locally
  hermes write_file --path "~/ai-newsletters/$(date +%F).md" --content "$markdown"

  # Deliver (example: WhatsApp)
  hermes send_message --target whatsapp --message "$markdown"
skills: []          # no extra skills needed
deliver: local      # keep file locally; send_message handles delivery
```

注册 cron job：

```bash
hermes cronjob create --file ~/.hermes/cron/jobs/daily-ai-newsletter.yaml
```

这样它每天早上就会自动跑：生成 newsletter、落地一份 markdown、再发到 WhatsApp（或你指定的平台）。


## 8. 后续更新 Skill

如果你之后把新版本 push 到 GitHub：

1. 先重复第 2 步，拉最新 `SKILL.md`。  
2. 用 patch（小改动推荐）或者 edit（整份替换）：

```bash
hermes skill manage patch \
  --name ai-newsletter-daily \
  --old_string "<old_unique_snippet>" \
  --new_string "<new_content>"
```

或者整份替换：

```bash
hermes skill manage edit \
  --name ai-newsletter-daily \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

3. 最后用 skill view 再确认一次。


## 9. 快速参考 Cheat‑Sheet

```bash
# 1. Download
mkdir -p /tmp/hermes-skill
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/<OWNER>/<REPO>/<BRANCH>/<PATH_TO_SKILL>"

# 2. Install (create)
hermes skill manage create \
  --name <SKILL_NAME> \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"

# 3. Verify
hermes skill view <SKILL_NAME>

# 4. Set env vars (example)
export BRAVE_API_KEY=...
export FIRECRAWL_API_KEY=...

# 5. Use skill (save markdown)
result=$(hermes skill run <SKILL_NAME> --set target_news_count=15)
markdown=$(echo "$result" | jq -r .markdown_newsletter)
hermes write_file --path "~/news/$(date +%F).md" --content "$markdown"

# 6. Optional: cron job (see YAML above)
hermes cronjob create --file ~/.hermes/cron/jobs/<JOB_NAME>.yaml
```

把 \<OWNER>、\<REPO>、\<BRANCH>、<PATH_TO_SKILL>、<SKILL_NAME> 换成你的实际值即可。

## 10. 安装验证快照

下面这段终端输出能证明 `ai-newsletter-daily` 已经作为本地 skill 成功安装到 Hermes：

```sh
hermes skills list
                             Installed Skills                             
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━┓
┃ Name                        ┃ Category             ┃ Source  ┃ Trust   ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━┩
│ ai-newsletter-daily         │                      │ local   │ local   │
│ dogfood                     │                      │ builtin │ builtin │
│ claude-code                 │ autonomous-ai-agents │ builtin │ builtin │
│ codex                       │ autonomous-ai-agents │ builtin │ builtin │
│ hermes-agent                │ autonomous-ai-agents │ builtin │ builtin │
│ ...                         │ ...                  │ builtin │ builtin │
└─────────────────────────────┴──────────────────────┴─────────┴─────────┘
0 hub-installed, 70 builtin, 1 local
```

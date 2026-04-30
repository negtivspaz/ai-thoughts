# How to Install a Custom Hermes Skill from GitHub

![hermes-skill](../imgs/260429-hermes-skill.png)

This guide documents a tested, end-to-end workflow to install a custom `SKILL.md` from GitHub into Hermes, verify the installation, run the skill, and automate it.

## Prerequisites

- Hermes with a channel setup
- Integrated with Nvidia Nemotron **free** model (through OpenRouter in my runtime. Of course you can use different model)

## Background

I published a custom `SKILL.md` on GitHub and needed a reliable installation path for Hermes. After going through multiple docs and still not finding a clear operational flow, I used an AI assistant session to execute the full procedure step by step.

This article is the cleaned and structured output of that session. It exists so anyone can repeat the same process quickly with fewer trial-and-error steps.

All technical commands in this document were tested in an actual environment and proved working. Code blocks are preserved exactly as used during validation.

## Conversation Context (Why this guide exists)

Original ask to the assistant:

> i've written a custom SKILL.md that has been published at my Github account. Please guide me how to install it either through chat here or in command line

Conversation summary:

- The required inputs were clarified first: repository URL, skill name, and path to `SKILL.md`.
- Provided URL: `https://github.com/j3ffyang/ai-custom-skills/tree/main/hermes/ai-newsletter-prompt`.
- The raw `SKILL.md` was fetched, inspected, and registered locally.
- The skill was installed as `ai-newsletter-daily` and verified with skill inspection/listing.
- The output behavior was confirmed: the skill returns `newsletter_items`, `markdown_newsletter`, and `json_newsletter`; persistence and delivery are handled by your own workflow (`write_file`, `send_message`, cron, etc.).

The actual SKILL.md > https://github.com/j3ffyang/ai-custom-skills

---

## 1. Gather the Required Information

| Item | What you need | Example from your case |
|------|---------------|------------------------|
| GitHub repo | Owner / repo name (or full URL) | j3ffyang/ai-custom-skills |
| Path to SKILL.md | Relative path inside the repo (if not at root) | hermes/ai-newsletter-prompt/SKILL.md |
| Skill name | Lower‑case identifier you want to use locally (hyphens/underscores allowed) | ai-newsletter-daily |
| Branch / tag / commit (optional) | Default is main/master; specify if different | main (implicit) |


## 2. Fetch the Raw SKILL.md File

Use `curl` (or `wget`) to download the raw file from GitHub. Replace the placeholders with your own values.

```bash
# Create a temporary directory (optional but tidy)
mkdir -p /tmp/hermes-skill

# Download the raw SKILL.md
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/<OWNER>/<REPO>/<BRANCH>/<PATH_TO_SKILL>"
```

Example with your data

```bash
mkdir -p /tmp/hermes-skill
curl -sL -o /tmp/hermes-skill/SKILL.md \
  "https://raw.githubusercontent.com/j3ffyang/ai-custom-skills/main/hermes/ai-newsletter-prompt/SKILL.md"
```


Verify the download (optional):

```bash
head -20 /tmp/hermes-skill/SKILL.md   # should show the YAML front‑matter
```

## 3. Register the Skill with Hermes

Hermes provides the skill_manage tool. If the skill does not yet exist, use create; if it exists and you want to update it, use patch or edit.

```bash
# Install (create) the skill
hermes skill manage create \
  --name <SKILL_NAME> \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

Example

```bash
hermes skill manage create \
  --name ai-newsletter-daily \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

You should see a success message similar to:

```sh
Success: Skill 'ai-newsletter-daily' created.
```

## 4. Verify the Installation

```bash
hermes skill view <SKILL_NAME>
```

Example

```bash
hermes skill view ai-newsletter-daily
```

The output will show the YAML front‑matter, description, tags, and the full procedure—confirming the skill is loaded correctly.


## 5. Set Required Environment Variables (if any)

Many Hermes skills declare needed environment variables in their metadata. Check the output of skill view for a section like required_environment_variables.

For the ai-newsletter-daily skill you need:

| Variable | Purpose | How to set |
|----------|---------|------------|
| BRAVE_API_KEY | Web search via Brave | export BRAVE_API_KEY=your_brave_key |
| FIRECRAWL_API_KEY | Web page fetching via Firecrawl | export FIRECRAWL_API_KEY=your_firecrawl_key |

Add these exports to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.) or export them in the session where you’ll run the skill.

```bash
export BRAVE_API_KEY=xxxxxxxxxxxxxxxxxxxx
export FIRECRAWL_API_KEY=yyyyyyyyyyyyyyyyyyyy
```


## 6. Using the Skill

The skill itself returns three values when invoked:

- newsletter_items – list of article objects  
- markdown_newsletter – ready‑to‑send Markdown formatted newsletter
- json_newsletter – JSON payload for downstream processing

Example: Run the skill interactively and save the markdown

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


Example: Send the newsletter via WhatsApp (or any other platform)

```bash
hermes send_message \
  --target whatsapp \
  --message "$markdown"
```

## 7. Automate with a Cron Job (Optional)

If you want the newsletter generated and delivered on a schedule, create a Hermes cron job.

Create a cron job definition (YAML)

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

Register the cron job

```bash
hermes cronjob create --file ~/.hermes/cron/jobs/daily-ai-newsletter.yaml
```

The job will now run each morning, generate the newsletter, store a markdown copy, and push it to WhatsApp (or whichever target you choose).


## 8. Updating the Skill Later

If you push a new version to GitHub:

1. Repeat step 2 to download the updated SKILL.md.  
2. Use patch (preferred for small changes) or edit (full rewrite):

```bash
hermes skill manage patch \
  --name ai-newsletter-daily \
  --old_string "<old_unique_snippet>" \
  --new_string "<new_content>"
```

   Or replace the whole file:

```bash
hermes skill manage edit \
  --name ai-newsletter-daily \
  --content "$(cat /tmp/hermes-skill/SKILL.md)"
```

3. Verify with skill view.


## 9. Quick Reference Cheat‑Sheet

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

Replace \<OWNER>, \<REPO>, \<BRANCH>, <PATH_TO_SKILL>, and <SKILL_NAME> with your actual values.

## 10. Verified Installation Snapshot

The following terminal output confirms `ai-newsletter-daily` is installed locally in Hermes:

```sh
hermes skills list
                             Installed Skills                             
┏━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━┓
┃ Name                  ┃ Category             ┃ Source  ┃ Trust   ┃
┡━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━┩
│ ai-newsletter-daily   │                      │ local   │ local   │
│ dogfood               │                      │ builtin │ builtin │
│ claude-code           │ autonomous-ai-agents │ builtin │ builtin │
│ codex                 │ autonomous-ai-agents │ builtin │ builtin │
│ hermes-agent          │ autonomous-ai-agents │ builtin │ builtin │
│ ...                   │ ...                  │ builtin │ builtin │
└───────────────────────┴──────────────────────┴─────────┴─────────┘
0 hub-installed, 70 builtin, 1 local
```

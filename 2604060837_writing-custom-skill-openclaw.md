# Writing Your Own Custom Skill in OpenClaw 🦞

## Introduction

Building a custom skill for OpenClaw turns a repeated workflow into a single, reproducible command. Whether you’re automating content polishing, integrating an internal API, or wiring up a device, a well-written SKILL.md captures intent, inputs, and the exact steps the agent should take. This article walks through the high-level thinking and practical details I used when teaching a friend how to write their own SKILL.md.

## 1. Start with the goal, not the implementation

Before you write code or a manifest, write a one-page description of the problem you want the skill to solve. Put it in plain English and keep it focused. For example, my friend — who runs several convenience stores — wanted a repeatable way to generate store-specific daily checklists and reorder suggestions. The goal wasn’t "write a script"; the goal was: "reduce morning prep time and avoid stockouts." 

Capture: who benefits, what success looks like, and what inputs are required. This single markdown becomes the canonical source-of-truth you’ll use to design the SKILL.md.

## 2. Preserve technical details (models, keys, and paths)

OpenClaw separates LLM/text models from image models. If your workflow needs images, explicitly configure an imageModel rather than assuming multimodal defaults. Keep configuration bits in a protected file like ~/.openclaw/.env and document them in your skill’s inputSchema, not in the code comments.

Example checklist for configuration:

- Confirm which models are available with openclaw models list.
- Store API keys in ~/.openclaw/.env and protect that file.
- Include default model aliases in your SKILL.md’s metadata so other users know what to change.

Keep commands and code blocks verbatim in the SKILL.md; the agent treats those as executable examples. Don’t hide important command flags inside paragraphs — put them in fenced code blocks so they’re preserved.

## 3. Think architecturally: write the workflow as steps

Break the skill into a short workflow: read input, validate, transform (LLM step), write output, and return a structured result. That approach keeps the skill easy to test and debug.

A minimal workflow looks like:

- init: set defaults and timestamp
- verify_input: fail early if files are missing
- read_draft: load source content
- polish: LLM-driven rewrite preserving code blocks
- prepare_output: name the file with a timestamp + slug
- write_file: save polished markdown
- finalize: emit the path(s) to the caller

Design each step so it’s idempotent and easy to rerun. If a step may take a long time (image generation, external API calls), indicate that in the skill and provide progress or logs.

## 4. Preserve fidelity when polishing content

When you use an LLM to polish drafts, resist the urge to rewrite technical examples or fabricate missing facts. Your goal is clarity and flow, not invention.

Practices that help:

- Keep code blocks and command examples exactly as the author wrote them unless a clear syntax error would break reproducibility.
- Shorten long paragraphs and add transitions; expand thin notes only when the draft supports it.
- Avoid marketing language — prefer concrete steps and examples.

Example command (preserve this block exactly):

```sh
openclaw models list
```

That exact command is what readers will type; changing it could cause confusion.

## 5. File naming, output paths, and reproducibility

Use deterministic naming that includes a short timestamp and a slug derived from the title. This avoids accidental overwrites and makes it easy to find historical versions.

My convention: yymmddHHMM_slug.md and a matching hero image yymmddHHMM_slug.png saved to the same output directory. For most blog workflows, a single hero image is enough — don’t generate one image per section unless the post explicitly needs it.

Example structure:

- ~/.openclaw/workspace/contentDraft/latestDraft.md (input)
- ~/.openclaw/workspace/contentPolished/2604060837_writing-custom-skill-openclaw.md (output)
- ~/.openclaw/workspace/contentPolished/2604060837_writing-custom-skill-openclaw.png (hero image)

## 6. SKILL.md structure and metadata

A robust SKILL.md contains both machine-friendly metadata and human-facing guidance. At minimum include:

- name, description, author, version
- inputSchema: required inputs and defaults
- outputSchema: expected outputs
- workflow: a small, repeatable set of steps
- usage examples and CLI snippets

Make defaults reasonable but configurable. If the skill touches external services (APIs, file systems), call that out and show how to configure credentials.

## 7. Testing and install

Before publishing a skill to ClawHub or sharing it, test the happy path and a few failure modes:

- Missing draft file → error quickly with an actionable message.
- Bad or missing API keys → clear guidance where to put them (~/.openclaw/.env).
- Partial outputs → leave artifacts with a timestamp so users can inspect intermediate results.

To install locally:

```sh
cd ~/.openclaw/workspace/skills
clawhub install your-skill-name
clawhub list
```

## Conclusion

Good skills make complex tasks trivial. Start with a clear goal, document required inputs and configuration, preserve technical fidelity, and structure the workflow into small, testable steps. Use timestamped filenames and a single hero image to keep outputs predictable and reproducible.

If you want, I can now:

- Save this polished article to ~/.openclaw/workspace/contentPolished/ (done),
- Generate a single-line hero image prompt for the post (below), and
- Produce the hero PNG using the workspace image model (ask me to generate it).

---

Saved as: 2604060837_writing-custom-skill-openclaw.md

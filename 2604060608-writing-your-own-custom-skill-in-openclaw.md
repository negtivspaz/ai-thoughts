---
title: "Writing Your Own Custom Skill in OpenClaw"
---

## Introduction

Creating a custom skill for OpenClaw lets you turn domain knowledge and repeatable workflows into first-class agent behavior. Whether you want to automate store operations, publish consistent blog posts, or integrate a bespoke toolchain, a well-formed SKILL.md documents intent, inputs, and a safe workflow that OpenClaw can run or expose to chat channels.

This guide explains the essential pieces of a skill: the input/output schema, the workflow steps, environment expectations, and how to wire image and model settings. It also covers safe handling of secrets and deployment practices so your skill remains reproducible and secure.

## Why write a SKILL.md?

A SKILL.md is the contract between your idea and the agent runtime. It defines what the skill does, what inputs it expects, and what the result looks like. When you express your automation as a skill you get several benefits:

- Reproducibility: the skill lists the exact steps an agent should follow, making behavior deterministic.
- Safety: the skill can constrain filesystem access, environment variables, and tool usage.
- Discoverability: installed skills appear in lists and can be triggered from chat channels.
- Reuse: a well-documented skill is easy to share and re-use across projects.

If you’ve ever repeated the same sequence of commands or prompts, turning those steps into a SKILL.md will save time and reduce errors.

## Core parts of a skill

Most skills follow the same structure. Key sections include:

- Metadata and schema: name, description, author, version, and a machine-readable input/output schema.
- Workflow: an ordered list of steps (init, verify_input, read_draft, polish, write_file, finalize) that the agent executes. Each step should be small, testable, and safe.
- Constraints: explicit rules about file paths, secrets, and allowed tools. These make policies auditable and portable.
- Examples: sample inputs and expected outputs so users and CI systems can validate the skill.

A clear schema makes it simple for the skill to be invoked by other tools and for tests to assert correct behavior.

## Configuring models and images

If your skill generates images or uses multi-modal models, declare the expected model names and how to find credentials (for example, keys stored in ~/.openclaw/.env). Do not hard-code secrets into SKILL.md — instead, read them from runtime configuration and document the expected environment variables.

For images, define the desired aspect ratio, pixel dimensions, and quality. If you only need one hero image, say so; if you need per-section images, state that clearly and keep file naming deterministic (e.g., {ts}_{slug}_section_{n}.png).

Security note: keep ~/.openclaw/.env protected. SKILL.md should instruct users to add API keys to that file rather than pasting them in chat.

## A small example workflow (practical)

1. init — set defaults and timestamp
2. verify_input — ensure the draft exists
3. read_draft — load the markdown into memory
4. polish — call LLM to rewrite and restructure the content while preserving code blocks
5. generate_image_prompt — synthesize one-line hero prompt (do not call image APIs directly unless allowed)
6. write_file — save polished markdown to contentPolished/
7. finalize — return a JSON object with paths and the prompt

This pattern isolates LLM work to a single step, so it’s trivial to test locally or run in CI.

## Best practices and safety

- Keep steps idempotent: rerunning a skill should not cause destructive side effects.
- Use deterministic filenames with timestamps and slugs to avoid accidental overwrites.
- Preserve author intent: do not invent facts or change code semantics when polishing technical content.
- Provide a dry-run mode in the skill so users can preview prompts and filenames without generating images or writing files.

## Installing and testing

Install into your workspace under ~/.openclaw/workspace/skills and run local checks. Use `clawhub` to package or publish your skill. Test by running the workflow with a sample draft and verifying the polished output matches expectations.

## Conclusion

SKILL.md turns repeatable human workflows into reliable agent behavior. Start small, keep your steps clear, and document inputs/outputs. When in doubt, prefer transparency and safety: explicit environment expectations, deterministic names, and a dry-run mode will make your skills robust and trustworthy.

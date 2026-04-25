pair program, coach juniors, delegate work, help seniors avoid usual traps - you get better results. 



https://www.reddit.com/r/AskVibecoders/comments/1sp5dnd/every_claude_tool_worth_knowing_dont_miss_out_by/?share_id=FIih4wu08I_yo__KDJhvI&utm_content=1&utm_medium=android_app&utm_name=androidcss&utm_source=share&utm_term=3

Go to AskVibecoders
r/AskVibecoders
•
3d ago
No_Tomatillo2993
Every Claude Tool Worth Knowing, Don't Miss out by Not Using it.

Here's everything worth knowing about Claude Toolings, organized by how much it actually changes your output.

Core interface tools

Start here before anything else.

Projects

Every conversation without a Project starts from zero. Claude knows nothing about you, your work, your voice, or your standards. You re-explain the same context you've already explained dozens of times.

Projects fix that. Spend 20-30 minutes building context files and uploading them to a dedicated Project. Every session after that starts informed.

Skills

A Skill is a markdown file with a pre-loaded instruction set. Call it in any chat or Project and Claude follows the instructions inside it.

Example: build a Brand Voice Skill with your tone, audience, banned words, and formatting rules. Instead of re-explaining all of that, you say: "Use my Brand Voice Skill to write this." Done.

To build one: Customize → Skills → Enable Skill-Creator. Then tell Claude: "I want to build a Skill for [x] workflow, help me build it."

Memory

Claude builds memories about you across conversations. Most people let this happen passively and never review what it actually retained.

Go to Settings → Memory and audit everything there. Delete anything outdated or wrong. Add what you actually want carried forward permanently.

One useful move: if you're migrating from ChatGPT, tell it "I'm moving this project to Claude, give me a document that will help me transfer over." Then upload that to Claude.

Connectors

Connectors give Claude direct access to your existing tools inside any conversation. Go to Settings → Connectors and link what you actually use.

The ones I use daily: Google Drive, Gmail, Slack, Notion, Google Calendar, and Excalidraw. There are 50+ available.

Research and thinking tools

Once Projects, Skills, Memory, and Connectors are set up, these are where output quality jumps.

Deep Research

When you activate Deep Research, Claude breaks down your query, searches across dozens of sources, cross-references findings, and returns a comprehensive cited report. Depending on complexity, it takes anywhere from five minutes to 45 minutes.

Enable it in the main chatbox under "+".

Extended Thinking

For strategy, analysis, and multi-step reasoning, you want Claude working through the problem before it responds, not just reacting quickly.

Two ways to trigger it: add language like "think deeply before responding" in your prompt, or toggle Extended Thinking directly in the model selection menu. For heavy reasoning tasks, pair it with Opus 4.6.

Artifacts

Artifacts are separate files, not buried text in a chat. HTML pages, React components, documents, diagrams, spreadsheets. You can view, edit, download, and iterate on them directly.

Ask Claude to "create this as an Artifact" or "build this as a downloadable file." They're enabled by default.

Agentic tools

This is where Claude starts doing real work without you driving every step.

Cowork

Cowork is only available in the downloaded Claude desktop app. Three features I use daily:

Scheduled Tasks: Claude runs tasks automatically on a fixed cadence. I use it for daily research and scanning Gmail and Calendar for briefs.

File Access: Claude gets access to your desktop files and folders. It can edit and work inside dedicated local workspaces directly.

Plug-Ins: Skills handle single repetitive tasks. Plug-Ins package multiple Skills into a single role, essentially a more capable version of the same idea.

Dispatch

Dispatch runs Cowork from your phone. Leave your laptop running, walk away, and prompt Claude to complete tasks remotely. Set it up inside Cowork under the Dispatch tab.

Claude in Chrome

Lets desktop app users start a task in Claude and have it handle browser work without switching windows.

Install from the Chrome Web Store under Anthropic's publisher page.

Building and coding

Claude Code

The most capable coding tool available right now. It handles complex debugging, ships websites and applications, writes and runs tests, runs security audits, plans coding sessions, and supports natural language coding.

Quickstart: code.claude.com/docs/en/quickstart

Slash commands

The built-in command system that speeds up how you operate Claude Code. You can also build your own custom slash commands on top of the defaults.

CLAUDE.md

A markdown file at the root of any project that Claude Code reads automatically before every session. It holds your project-specific instructions: coding standards, architecture decisions, file structure, things Claude should never touch.

Most Claude Code users never set this up. It's the single biggest quality improvement available.

Example:

This is a Next.js project using TypeScript and Tailwind.
Always use functional components. Never use class components.
Run npm run lint before committing any changes.
All API calls go through the /lib/api folder. Never call APIs directly from components.
Do not modify the /config folder without asking first.

Multi-agent mode

Claude Code can spin up multiple subagents working in parallel on different parts of the same project. One agent writes tests while another builds the feature. One refactors code while another handles documentation.

Claude Code decides when to use subagents based on task complexity. To trigger them effectively, give high-level outcome-oriented prompts rather than step-by-step instructions.

Memory (/memory)

Claude Code has its own memory system, separate from the web memory. Tell it to remember specific things about your project, preferences, or coding standards. It stores this in a memory file and carries it forward across sessions.

Combined with CLAUDE.md, this builds compounding context over time. Claude Code stops starting from scratch every session and starts building on what it already knows about your project.




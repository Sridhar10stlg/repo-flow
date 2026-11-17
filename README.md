# Repo Flow

Task management + an engineering workflow, inside VS Code.

> **Status: Phase 0.2.0, workflow kit (in progress).** The workflow kit is drafted in [kit/repo-flow/](kit/repo-flow/) and is installed once per machine as the personal Claude Code skill `/repo-flow`. See the [user guide](docs/USER_GUIDE.md). The VS Code extension isn't built yet. See [Plan.md](Plan.md) for the phased roadmap.

## What it is

Repo Flow helps one developer start, keep moving, and resume work on development tasks that an AI coding tool does most of the coding for. Repo Flow targets Claude Code. The workflow kit is plain Markdown, so support for other tools could be added later.

**Core principle: Repo Flow builds only what AI coding tools can't do.** Claude Code and similar tools already read and edit files, run `git`, follow written workflows, ask questions, and commit with approval. Repo Flow doesn't rebuild any of that.

Repo Flow has two parts:

- **Repo Flow extension (VS Code):** the attention layer. It keeps your tasks and the next action visible, helps you start or resume a session in one step, offers break mode, and sends respectful reminders and nudges. It is **read-only** on your repository files.
- **Repo Flow workflow kit:** Markdown instructions that your AI tool follows to work through a task. It isn't application code.

## Why an extension at all

An AI coding tool only acts after you open a session and type something. The hard moments are **before** that: getting started, and sitting back down after a break. Often a Claude Code session sits idle simply because the next step isn't clear.

The extension is for those moments. It can:

- show you the task and its next action before you've asked anything,
- notice when you're back from a break and show a resume card,
- remind you at a planned start time, or nudge you when a session has gone quiet,
- stay out of the way during breaks, outside working hours, and once you've had enough nudges for the day.

Repo Flow doesn't treat productivity as a matter of willpower. An idle session might mean you're thinking, reading, or away, and that's fine. Reminders are configurable, easy to snooze or dismiss, and never shame or nag.

## How it fits with Microsoft To Do

**Early releases:** Microsoft To Do stays your daily task list. Repo Flow takes over once you pick a development task to work on with your AI tool.

**Long-term goal:** Repo Flow may grow into a full daily task manager that replaces Microsoft To Do (Phase 0.9.0 in [Plan.md](Plan.md)). How to get there is still open, including where non-repository tasks would live and how you'd reach them from a phone.

## Task flow (yours to define, run by your AI tool)

The steps a task goes through are **not hardcoded**. You define the flow as a numbered list in an editable Markdown file, and your AI tool follows whatever steps you've written. You can rename, reorder, add, or remove steps, and add a note such as "approval required" to a step. Your personal default flow applies to every repository. A repository can have its own flow, and a single task can have its own `## Flow` section. See the [user guide](docs/USER_GUIDE.md#5-editing-your-flow). The extension reads the flow to show progress, such as "Step 3 of 5, next: Recording decisions".

Repo Flow ships with this **default flow**:

1. Impact analysis
2. Documentation
3. Questions and answers (in your AI tool's chat; there's no separate to-do chat)
4. Recording decisions
5. Building
6. Committing

Steps can also be skipped for a single task, with a recorded reason. However you change the flow, your approval is always required before project files are changed, task files are moved, or anything is committed. The extension's "Start session" button prepares a short prompt that points your AI tool at the task file and the kit. For Claude Code, the plan is to open a Claude Code tab with the prompt already filled in, so you just press Enter. For other tools, the prompt is copied for you to paste.

## Where tasks live

- **Tasks are read only from Markdown files under `docs/to-do/`**, including its subfolders (`README.md` files there are ignored). Nothing else in the repository is treated as a task.
- **The workflow kit is installed once per machine** (`~/.claude/skills/repo-flow/`), not in each repository. A repository needs only `docs/to-do/`, plus an optional `docs/repo-flow/flow.md` if it wants its own steps.
- **Completed tasks move to `docs/completed/`**, done by your AI tool after you confirm each move. Subfolders are mirrored, and filenames start with the task's stable ID.
- Impact analysis may read project files. That's separate from finding tasks. It's deliberate and bounded, and it lists the files it inspected.

## Principles

- **Don't duplicate AI tools.** Before any feature is built, check whether the AI tool already does it.
- **Human review:** you approve consequential actions, such as changing project files, moving task files, or committing.
- **Respectful reminders:** configurable, snoozable, and aware of working hours and breaks.
- **Privacy:** everything stays local unless you explicitly opt in to something else.

## Project documents

- [docs/USER_GUIDE.md](docs/USER_GUIDE.md): how to install the kit, use `/repo-flow`, edit your flow, and (later) use the extension
- [Plan.md](Plan.md): phased roadmap, overlap audit, task file format, decisions
- [CLAUDE.md](CLAUDE.md): instructions for Claude Code working in this repository
- [AI_Guidelines.md](AI_Guidelines.md): guidelines for AI-assisted work and future AI features

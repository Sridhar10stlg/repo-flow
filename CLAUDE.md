# CLAUDE.md

Instructions for Claude Code working in the Repo Flow repository.

## Project purpose and status

Repo Flow is a planned VS Code extension plus a Markdown workflow kit. It helps one developer start, keep moving, and resume AI-driven development tasks. See [README.md](README.md) for the overview.

**Current status: planning (Phase 0.1.0 in [Plan.md](Plan.md)).** No extension code or kit files exist yet. Don't describe any feature as implemented. Don't scaffold, install dependencies, or write application code until the plan says the relevant phase has started and the developer asks for it.

## Core principle: build only what AI coding tools can't do

Claude Code, Codex, and similar tools already edit files, run `git`, follow written workflows, ask questions, and commit. Repo Flow doesn't rebuild any of that.

- **Extension (attention layer):**
  - task visibility and the next action,
  - Start session (copies a prompt),
  - resume cards and break mode,
  - reminders and idle nudges.
- **Workflow kit (Markdown instructions):** the **user-defined flow** (the default is six steps, but the developer can edit it), creating and editing task files, recording decisions, `git mv`, and committing. All of this is done by the AI tool.

Before proposing or building any feature, check the overlap audit in [Plan.md](Plan.md). If the AI tool already provides the feature, don't build it. If the audit has no verdict for it yet, flag that instead of assuming.

## The problem we are solving

The developer's main difficulty is **starting work** and **resuming after a break**, plus AI sessions that sit idle or underused. An AI tool can only act after it's prompted, so the extension's job is the moment before a session starts and the moment after a break.

An idle session is **not** proof of procrastination. The developer may be thinking, reading, or away.

## Hard rules

1. **The extension is read-only on repository files.** It never creates, edits, moves, or deletes them. Where a change is needed, the extension offers a "copy prompt" action and the AI tool does the work. The one exception is the approval-gated "Repo Flow: Set up this repository" command, which copies the kit into `docs/repo-flow/`.
2. **Task-file boundary.** Task input comes only from Markdown files under `docs/to-do/` (subfolders included, `README.md` files ignored). This applies to the extension and to the kit. One named exception: the extension may read the repository flow file `docs/repo-flow/flow.md`, as configuration and never as task input. Completed tasks move to `docs/completed/` only after confirmation, following the completion-move rules in [Plan.md](Plan.md). `docs/completed/` may only be checked for whether a destination path exists. It's never listed or parsed.
3. **Discovery is not impact analysis.** Impact analysis may read project files, but only deliberately and within bounds, and it must list every file it inspected.
4. **The flow is user-defined.** Never hardcode the default six steps in the kit, the extension, or examples presented as rules. Follow the steps in the effective flow: the task's own "## Flow" section if it has one, otherwise the repository flow file. The format is a numbered list with optional notes after a colon. No flow edit removes rule 7 (approval).
5. **Q&A happens in the AI tool's chat.** Don't design a separate to-do agent chat.
6. **No assumed integration APIs.** Every mechanism must have evidence in the overlap audit in [Plan.md](Plan.md), and rows marked "Experiment needed" must pass their experiment before anything depends on them. Current plan: Start session pre-fills a Claude Code tab through the documented URI handler, with the clipboard as the fallback. Repo Flow never repeats notifications the AI tool already sends.
7. **Human approval** before changing project files, moving or renaming task files, committing, or sending data off the machine.
8. **Respectful reminders:** configurable, easy to snooze or dismiss, respectful of working hours and breaks, and worded without judgment.
9. **Microsoft To Do** stays the daily list in early phases. Replacing it is a long-term goal (Phase 0.9.0). Don't build daily-task features or widen the `docs/to-do/` boundary before an approved decision.
10. **No silent data sharing** with external services or other models.

## Required reading before work

- **[Plan.md](Plan.md):** follow the phase order (kit in 0.2.0, then the extension MVP in 0.3.0). Don't claim a phase is complete unless every success criterion is met and the evidence is recorded.
- **[AI_Guidelines.md](AI_Guidelines.md):** consult it before any AI-related design or implementation.

## Documentation and planning conventions

- Keep docs concise Markdown with clear headings.
- Label content as **Requirement**, **Proposal**, or **Open decision**. Don't promote a proposal to a requirement without the developer's approval.
- Record decisions in the Decision log in [Plan.md](Plan.md), with the date and rationale. Mark replaced decisions as superseded instead of deleting them.
- Keep README.md, CLAUDE.md, Plan.md, and AI_Guidelines.md consistent. If you change one, check the others.
- Use version numbers from Plan.md when referring to phases.
- Don't put secrets, credentials, tokens, or private personal data in any repository file.
- Don't add AI attribution lines to repository documents.

## Working style

- Separate observed facts, assumptions, and recommendations.
- Cite files you inspected. Don't make claims about files you haven't read.
- When a requirement is ambiguous, ask one focused question instead of guessing.
- Keep proposed changes small and reviewable.
- Don't commit unless the developer asks.

## Repository notes

- The extension language is **TypeScript** (decided 2026-11-17). The existing `.gitignore` is still a Python template. Replace it with TypeScript/Node entries when extension scaffolding starts in Phase 0.3.0, not before.
- The extension stores its local state (break mode, settings, activity signals, feedback) in VS Code's own storage, not in the repository. It is kept for 90 days.
- The workflow kit lives in `docs/repo-flow/`.

---
name: repo-flow
description: Run the Repo Flow workflow on Markdown tasks in the current repository's docs/to-do/. Commands are new, start, pause, skip, and done. Use when the user types /repo-flow, or asks to create, start, continue, pause, skip a step of, or finish a Repo Flow task.
---

# Repo Flow

This is a personal skill, installed once at `~/.claude/skills/repo-flow/`. It works in any repository. The only thing a repository needs is its `docs/to-do/` folder, and the `new` command creates it.

1. Read `~/.claude/skills/repo-flow/workflow.md` in full and follow it exactly.
2. Find the task's effective flow as that file describes:
   - the task's own `## Flow` section,
   - otherwise the repository's `docs/repo-flow/flow.md`, if it exists,
   - otherwise `~/.claude/skills/repo-flow/flow.md`.
3. Run the command the user asked for: `new`, `start`, `pause`, `skip`, or `done`. If they gave no command, list the tasks in `docs/to-do/` (status, stage, next action) and ask which to work on.

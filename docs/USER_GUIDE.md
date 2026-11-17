# Repo Flow user guide

Repo Flow helps you start, keep moving, and resume development tasks that Claude Code works on with you. It has two parts:

- **The workflow kit** (available now): a personal Claude Code skill, `/repo-flow`, that runs your tasks through the steps you choose.
- **The VS Code extension** (planned, not built yet): keeps your tasks and next action visible, helps you start and resume sessions, and sends respectful reminders.

Contents:
1. [Install the kit](#1-install-the-kit)
2. [Use Repo Flow in a repository](#2-use-repo-flow-in-a-repository)
3. [Commands](#3-commands)
4. [Task files](#4-task-files)
5. [Editing your flow](#5-editing-your-flow)
6. [Taking a break and coming back](#6-taking-a-break-and-coming-back)
7. [Finishing and archiving](#7-finishing-and-archiving)
8. [The VS Code extension (planned)](#8-the-vs-code-extension-planned)
9. [Privacy](#9-privacy)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Install the kit

The kit is installed **once per machine** and works in every repository. Nothing is copied into your repositories.

From a clone of the Repo Flow repository:

```bash
mkdir -p ~/.claude/skills/repo-flow
cp kit/repo-flow/SKILL.md kit/repo-flow/workflow.md kit/repo-flow/task-template.md ~/.claude/skills/repo-flow/
cp --update=none kit/repo-flow/flow.md ~/.claude/skills/repo-flow/   # only if you don't have one yet
```

Then start a **new** Claude Code session. Skills are picked up when a session starts. Type `/repo-flow` to check it's there.

**To update the kit later,** run the same commands. The second line never overwrites your own `flow.md`.

Once the extension exists, its **Repo Flow: Install or update** command will do this for you, with the same rule about `flow.md`.

**To uninstall,** delete `~/.claude/skills/repo-flow/`. Your task files stay in your repositories.

---

## 2. Use Repo Flow in a repository

A repository needs only one thing: a `docs/to-do/` folder for its tasks. You don't have to create it. `/repo-flow new` offers to create it with your first task.

```
your-repo/
  docs/
    to-do/          ← active tasks (subfolders allowed, e.g. docs/to-do/backend/)
    completed/      ← finished tasks, moved here when you archive them
    repo-flow/
      flow.md       ← optional: steps for this repository only
```

Rules Repo Flow always follows:
- **Only files in `docs/to-do/` are tasks.** Nothing else in the repository is treated as one. `README.md` files there are ignored.
- **Nothing changes without your yes.** Claude shows every file change, move, or commit first.
- **No commits or pushes** unless you approve that specific one.

---

## 3. Commands

Type these in Claude Code. You can also just say what you want, for example "pause the retry task".

| Command | What it does |
|---|---|
| `/repo-flow` | Lists your tasks with status, current step, and next action, then asks which one to work on |
| `/repo-flow new <title>` | Creates a task. Claude drafts the file and shows it before writing it |
| `/repo-flow start <task>` | Starts or continues a task: a short summary of where it stands, then the next step. `<task>` can be the ID, or a few words of the title |
| `/repo-flow pause` | Before a break: writes a Resume note and the next action |
| `/repo-flow skip <step> <reason>` | Skips one step for this task and records why |
| `/repo-flow done` | Marks the task done, then asks whether to move it to `docs/completed/` |

Example:
```
/repo-flow new Add retry to the sync client
/repo-flow start retry
```

---

## 4. Task files

Each task is one Markdown file named `<id>-<slug>.md`, for example `docs/to-do/20261117-1432-a7k2-add-retry-to-sync-client.md`.

```markdown
---
id: 20261117-1432-a7k2
title: Add retry to sync client
status: in-progress          # todo | in-progress | blocked | done
stage: Questions and answers # the current step of your flow
due: 2026-11-20              # optional: date, or date and time (local)
start_at: 2026-11-18 09:30   # optional: planned start (for reminders)
created: 2026-11-17
updated: 2026-11-17
next_action: Decide the maximum number of retries
---

# Add retry to sync client

## Context
## Resume note
## Impact analysis
## Documentation
## Questions and answers
## Decisions
## Build notes
## Commit
## Skipped stages
```

You can edit task files by hand at any time. Keep `next_action` to **one concrete step**. That line is what helps you get started again.

---

## 5. Editing your flow

Your **flow** is the list of steps a task goes through. Nothing is hardcoded: Claude follows whatever steps you write.

### Where flows live

Claude uses the first of these that exists:

| Priority | Location | Applies to |
|---|---|---|
| 1 | A `## Flow` section inside a task file | That task only |
| 2 | `docs/repo-flow/flow.md` in a repository | That repository only (optional) |
| 3 | `~/.claude/skills/repo-flow/flow.md` | **Your default for every repository** |

Most people only ever edit number 3.

### Format

A numbered list under a `## Flow` heading. One step per line. You can add a note after a colon.

```markdown
## Flow
1. Impact analysis
2. Documentation
3. Questions and answers
4. Recording decisions
5. Building: approval before changing project files
6. Committing: approval required
```

That's the default flow Repo Flow starts with.

### Common edits

**Rename a step:** just change the text.
```markdown
3. Clarify with Claude
```

**Add a step:**
```markdown
6. Run tests: all tests must pass before committing
7. Committing: approval required
```

**Remove a step:** delete the line and renumber.

**Reorder:** change the order of the lines and renumber.

**Add a checkpoint or instruction:** put it after a colon.
```markdown
2. Documentation: update README only if user-facing behaviour changes
```

**A lighter flow for one repository:** create `docs/repo-flow/flow.md` in that repository:
```markdown
## Flow
1. Building: approval before changing project files
2. Committing: approval required
```

**A different flow for one task:** add a `## Flow` section to that task file. It replaces the other flows for that task only.

**Skip a step just this once:** use `/repo-flow skip <step> <reason>` instead of editing the flow. The skip and its reason are recorded in the task.

### Good to know

- **Approvals you can't remove:** whatever your flow says, Claude still asks before changing project files, moving task files, committing, or sending data anywhere. A flow can add checkpoints but can't remove these.
- **Changes apply straight away.** Claude reads the flow at the start of each command.
- **Renaming a step that a task is currently on:** Claude matches the task's `stage` ignoring capital letters and extra spaces. If it still can't find the step, it asks. You can also update `stage` in the task file by hand.
- **Claude won't change your flow by itself.** It may suggest a change, but you decide.

---

## 6. Taking a break and coming back

**Before a break**, run:
```
/repo-flow pause
```
Claude writes a 2–4 line **Resume note** (what's done, what's in progress, what to do first) and updates `next_action`. It's quick on purpose, so you can leave.

**When you come back**, run:
```
/repo-flow start <task>
```
Claude reads the Resume note, checks what changed, summarizes where you are in a few lines, and proposes the next step.

If you forgot to pause, `start` still works. It rebuilds the picture from the task file and recent changes.

---

## 7. Finishing and archiving

```
/repo-flow done
```

1. Claude proposes setting `status: done`.
2. It asks whether to move the task to `docs/completed/` now.
   - **No:** the task stays in `docs/to-do/` as done. Run `done` again later to archive it.
   - **Yes:** Claude shows the plan first:
     - the destination, keeping any subfolder (`docs/to-do/backend/x.md` → `docs/completed/backend/x.md`),
     - fixes to relative links inside the file,
     - any other files that link to it. These are listed but not edited.

     It never overwrites an existing file. It uses `git mv` if the file is tracked, and it doesn't commit unless you approve.

---

## 8. The VS Code extension (planned)

> **Not built yet.** This section describes planned behaviour (Phase 0.3.0 onward in [Plan.md](../Plan.md)). It will change as it's built and tested.

The extension handles what Claude Code can't do on its own: helping **before** a session starts and **after** a break. It never edits your repository files.

**Task view:** your tasks from `docs/to-do/`, grouped by subfolder, with status, due date, and a "Done, awaiting archive" group. Files with problems are flagged with a "Copy fix prompt" button. Nothing is hidden.

**Today / Focus:** pick one task and see its next action and Resume note, plus progress such as "Step 3 of 6: Questions and answers. Next: Recording decisions".

**Start session:** opens a new Claude Code tab with a ready-made prompt filled in, for example "Continue task … using Repo Flow". You press Enter. VS Code may first ask you to allow the link. If that doesn't work, the prompt is copied for you to paste.

**Resume card:** when you come back after a while away, it shows your current task, its Resume note and how old that note is, with a Start session button.

**Break mode:** "I'm on a break" pauses all reminders until you're back or until a time you choose.

**Reminders and nudges** (all optional, off unless you turn them on):
- a reminder at a task's planned start time (`start_at`),
- a gentle nudge when a task is in progress and its session has been waiting for you a while.

Every reminder shows the concrete next action, can be snoozed or dismissed in one click, and respects your working hours, quiet hours, and a daily limit. A small "helpful / not now / not helpful" button helps tune them.

**Settings:** reminders on/off, working hours, quiet hours, daily nudge limit, snooze lengths, and which activity signals are allowed.

**Install or update:** installs the kit into `~/.claude/skills/repo-flow/` (see section 1), with your approval.

---

## 9. Privacy

- Everything stays on your machine by default.
- The extension keeps its own data (break mode, settings, activity timestamps, reminder feedback) in VS Code's storage, not in your repository. Data is deleted after **90 days**, and a "Delete all Repo Flow data" command clears it at any time.
- Activity signals are opt-in, one at a time. Planned signals:
  - window focus,
  - editor and terminal activity (timestamps only),
  - task file changes,
  - Claude Code session events. These come from an optional hook that records only the event name, time, and session ID, never your prompts or code.
- Nothing is sent to any service or model without your explicit opt-in.

---

## 10. Troubleshooting

| Problem | Fix |
|---|---|
| `/repo-flow` isn't recognised | Check that `~/.claude/skills/repo-flow/SKILL.md` exists, then start a **new** Claude Code session |
| A task doesn't appear | It must be a `.md` file under `docs/to-do/` and not named `README.md`. If its front matter is broken, Claude (and later the extension) will flag it |
| Claude uses the wrong steps | Check the three flow locations in priority order: the task's `## Flow` section, then the repository's `docs/repo-flow/flow.md`, then your personal `flow.md` |
| The current step isn't found after renaming | Update `stage` in the task file, or run `/repo-flow start` and Claude will ask |
| You updated the kit but your flow reset | Make sure the update didn't overwrite `~/.claude/skills/repo-flow/flow.md`. The install commands in section 1 never do |

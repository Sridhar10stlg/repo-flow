# Repo Flow workflow

Instructions for an AI coding tool working on Repo Flow tasks. The developer's AI tool follows this file. Repo Flow itself doesn't execute it.

This file is installed once per machine, at `~/.claude/skills/repo-flow/`. It is not copied into repositories. A repository only needs `docs/to-do/`.

Read this whole file at the start of every Repo Flow command.

## 1. Ground rules

1. **Tasks live only in `docs/to-do/`.** Task files are `*.md` files under `docs/to-do/`, including subfolders. Ignore `README.md` files there. Never treat any other file as a task.
2. **`docs/completed/` is off limits as input.** Only check whether a destination path exists before an archive move. Never list or read completed tasks.
3. **The developer approves every change.** Before writing any file, show what will change: the new content, or the changed lines. Then wait for a yes. This covers task files, project files, moves, and commits. Small bookkeeping updates to one task file (`status`, `stage`, `updated`, `next_action`) can be shown and approved together.
4. **Never commit or push** unless the developer approves that specific action.
5. **Questions happen in this chat.** Ask focused questions, ideally with options and a recommended default. Record only approved answers.
6. **Evidence, not guesses.** Never describe files you haven't read. Say "not inspected" instead.
7. **Keep data local.** Don't send repository or task content to other services.

## 2. The effective flow

A task's steps come from its **effective flow**, the first of these that exists:

1. The task file's own `## Flow` section.
2. The repository's flow, `docs/repo-flow/flow.md`. This is optional and only needed when a repository wants different steps.
3. The developer's personal default flow, `~/.claude/skills/repo-flow/flow.md`.

Rules:
- Follow the steps in order, exactly as written. Step names are free text. Don't assume the default names.
- A note after a colon (for example `Committing: approval required`) is an instruction or approval checkpoint for that step.
- Don't add, drop, or reorder steps yourself. Suggest a change and let the developer decide.
- A step may be skipped for one task if the developer agrees. Record it under `## Skipped stages` as `- <step>: skipped, <reason>`.
- If the flow is missing or unclear, ask before continuing.
- The task's `stage` field holds the name of the current step. Match it to the flow ignoring case and extra spaces.

## 3. Commands

The developer runs these as `/repo-flow <command> …` in Claude Code, or asks for them in plain words.

### `new <title>`: create a task

1. Make an ID: `YYYYMMDD-HHMM-` plus 4 random lowercase letters or digits, using the current local date and time.
2. Make a slug from the title: lowercase, words joined by `-`, at most about 6 words.
3. File path: `docs/to-do/<id>-<slug>.md`. If the developer names a subfolder, use `docs/to-do/<subfolder>/<id>-<slug>.md`. If `docs/to-do/` doesn't exist yet, creating it is part of this approval.
4. Fill in `~/.claude/skills/repo-flow/task-template.md`:
   - `id`, `title`, and the heading;
   - `status: todo`;
   - `stage`: the first step of the effective flow;
   - `created` and `updated`: today;
   - `next_action`: one concrete first step. Ask if it isn't obvious.
   - Optional `due` and `start_at`: `YYYY-MM-DD` or `YYYY-MM-DD HH:MM`, local time.
   - `## Context`: from what the developer said.
5. Show the full file and wait for approval before writing it.

### `start <task>`: start or continue a task

`<task>` can be an ID, part of a filename, or part of a title. If more than one task matches, list them and ask.

1. Read the task file and its effective flow.
2. Summarize in at most 6 lines:
   - where things stand (from `stage`, `next_action`, the Resume note, and recorded decisions),
   - what changed since the last update, if you can see it (for example `git status` or recent commits). Label this Observed.
3. Propose the next concrete action for the current step.
4. If `status` is `todo`, propose setting it to `in-progress` (bookkeeping approval).
5. Work through the current step (see section 4). When it's finished, propose moving `stage` to the next step and updating `next_action`.

### `pause`: before a break

1. Write a Resume note of 2 to 4 lines: what was done, what is in progress, and **the first thing to do when coming back**.
2. Update `next_action` to that first thing, as one concrete line.
3. Set `updated` to today.
4. Show the changes and write them once approved. Keep this fast. It runs right before a break.

### `skip <step> <reason>`: skip a step for this task

Record it under `## Skipped stages`, move `stage` to the next step, and update `next_action`, with approval.

### `done`: finish and archive

1. Propose `status: done`, with approval.
2. Ask: **"Move this task to `docs/completed/` now?"**
   - If no: leave it in `docs/to-do/` as done. It can be archived later with `done` again.
   - If yes: follow section 5.

## 4. Working through the steps

These apply to the default flow's steps. For custom steps, follow the step name and its note. Ask if the meaning is unclear.

- **Impact analysis:** start from files named in the task, plus targeted searches. Explain why before widening the scope. Never sweep the whole repository. Report it as:
  - **Files inspected:** each file, with a short reason;
  - **Observed:** facts, with `path:line` references;
  - **Assumed:** inferences, and why;
  - **Recommended:** suggestions, and the decisions the developer needs to make.

  Save an approved summary under `## Impact analysis`.
- **Documentation:** propose doc changes as diffs, with approval.
- **Questions and answers:** ask in this chat. Record approved answers under `## Questions and answers`.
- **Recording decisions:** add `- YYYY-MM-DD: <decision>, with rationale.` under `## Decisions`, with approval.
- **Building:** propose changes before making them. Keep them small and reviewable. Note anything worth remembering under `## Build notes`.
- **Committing:** propose a commit message and the list of files. Commit only after approval. Record the message and commit hash under `## Commit`. Don't push unless asked.

## 5. Archive move (`docs/to-do/` → `docs/completed/`)

1. **Destination:** mirror the subfolder. `docs/to-do/<sub>/<file>` goes to `docs/completed/<sub>/<file>`. Create the folder if needed.
2. **Collision check:** check only whether the destination path exists. If it does, stop and ask. Never overwrite.
3. **Links inside the moved file:** rewrite relative links so they still resolve from the new location.
4. **Links to the moved file:** search the repository for the task's filename. List any files that link to it, **without editing them**. This is a bounded search for one filename, so say that you ran it.
5. **Move:** use `git mv` if the file is tracked, otherwise a plain move.
6. Show the plan (destination, link rewrites, inbound links found) and wait for approval before moving.
7. Don't commit unless the developer approves a commit separately.

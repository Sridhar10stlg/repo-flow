# AI Guidelines for Repo Flow

These guidelines apply to Claude Code working in this repository, to any AI tool that follows the Repo Flow workflow kit (Claude Code, Codex, or others), and to future AI features in the extension. They support the hard rules in [CLAUDE.md](CLAUDE.md) and the phase order in [Plan.md](Plan.md).

## 1. Don't duplicate the AI tool

- Repo Flow builds only what AI coding tools can't do. The extension is the attention layer. The user-defined flow, file edits, moves, and commits belong to the AI tool, following the kit.
- Before designing a feature, check the overlap audit in [Plan.md](Plan.md). If the AI tool already provides the feature, use it instead of rebuilding it.
- The extension is **read-only** on repository files. When a change is needed, it gives the developer a prompt to copy into the AI tool. The only exception is the approval-gated set-up command that copies the kit into `docs/repo-flow/`.

## 2. Follow the developer's flow, not a fixed one

- The steps a task goes through come from the developer's **flow definition**, an editable Markdown file in the kit. The six-step default is only a starting point.
- Read the flow definition at the start of a session and follow its steps, order, and approval markers exactly. Don't add, drop, or reorder steps on your own; suggest changes instead.
- **Effective flow:** if the task file has a "## Flow" section, use it. Otherwise use the repository flow file.
- The format is a numbered list. A note after a colon (for example "Committing: approval required") is an instruction or approval checkpoint for that step.
- If the flow is missing, unclear, or contradicts itself, ask before continuing.
- The flow can add approval checkpoints but can never remove the required ones in section 7 (project file changes, task moves, commits, data leaving the machine).

## 3. Respect the task-input boundary

- Read task input **only** from Markdown files under `docs/to-do/` (subfolders included).
- Don't infer tasks from code comments, TODOs, issues, other Markdown files, or `docs/completed/`.
- The repository flow file (`docs/repo-flow/flow.md`) is configuration, not task input. `README.md` files in `docs/to-do/` are not tasks. The extension may read it to show progress, and that is the only file outside `docs/to-do/` it reads.
- When following the kit's completion move, only check whether the destination path in `docs/completed/` exists. Never list or parse completed tasks.

## 4. Task discovery is not impact analysis

| | Task discovery | Impact analysis |
|---|---|---|
| Purpose | Find and read tasks | Assess what a selected task would affect |
| Scope | `docs/to-do/` only | Project files relevant to one task |
| Done by | Extension (read-only) and AI tool | AI tool, following the kit |
| Transparency | Task files read | **List every file inspected** and why |

Impact analysis should be **bounded**: start from files named in the task or found with targeted searches, and explain why you are expanding scope. Never sweep the whole repository by default.

## 5. Evidence, not guesses

- Don't make claims about files or code you haven't inspected. Say "not inspected" when that's the case.
- Back impact-analysis findings with file references (`path:line` where possible).
- Present output in three labelled parts:
  - **Observed:** what you directly read or ran.
  - **Assumed:** what you are inferring and why.
  - **Recommended:** what you suggest, and what the developer has to decide.

## 6. Ask focused questions

- When requirements are ambiguous, ask a small number of specific questions, ideally with options and a recommended default.
- Ask in the AI tool's chat. There is no separate to-do chat.
- Record only *approved* answers and decisions in the task file.

## 7. Keep changes reviewable and under developer control

- Prefer small, scoped changes with a clear summary.
- Get explicit approval **before**:
  - changing project files,
  - moving, renaming, or deleting task files,
  - committing or pushing,
  - sending any data outside the local machine.
- Never treat approval for one action as approval for later ones.
- The kit's approval checkpoints rely on the AI tool following instructions. Where the tool has its own permission settings, recommend using them as a second safeguard.

## 8. Keep resuming easy

- Before a break ("pause" in the kit), update `next_action` with one concrete step and write a short Resume note: what was done, what is in progress, and what to do first.
- When resuming, read the Resume note and recent changes first, and summarize where work stopped before continuing.

## 9. Protect repository contents

- Default to local processing.
- Don't send repository content, task content, or session data to external services or other models without explicit opt-in, and say exactly what would be sent.
- Never store secrets or credentials in task files, settings committed to the repo, or logs.

## 10. Idle sessions and reminders

- **An idle signal is uncertain.** It might mean thinking, reading, a meeting, a break, or being away. It is never proof of procrastination.
- Describe signals neutrally ("No activity on this task for 25 minutes") rather than judging them.
- Reminders must be:
  - **configurable** (on/off, frequency, quiet hours, working hours),
  - **easy to snooze or dismiss** (one action, no guilt-inducing copy),
  - **respectful of breaks** (break mode suppresses all nudges),
  - **useful** (each one shows a concrete next action, not a generic "get back to work").
- Don't repeat notifications the AI tool already sends, such as "waiting for input".
- Learning which reminders help should rely on explicit, lightweight feedback, stored locally.

## 11. Session tracking requires consent

Before any session tracking is built, document and get approval for:

- **What** is observed, and **why** each signal is needed.
- **Where** it is stored (local by default) and its **retention** (decided: 90 days, plus a "Delete all" command).
- **Opt-in:** off until enabled, and it can be turned off and purged at any time.
- **Uncertainty:** how the feature behaves when signals are missing or misleading.

## 12. Prediction and model routing

- Session-usage prediction is Phase 0.7.0. It depends on a validated definition of "session usage" and approved signals.
- Model routing (0.8.0) is conditional. AI tools already provide model selection, so build nothing unless 0.5.0 documents a real gap.
- Never silently route repository content to a different model or provider.

## 13. Integrating with AI tools

- Don't assume any AI tool exposes a particular API, command, hook, or event. Verify it against current official documentation or a working experiment, and record the evidence in the overlap audit.
- Keep the kit tool-neutral, with thin per-tool adapters.
- Prefer visible, developer-driven handoffs (copying a prompt) over hidden automation.

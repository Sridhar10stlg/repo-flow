# Repo Flow: Plan

Phased roadmap for Repo Flow. See [README.md](README.md) for the overview, [CLAUDE.md](CLAUDE.md) for working rules, and [AI_Guidelines.md](AI_Guidelines.md) for AI-specific guidance.

**Current phase: 0.2.0 (Workflow kit).** Phase 0.1.0 completed on 2026-11-17. No extension code exists.

Labels used in this document:
- **Requirement:** firm. Change it only with the developer's explicit approval.
- **Proposal:** a suggested approach that still needs approval.
- **Open decision:** unresolved. Don't settle it silently.

---

## Core principle: build only what AI coding tools can't do

Claude Code, Codex, and similar AI coding tools already read and edit files, run `git`, follow written workflows, ask questions, and commit with approval. **Repo Flow doesn't rebuild any of that.**

Repo Flow has two parts:

| Part | What it is | What it does |
|---|---|---|
| **Repo Flow extension** (VS Code, TypeScript) | The *attention layer*: code that runs in the background, stays visible, and can act first | Shows tasks and the next action, helps you start and resume a session, break mode, reminders, idle nudges |
| **Repo Flow workflow kit** | Markdown conventions and instructions, **not application code** | Tells your AI tool how to run **your flow** on a task file (an editable list of steps; the default is impact analysis, docs, Q&A, decisions, building, committing), plus moving finished tasks |

**Why the extension exists:** an AI coding tool only acts after you open a session and prompt it. The developer's problem is the moment *before* that: starting, and coming back after a break. An extension can show the next action, notice a return from a break, and nudge at a planned time without being asked.

**Who does what**

| Job | Owner |
|---|---|
| Task list, Today/Focus view, next action always visible | Extension |
| Resume cards, break mode, reminders, idle nudges | Extension |
| "Start session" (copy a ready-made prompt for the AI tool) | Extension |
| Running the steps of the user-defined flow (default: impact analysis, docs, Q&A, decisions, building) | AI tool, following the kit |
| Creating tasks, editing task files, recording decisions, `git mv`, committing | AI tool, following the kit, with developer approval |

---

## Firm requirements (all phases)

1. **No duplication.** Before building any feature, check whether the developer's AI tools already provide it. Record the evidence. Drop the feature if they do (see "Overlap audit").
2. **The extension is read-only on repository files.** It never creates, edits, moves, or deletes them, and all file changes go through the AI tool following the kit. The kit is installed **once per machine**, in `~/.claude/skills/repo-flow/`, and is never copied into repositories. The extension's "Install or update Repo Flow" command writes only there, after showing exactly what it will add and getting approval.
3. Task input is read **only** from Markdown files under `docs/to-do/`, subfolders included (`README.md` files are ignored). This applies to the extension and to the kit. **Named exception:** the extension may also read flow files, as configuration and never as task input. These are the optional repository flow `docs/repo-flow/flow.md` and the personal default `~/.claude/skills/repo-flow/flow.md`.
4. Completed tasks move from `docs/to-do/` to `docs/completed/`. The AI tool does the move, following the kit's completion-move rules.
5. Task discovery is separate from impact analysis. Impact analysis is deliberate, bounded, and lists the files it inspected.
6. Questions and answers happen in the developer's AI coding tool chat (Claude Code by default). There is no separate to-do agent chat.
7. Human approval is required before changing project files, moving task files, committing, or sending data off the machine. **A custom flow can add approval checkpoints but never remove these.**
8. **The flow is user-defined.** The steps a task goes through come from an editable flow definition, not hardcoded stages. The six-step default is only a starting point. The kit and the extension must work with any list of steps.
9. Reminders are configurable, easy to snooze or dismiss, and respect working hours and breaks.
10. Microsoft To Do stays the daily task list through the early phases. Replacing it is a long-term goal (Phase 0.9.0).
11. No session or repository data is sent to a service or model without explicit opt-in.

## Phase overview

| Version | Phase | Built from | Core focus |
|---|---|---|---|
| 0.1.0 | Product specification and overlap audit | Docs | Decide what to build. Verify what AI tools already do. Record a baseline. |
| 0.2.0 | Workflow kit | Markdown instructions only | Editable flow definition. The AI tool runs your steps and completion moves on task files |
| **0.3.0** | **MVP: attention extension** | Extension | Task visibility, next action, Start session, resume card, break mode, planned-start reminders |
| 0.4.0 | Idle-session nudges and feedback | Extension | Nudges from verified signals, local "was this helpful?" feedback |
| 0.5.0 | Daily-use validation and local insights | Extension | Measure what helps, tune it, re-check overlap |
| 0.6.0 | Optional hosted reminders (laptop off) | Extension plus a service | External notification channel |
| 0.7.0 | Session-usage prediction | Extension | Limited predictions from approved signals |
| 0.8.0 | Model and tool suggestions (*conditional*) | Probably nothing | Build only if 0.5.0 finds a gap AI tools don't cover |
| 0.9.0 | Full daily task manager (Microsoft To Do replacement) | Extension plus a companion app | Everyday and development tasks in one place |

**Why this order:**
- **The kit comes before the extension.** It is cheap (Markdown only), usable straight away in Claude Code, and it settles the task-file conventions the extension will read.
- **The MVP is the attention layer**, because that is the only reason an extension needs to exist. If resume cards and reminders don't help, 0.5.0 should be willing to conclude that the extension isn't worth continuing.
- Earlier versions of this plan had the extension edit and move task files and run the lifecycle. That was removed on 2026-11-17 because AI tools already do it (see the Decision log).
- **0.8.0 is conditional.** AI tools already choose and switch models, so routing would probably duplicate them.
- **0.9.0 comes last.** It needs hosted reminders (0.6.0), cross-device access, and a validated core.

---

## Phase 0.1.0: Product specification and overlap audit

**Purpose and scope**
- **Overlap audit:** for every planned extension feature, check whether Claude Code (and Codex, if used) already provides it. Record the evidence (see "Overlap audit").
- Finalize the task file format and discovery rules (see "Task file format").
- Define the **flow definition format** (how steps, their order, and approval markers are written) and write the **default flow** (the six steps) in that format.
- Choose where the workflow kit lives and how it is installed into a repository.
- Decide the minimum VS Code version and build/test tooling (decided as a rule; the concrete values are set at scaffolding in 0.3.0).
- Record a **one-week baseline** with the daily log (see "Measurement method").

**Exclusions:** no extension code, kit files, dependencies, or external services.

**Success criteria (complete when all are true)**
- [x] The overlap audit table has a verdict (Provided / Partly provided / Not provided / Unverified) and a source (doc link or experiment notes) for every row. Every feature marked Provided is removed from later phases. (2026-11-17: done for Claude Code; Codex is out of scope because the developer won't use it.)
- [x] The task format spec gives every field a status (all fields decided, 2026-11-17).
- [x] Discovery rules are written, including at least five malformed-input cases and how the extension displays each one (eight cases, decided 2026-11-17; see "Task file format").
- [x] Completion-move rules are decided and recorded (2026-11-17).
- [x] The flow definition format is decided (a numbered list with optional notes), with the six-step default and approval markers shown (2026-11-17).
- [x] Where flows live is decided: a repository flow file plus an optional per-task override (2026-11-17).
- [x] Extension language decided: TypeScript (2026-11-17). Minimum VS Code version and tooling decided as rules (2026-11-17); the concrete values are pinned at scaffolding.
- [x] Kit location and installation method are recorded in the Decision log (2026-11-17; revised the same day to a one-time install in `~/.claude/skills/repo-flow/`).
- [x] Baseline log entries exist for at least 5 working days (2026-11-10 to 2026-11-14, kept privately outside the repository). *Caveat:* filled in retrospectively from memory, not at the end of each day, so it's less reliable than a same-day log.

**Dependencies:** none.

**Open questions and risks**
- AI tools add features quickly. A feature that is "Not provided" today may be provided next quarter, so the audit is repeated in 0.5.0.
- Keeping a baseline log is itself a small habit and may be skipped. Keep it to under 1 minute a day.

---

## Phase 0.2.0: Workflow kit (no extension code)

**Purpose and scope**
- A **tool-neutral workflow file** that any AI coding tool can follow. It covers:
  - creating a task from the template,
  - **reading the developer's flow definition** and following its steps, order, and approval markers (the default flow ships with the kit),
  - impact analysis rules (bounded, list the inspected files; see [AI_Guidelines.md](AI_Guidelines.md)),
  - recording approved answers and decisions in the task file,
  - **pause:** update `next_action` and the Resume note before a break,
  - **complete and archive:** the completion-move rules below.
- **Thin adapters** so each tool can find the kit. For Claude Code, a custom command or skill; for Codex, its repository instructions file. Each adapter's mechanism must be verified first.
- The task file template.

**Exclusions:** extension code, reminders, and anything that requires a specific tool's private API.

**Success criteria**
- [ ] With Claude Code, at least 3 real tasks go from creation to completion through the flow, with decisions recorded in their task files.
- [ ] **Custom flow test:** after the flow is edited (at least one step renamed, one added, one removed, and the order changed), the AI tool follows the edited flow on a real task without being reminded. No default step reappears.
- [ ] In those runs, the AI tool asks for approval before every task-file move and every commit (checked against the session transcripts).
- [ ] At least one completion move follows every completion-move rule: confirmation, mirrored path, ID filename, rewritten internal links, inbound links reported, `git mv`.
- [ ] Using "pause" before a break leaves a `next_action` and Resume note that the developer rates as enough to restart from (yes/no, recorded for at least 3 breaks).
- [x] ~~If Codex is in use: at least one task runs through the kit in Codex.~~ Not applicable: the developer won't use Codex (2026-11-17).

**Dependencies:** 0.1.0 complete.

**Open questions and risks**
- **Approval is instruction-following, not code.** The kit can't *force* an AI tool to ask before moving or committing. Mitigation: also use each tool's own permission settings (for example requiring approval for `git` commands), if that is verified to be available.
- Different tools may follow the same instructions differently.

---

## Phase 0.3.0: MVP, the attention extension ⭐

**Purpose and scope**
- **Task view (read-only):** lists tasks from `docs/to-do/**/*.md`, grouped by subfolder, with status and due date. Includes a collapsed "Done, awaiting archive" group.
- **Today / Focus view:** pick one task and see its `next_action` and Resume note prominently.
- **Flow progress:** reads the task's effective flow (its own "## Flow" section if it has one, otherwise the repository flow file) and shows, for example, "Step 3 of 5: Questions and answers. Next: Recording decisions". If the `stage` doesn't match any step, or there's no flow file, it shows the `stage` text with a gentle warning.
- **Start session:** copies a short prompt to the clipboard. The prompt names the task file and tells the AI tool to continue it using the kit. On a resume, it also asks the tool to summarize where work stopped. **For Claude Code (decided 2026-11-17):** open a new Claude Code tab with the prompt pre-filled through the documented URI handler. The developer presses Enter. The clipboard is the fallback for Codex and other tools, or if the link fails. This depends on the 0.1.0 experiment (audit row 8).
- **Copy-prompt actions** instead of file edits: "Fix front matter", "Archive this task", "Create a task". The extension never writes task files itself.
- **"Repo Flow: Install or update" command:** installs the kit from the extension into `~/.claude/skills/repo-flow/` (`SKILL.md`, `workflow.md`, `task-template.md`, and `flow.md` with the default flow) after showing the file list and getting approval. Kit files are updated, but the developer's `flow.md` is never overwritten. Nothing is written into repositories. A repository only needs `docs/to-do/`, which `/repo-flow new` creates (Decided).
- **Resume card:** after a configurable inactivity gap (or on VS Code focus), shows the current task, its Resume note, how old that note is, and a Start session button.
- **Break mode:** "I'm on a break" pauses all nudges until you return or until a chosen time. The state is stored locally, outside the repository.
- **Planned-start reminder:** a local VS Code notification at a task's planned start time.
- **Settings:** reminders on/off, working hours, quiet hours, daily nudge cap, snooze durations.

**Exclusions:**
- writing, moving, or deleting any repository file,
- running flow steps (the kit handles them). The extension must not assume the default step names,
- idle-session nudges (0.4.0),
- network access, analytics, and Microsoft To Do integration.

**Success criteria**
- [ ] Fixture-repository tests: only `docs/to-do/**/*.md` files are listed. Markdown files elsewhere (repo root, `docs/`, `docs/completed/`, `src/`) never are.
- [ ] Each malformed-input case from 0.1.0 has a test showing the specified display.
- [ ] Tests and code review show that the extension makes no repository writes at all, and no network calls.
- [ ] Install command tests: cancelling writes nothing, approving writes exactly the listed files in `~/.claude/skills/repo-flow/`, and an existing personal `flow.md` is left untouched.
- [ ] From a freshly opened VS Code window, the selected task's next action is visible in **2 interactions or fewer** (manual test script).
- [ ] Flow progress tests: the repository flow, a task override, a non-default custom flow, a missing flow file, and a `stage` that matches no step each give the specified display. The only file read outside `docs/to-do/` is the flow file (test plus code review).
- [ ] No reminder appears when reminders are off, outside working hours, during break mode, or above the daily cap (tests with a simulated clock).
- [ ] Every reminder can be snoozed or dismissed in one action. Its wording passes a review against [AI_Guidelines.md](AI_Guidelines.md).
- [ ] The developer uses the extension on at least 5 working days that include breaks. The daily log records whether they started a task and whether they resumed after their longest break.

**Dependencies:** 0.2.0 complete, since the MVP reads the kit's task format and its prompts invoke the kit.

**Open questions and risks**
- If the URI handler experiment fails, Claude Code also uses the clipboard. Codex support for a pre-filled launch is unverified.

---

## Phase 0.4.0: Idle-session nudges and feedback

**Purpose and scope**
- An optional nudge when a task is in progress and no activity has been seen for a configurable period. Each nudge shows the task's concrete next action.
- **No duplication:** if the AI tool already notifies you when a session is waiting for input (check this in the overlap audit), Repo Flow doesn't repeat it. Repo Flow's nudge covers the human side instead: you've been away, you have planned work, here is the next step.
- **Approved signals (decided 2026-11-17), each opt-in and local:**
  - VS Code window focus,
  - editor and terminal activity in the workspace (timestamps only, no content),
  - changes to the task file (`stage`, `next_action`),
  - AI tool session events (verified by E2). These come from an opt-in hook in the kit's Claude Code adapter. The hook writes only the event name, time, and session ID to a local file in the developer's home folder, never in the repository, and the extension reads that file. It is installed only by the approval-gated install command, into user-level Claude Code settings, and the same 90-day retention applies.
- **Retention:** 90 days for activity and feedback data, then automatic deletion. A "Delete all Repo Flow data" command is always available.
- ~~A light overview of which tasks have a session that has gone quiet.~~ Removed: Claude Code's agent view already covers this (overlap audit row 7).
- One-click "helpful / not now / not helpful" feedback on each nudge, stored locally.

**Exclusions:** notifications when VS Code or the laptop is off, email, SMS, external services, prediction, and sending data off the machine.

**Success criteria**
- [ ] Each approved signal is documented with what it does and doesn't mean, has its own opt-in setting, and is stored locally with 90-day retention (tested with a simulated clock).
- [ ] No nudge duplicates a notification the AI tool already sends (checked against the overlap audit).
- [ ] Feedback data is viewable and deletable from a command. The retention default is documented.
- [ ] Nudges are used for at least 5 working days, and feedback is recorded for at least 10 nudges.

**Dependencies:** 0.3.0 complete. The signals must be verified as observable.

**Open questions and risks**
- VS Code can't see whether you are thinking or reading on paper, so there will be false positives. Use conservative defaults and an easy snooze.
- Nudges that are too frequent cause reminder fatigue. Start with a low cap.

---

## Phase 0.5.0: Daily-use validation and local insights

**Purpose and scope**
- A defined dogfooding period (proposal: 3 weeks) using 0.4.0 in real work.
- A local insights view: tasks started per day, days with a resume after a break, and nudge feedback ratios.
- Tune the defaults (thresholds, caps, wording).
- **Repeat the overlap audit.** Remove any extension feature that AI tools now provide.
- Go/no-go decisions for 0.6.0 to 0.9.0, including where everyday tasks would be stored for 0.9.0.

**Exclusions:** hosted services, prediction models, and sharing data off the machine.

**Success criteria**
- [ ] Log or insight data exists for at least 15 working days.
- [ ] A written comparison against the 0.1.0 baseline uses the measurement method below. It is honest about n=1 and self-report limits.
- [ ] At least one reminder setting or wording is changed based on feedback, or a written rationale explains why none was needed.
- [ ] The overlap audit is updated, and any newly duplicated features are removed or scheduled for removal.
- [ ] A go/no-go decision for each of 0.6.0 to 0.9.0 is recorded in the Decision log.

**Dependencies:** 0.4.0 complete.

**Open questions and risks:** with a single user, results are anecdotal and won't prove causation. Treat them as decision support, not proof.

---

## Phase 0.6.0: Optional hosted reminders (laptop off)

**Purpose and scope**
- Deliver a reminder when the laptop or VS Code is off, for example a phone push notification at a planned start time.
- **Step 1 (decided 2026-11-17):** test whether a Claude Code cloud routine can deliver a "time to start task X" push to the phone (audit row 10). If it can, Repo Flow only helps set this up and builds no service. **Done 2026-11-17: failed** (push requested, not delivered; see experiment E3). Retest in the 0.5.0 overlap audit, since this Claude Code feature is in research preview.
- **Step 2, now the plan:** an external service is needed. Evaluate the options (self-hosted, a third-party push service, the reminders Microsoft To Do already has) for cost, privacy, and reliability.
- **Minimum data:** send only a time and a generic label by default. Never send task content or repository data.
- Email and SMS are candidate channels, in this phase or later, and only with an explicit decision.

**Exclusions:** sending repository content, prediction, and model routing.

**Success criteria**
- [ ] A written options comparison covers monthly cost, data sent, failure modes, and account requirements.
- [ ] The developer opts in through an explicit setting that shows exactly which fields are sent.
- [ ] A test reminder scheduled for when the laptop is off arrives on the chosen channel in 5 out of 5 trials.
- [ ] Turning the feature off deletes the scheduled reminders on the service (verified).

**Dependencies:** a go decision in 0.5.0.

**Open questions and risks:** ongoing operating cost and maintenance, credential handling, and whether the reminders Microsoft To Do already offers make this phase unnecessary.

---

## Phase 0.7.0: Session-usage prediction

**Purpose and scope**
- **First define "session usage"**: for example, the time an AI session is open versus actively exchanging messages versus making progress on a task stage. Use only signals that are reliably observable and were approved in 0.4.0.
- Explore limited predictions:
  - Is this session likely to stay idle?
  - When might a restart reminder help?
  - Which task suits the next session?
- Start with transparent rules or simple statistics, stored locally.

**Exclusions:** opaque models, cloud training, using repository content as input without separate consent, and judgments about character or discipline.

**Success criteria**
- [ ] A written definition of "session usage" and the list of signals, each with its consent status, retention period, and known blind spots.
- [ ] Predictions are evaluated retrospectively on local data with a stated metric (for example precision of "likely to stay idle") and compared against a fixed-timer baseline.
- [ ] Prediction-driven nudges are opt-in, separate from normal reminders, and show *why* they triggered.
- [ ] Over at least 2 weeks, prediction-driven nudges are rated helpful more often than fixed-timer nudges. If not, the feature stays off by default.

**Dependencies:** 0.5.0 go decision and at least 15 days of local history.

**Open questions and risks:** with small data, predictions may be no better than a timer. Each new signal raises the privacy concern, and the tool risks feeling like surveillance.

---

## Phase 0.8.0: Model and tool suggestions (conditional)

**Purpose and scope**
- AI coding tools already let you choose and switch models, so **Repo Flow builds nothing here by default**.
- This phase happens only if 0.5.0 identifies a concrete gap, for example suggesting *which AI tool* suits a task when you use several. Any suggestion is shown with its reason, never applied silently.

**Exclusions:** silent routing, sending repository content to a new provider without explicit per-provider consent, and reimplementing model selection that AI tools already offer.

**Success criteria**
- [ ] Either the gap is documented with evidence and an approved scope, or the phase is formally dropped in the Decision log.
- [ ] If built: workload categories and a tradeoff table (latency, cost, quality, privacy, context size) are approved, each provider has a data-handling note and a separate opt-in, and an evaluation on at least 10 real tasks uses a stated rubric.

**Dependencies:** 0.5.0 go decision.

**Open questions and risks:** models and tools change quickly, so maintaining this may cost more than it saves.

---

## Phase 0.9.0: Full daily task manager (Microsoft To Do replacement)

**Purpose and scope**
- Grow Repo Flow into the developer's **single daily task list**, replacing Microsoft To Do for everyday and development tasks.
- Microsoft To Do-style basics: My Day, lists, due dates and reminders, recurring tasks, and quick capture. These fit the core principle, because AI coding tools don't provide them.
- Migration from Microsoft To Do: a one-time import (proposal) or a transition period with sync (open decision).
- Daily reminders on devices other than the laptop, building on 0.6.0.

**Exclusions:** multi-user or team features, and anything that weakens the development-task rules.

**Success criteria**
- [ ] A written design resolves the storage question below and records it in the Decision log, including any approved change to the `docs/to-do/` boundary.
- [ ] All active Microsoft To Do tasks are imported, or a documented manual migration is done. Spot-checking 20 items shows no lost titles, due dates, or notes.
- [ ] The developer uses Repo Flow as the **only** daily task list for at least 4 consecutive weeks. The daily log records any day Microsoft To Do was needed, and why.
- [ ] Everyday tasks can be captured and checked off on the developer's phone, or a deliberate decision not to support that is recorded.
- [ ] Development tasks still work with the kit and the extension unchanged (regression tests pass).

**Dependencies:** 0.5.0 go decision and 0.6.0 (or an equivalent cross-device channel).

**Open questions and risks**
- **Conflict with the task-input boundary.** Everyday tasks don't belong in a code repository's `docs/to-do/`. Options:
  - a personal "tasks" repository with the same layout,
  - a separate store for non-repository tasks,
  - widening the boundary.
  This must be an explicit decision, deferred to the 0.5.0 go/no-go.
- **Leaving VS Code.** Microsoft To Do runs on phone, web, and desktop. Matching it likely needs a companion app, which is a large increase in scope, cost, and maintenance.
- **Since the extension is read-only on repository files,** capturing tasks on a phone needs a decided write path that respects the approval rules.
- **Microsoft Graph access** for import or sync needs authentication, consent, and token handling.
- **Privacy:** personal tasks need their own retention and consent rules.
- **Feature creep:** define the minimum Microsoft To Do features actually used before building.

---

## Unscheduled candidates

- **Microsoft To Do import or sync before 0.9.0:** open. Revisit in 0.5.0. If pursued earlier, it needs Microsoft Graph authentication and a consent design, and Microsoft To Do content must not become a task source unless the developer explicitly chooses that.

---

## Overlap audit (0.1.0 deliverable, repeated in 0.5.0)

**First pass: 2026-11-17.** Sources are the official docs at code.claude.com and developers.openai.com/codex, read through web search and page fetches. Some details came from summaries of long pages, marked "(summary)". Rows marked **Experiment needed** must be confirmed on the developer's machine before anything depends on them. Docs change often, so repeat this audit in 0.5.0.

Verdicts: **Provided** / **Partly provided** / **Not provided** (no feature found in the docs) / **Unverified**.

| # | Feature | Claude Code | Codex | Impact on Repo Flow |
|---|---|---|---|---|
| 1 | Edit task files, record decisions, `git mv`, commit | **Provided** (core function) | **Provided** (core function) | Already assigned to the AI tool through the kit. No change. |
| 2 | Follow a written workflow from repository instructions | **Provided**: skills are Markdown instructions Claude follows ([skills](https://code.claude.com/docs/en/skills)) | **Provided**: `AGENTS.md` guidance is loaded at session start ([config](https://developers.openai.com/codex/config-advanced)) | Confirms the kit approach. |
| 3 | Custom command or skill to start the workflow | **Provided**: `.claude/skills/<name>/SKILL.md` creates `/<name>`; the older `.claude/commands/` format still works ([skills](https://code.claude.com/docs/en/skills)) | **Provided**: `/skills` ([CLI reference](https://developers.openai.com/codex/cli/reference)). Custom prompts in `~/.codex/prompts/`: Unverified (third-party sources only) | Adapter: a personal Claude Code skill, `~/.claude/skills/repo-flow/` (`/repo-flow`), installed once per machine. |
| 4 | Resume a previous AI session | **Provided**: `claude --continue`, `claude --resume [name or id]` ([sessions](https://code.claude.com/docs/en/sessions)) | **Provided**: `codex resume` ([CLI reference](https://developers.openai.com/codex/cli/reference)) | Repo Flow doesn't track session history. The resume card may *suggest* the tool's own resume command. |
| 5 | Notify when a session is waiting for input | **Provided**: Notification hook types such as `permission_prompt`, `idle_prompt`, `agent_needs_input`, `agent_completed` ([hooks](https://code.claude.com/docs/en/hooks)). Phone push when it needs a decision (Remote Control; [mobile](https://code.claude.com/docs/en/mobile)) | **Partly provided**: `notify` runs a program on `agent-turn-complete` only ([config](https://developers.openai.com/codex/config-advanced)) | **Removed from Repo Flow.** It never sends "session is waiting" notifications. |
| 6 | Hooks or events that report session activity | **Provided, verified by E2** (no `Notification` seen when idle): `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `Stop`, `Notification` ([hooks](https://code.claude.com/docs/en/hooks)) | **Provided**: `SessionStart`, `SessionEnd`, `Stop` and others, via `hooks.json` or `config.toml` ([hooks](https://developers.openai.com/codex/hooks)) | The "AI tool session events" signal is feasible: a hook in the kit's adapter could record timestamps locally for the extension. **Experiment needed.** Design decision below. |
| 7 | Overview of many sessions: which are idle or need input | **Partly provided**: agent view (`claude agents`, CLI, research preview) shows background sessions as Working, Needs input, Idle, or Completed, across projects ([agent view](https://code.claude.com/docs/en/agent-view)). Interactive sessions in other terminals appear only once backgrounded | Not found | **Removed from Repo Flow:** the "overview of quiet sessions" in 0.4.0. Repo Flow keeps the *task* view, not a session view. |
| 8 | Open the AI tool with a prompt from VS Code | **Provided, verified by E1** (pre-fill works; VS Code asks to allow the link when it comes from outside VS Code): `vscode://anthropic.claude-code/open?prompt=…&session=…` pre-fills the prompt without submitting it; `claude-cli://` opens a terminal session ([VS Code](https://code.claude.com/docs/en/vs-code), [deep links](https://code.claude.com/docs/en/deep-links)) (summary). **Experiment needed** | Unverified | "Start session" can open Claude Code with the prompt pre-filled. You press Enter, and the clipboard stays as a fallback. Decision below. |
| 9 | Run AI work on a schedule | **Provided**: `/loop` (session open), desktop scheduled tasks (machine on), cloud routines (machine can be off, minimum 1 hour) ([scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks)) | Not checked | Different from Repo Flow's reminders. These run *the AI*; they don't remind *you* to start. Repo Flow doesn't build AI scheduling. |
| 10 | Phone notification while the laptop is off | **Partly provided**: push needs Remote Control with the machine on ([mobile](https://code.claude.com/docs/en/mobile)). Cloud routines run with the machine off, but the docs don't say whether a routine can push to your phone | Not found | **E3 failed (2026-11-17):** a cloud routine ran with the laptop asleep and requested the push, but it never reached the phone. **Kept in Repo Flow:** 0.6.0 evaluates a hosted service. Retest in 0.5.0. |
| 11 | Task list and next action visible **before** a session starts | **Not provided** (agent view lists sessions, not repository tasks) | Not found | **Kept.** This is Repo Flow's core. |
| 12 | Resume card when you return from a break | **Not provided** | Not found | **Kept.** |
| 13 | Break mode, working hours, planned-start reminders to the developer | **Not provided** (scheduling runs prompts, not reminders) | Not found | **Kept.** |
| 14 | Idle nudges tied to a task's next action, aware of working hours and breaks | **Not provided**. Notifications are about session state, not your planned work | Not found | **Kept**, but must never repeat row 5's notifications. |

**Result:** the extension's core (rows 11–14) is confirmed as not provided by either tool. Two planned items are removed: "session waiting" notifications and the multi-session overview. Three items needed a hands-on experiment (rows 6, 8, 10): E1 and E2 passed, and E3 failed.

**0.1.0 criterion: complete (2026-11-17).** The Codex column is informational only, because the developer won't use Codex.

**Experiments (0.1.0, on the developer's machine):**
- **E1, URI handler (row 8):** open `vscode://anthropic.claude-code/open?prompt=<encoded>` from a VS Code command or terminal. Pass: a Claude Code tab opens with the prompt pre-filled and unsent, in 3 out of 3 trials.
  - **Result (2026-11-17, WSL, Claude Code extension 2.1.326): passed with one caveat.**
    - Trial 1, opened through `cmd.exe start`: the tab opened in the right WSL window but the prompt was empty, because `cmd.exe` mangles `%`-encoded text.
    - Trials 2 and 3, opened through PowerShell `Start-Process`: the tab opened in the repo-flow WSL window with the prompt pre-filled and **not sent**.
    - Every time, VS Code first asked "Allow 'Claude Code for VS Code' extension to open this URI?" (one extra click). This is VS Code's prompt for links from outside VS Code.
  - **Follow-up for 0.3.0:** confirm the behavior, and whether the dialog appears, when the link is opened from *inside* VS Code by the Repo Flow extension.
- **E2, session-event hooks (row 6):** add a Claude Code hook (`SessionStart`, `UserPromptSubmit`, `Stop`, `Notification`) that writes only the event type and time to a local file outside the repository. Pass: events appear within seconds, and no prompt or code content is recorded. Then decide how events reach the extension (open decision).
  - **Result (2026-11-17): passed.**
    - The hooks were a project-local `.claude/settings.local.json` running a small Python logger that writes to `~/.repo-flow-experiment/events.log`.
    - `SessionStart`, `UserPromptSubmit`, and `Stop` were logged within seconds, each with its session ID, so separate sessions stay distinguishable.
    - **No prompt text was recorded.** The logger drops everything except the event name, time, and session ID, and a search of the log for the test prompts found nothing.
    - The already-running session also picked up the new hooks without a restart.
  - **`Notification` (for example `idle_prompt`) never fired** in about 5 minutes of idle time in the VS Code extension. It isn't needed: "time since the last `Stop` with no later `UserPromptSubmit`" gives how long a session has been waiting for the developer.
- **E3, cloud routine push (row 10):** create a cloud routine that, at a set time, reads a task's `next_action` and sends a phone push. Pass: the push arrives with the laptop off in 3 out of 3 trials. Result decides 0.6.0 step 1 or 2. *Note:* this runs in Anthropic's cloud on a copy of the repository, so it needs the developer's explicit OK about that data flow first.
  - **Result (2026-11-17): failed. The push was requested but never delivered.** A first-pass delivery-only test was run with the developer's OK.
    - Setup: a one-time cloud routine with no repository (GitHub isn't connected to the Claude account), no connectors, and only the `PushNotification` tool allowed. The laptop was asleep.
    - Run 1 was scheduled for 11:16:15 UTC and fired at 11:17:56 (about 1 minute 40 seconds late).
    - Run 2 was started manually at 11:23:53, after "Push when Claude decides" (`agentPushNotifEnabled`) was turned on.
    - In both runs, `PushNotification` returned "Mobile push requested." and the run succeeded.
    - Run 3 was started manually at 11:30:41, after the developer tried `claude rc` (Remote Control) to register the phone. Push requested at 11:30:46.
    - **Nothing arrived on the developer's phone in any of the 3 runs.** The phone was signed in to the same account with notifications allowed.
    - Likely cause (unconfirmed): the phone isn't registered for Claude Code pushes. `/config` ("No mobile registered") couldn't be checked, because the CLI isn't installed.
    - Reading `next_action` from the repository was not tested.
    - Cleanup: the push setting was removed afterwards, and the developer deletes the routine on claude.ai.

---

## Task file format

**Location:** `docs/to-do/[<subfolder>/]<id>-<slug>.md` (Decided). On completion: `docs/completed/[<same subfolder>/]<same filename>` (Decided).

```markdown
---
id: 20261117-1432-a7     # Decided: timestamp + short random suffix
title: Add retry to sync client
status: todo             # Decided: todo | in-progress | blocked | done
stage: impact-analysis   # Free text: the current step name from your flow (here, the default flow)
due: 2026-11-20          # Optional: date, or date + time (local time zone)
start_at: 2026-11-18 09:30   # Optional: planned start, triggers a local reminder
created: 2026-11-17
updated: 2026-11-17
next_action: Ask the AI tool to list call sites of SyncClient.fetch   # Decided
---

# Add retry to sync client

## Context
Why this task exists. Links to issues or a Microsoft To Do item (optional).

## Resume note
Where I left off and what to do first when I come back.

## Impact analysis
Files inspected: …  Observed / Assumed / Recommended.

## Documentation

## Questions and answers
Approved answers only.

## Decisions
- 2026-11-17: Decision, with rationale.

## Build notes

## Commit
Suggested message, commit hash once committed.

## Skipped stages
- documentation: skipped, one-line config change.

## Flow
<!-- Optional. Only if this task needs a different flow from the repository's. Overrides it entirely. -->
1. Building
2. Committing: approval required
```

**Who writes what:** the AI tool, following the kit and with approval, writes every field and section. The developer can also edit by hand. **The extension only reads.**

**Field notes**

| Field | Status | Notes |
|---|---|---|
| `id` | **Decided** | `YYYYMMDD-HHMM-<2–4 random chars>`. Stable across the move. Generated without reading `docs/completed/`. Also the filename prefix. |
| `title` | **Decided** | If missing, use the first `#` heading. If there is none, use the filename without the ID prefix. No warning is shown. |
| `status` | **Decided** | `todo`, `in-progress`, `blocked`, or `done`. A `done` task may stay in `docs/to-do/` until archived, and the extension shows it as "Done, awaiting archive". |
| `stage` | **Decided** (concept) | The name of the current step in the developer's flow. Not limited to the default six. Steps may be skipped for one task, with the reason recorded in "Skipped stages". |
| `due` | **Decided** | Optional. `YYYY-MM-DD` or `YYYY-MM-DD HH:MM`, interpreted in the computer's local time zone. |
| `start_at` | **Decided** | Optional, in front matter, same date or date + time format as `due`. The extension reads it to send a local reminder. A date-only value fires at the start of working hours. Set by the developer or the AI tool. |
| `next_action` | **Decided** | A one-line field in front matter **and** a freeform "Resume note" section. The kit's "pause" step updates both. |

**Discovery and display (extension; read-only)**
- Only `*.md` files under `docs/to-do/`, **including subfolders** (Decided).
- `README.md` files (any case) in `docs/to-do/` or its subfolders are **ignored** (Decided).
- Malformed-input handling (draft for the 0.1.0 criterion). Nothing is ever hidden, and every warning has a "Copy fix prompt" action:

| # | Case | Display | Status |
|---|---|---|---|
| 1 | No front matter | Listed as "needs setup" | Decided |
| 2 | Front matter that can't be parsed | Listed as "needs setup" | Decided |
| 3 | Missing `id` | Listed as "needs setup" | Decided |
| 4 | Two files with the same `id` | Both listed, with a conflict warning | Decided |
| 5 | Unknown `status` value | Listed under "Needs attention", with a warning | Decided |
| 6 | Invalid `due` or `start_at` | Listed, with a warning. No reminder is scheduled | Decided |
| 7 | `stage` not in the effective flow | Matched ignoring case and extra spaces; otherwise the stage text is shown with "not in flow" | Decided |
| 8 | Very large file (over about 1 MB) | Only the front matter is read (enough for the list and the next action), with a "large file" note | Decided |

**Completion move (executed by the AI tool, following the kit)**
- **Confirmation:** each move needs explicit confirmation. Marking a task done never moves it automatically.
- **Path:** mirror the subfolder (`docs/to-do/backend/x.md` → `docs/completed/backend/x.md`).
- **Collisions:** ID-prefixed filenames make them practically impossible. If the destination exists anyway, stop and ask. Never overwrite.
- **Access to `docs/completed/`:** only check whether the destination path exists. Never list or parse completed tasks.
- **Links:** rewrite relative links *inside* the moved file, and report other files that link to it without editing them.
- **Method:** `git mv` if the file is tracked, a plain move otherwise. Don't commit unless the developer approves.

---

## Flow definition (Decided)

**Decided (2026-11-17):**
- **Scope:** one **repository flow file** plus an optional **per-task override**, written as a "## Flow" section in the task file. If the task has an override, it replaces the repository flow for that task. There is no personal cross-repository default for now.
- **Format:** a plain numbered Markdown list. An optional note after a step name (after a colon) adds an approval checkpoint or a short instruction.
- **Extension access:** the extension may read the repository flow file (as configuration, not task input) and a task's own "## Flow" section, to show progress.

The AI tool reads the effective flow at the start of each session. The default flow:

```markdown
## Flow
1. Impact analysis
2. Documentation
3. Questions and answers
4. Recording decisions
5. Building: approval before changing project files
6. Committing: approval required
```

- Steps can be renamed, reordered, added, or removed by editing this list.
- Required approvals (firm requirement 7) apply whatever the flow says.
- **Locations (decided, revised 2026-11-17), highest priority first:**
  1. the task's `## Flow` section,
  2. the optional repository flow `docs/repo-flow/flow.md`,
  3. the personal default `~/.claude/skills/repo-flow/flow.md`.

  The workflow instructions are in `~/.claude/skills/repo-flow/workflow.md`. Their source is `kit/repo-flow/` in this repository.

---

## Measurement method

These outcomes are about behavior and can't be measured automatically with any confidence. The proposed method:

- **Daily log (baseline in 0.1.0, continued after):** three yes/no answers, taking under a minute:
  1. Did I start a planned development task today?
  2. After my longest break, did I resume work that day?
  3. Did at least one AI coding session make meaningful progress on a task stage?
- From 0.3.0: local counts of reminders shown, snoozed, and dismissed. From 0.4.0: nudges rated helpful or not helpful.
- Report the results as counts and proportions over days, with the n=1, self-report, and no-control-group limits stated. Don't claim causation.

---

## Open decisions (summary)

Resolved items are in the Decision log. Still open:

**Need checking during a later phase:**
1. The personal Claude Code skill at `~/.claude/skills/repo-flow/` (supported according to the docs, audit row 3). Confirm it on a real task in 0.2.0.
2. Retest E3 (cloud routine push) in the 0.5.0 overlap audit.

**Deferred to a later phase by design:**
3. Microsoft To Do: interim integration before 0.9.0. For the replacement: migration, storage for everyday tasks, any boundary change, and a delivery surface beyond VS Code (0.5.0 go/no-go).
4. Hosted reminder channel and provider (0.6.0). E3 failed, so this is needed unless a 0.5.0 retest passes.
5. Definition of "session usage" (0.7.0).
6. Whether 0.8.0 happens at all (0.5.0).

## Decision log

| Date | Decision | Rationale |
|---|---|---|
| 2026-11-17 | Completing a task requires confirmation for each move. | Consistent with the human-approval rule. |
| 2026-11-17 | If the move is declined, the task stays `done` in `docs/to-do/` and is shown as "Done, awaiting archive". | Doesn't undo the developer's "done". The move can happen later. |
| 2026-11-17 | Task IDs are `YYYYMMDD-HHMM-<random>`, and filenames are `<id>-<slug>.md`. | Unique without reading `docs/completed/`. Prevents filename collisions. |
| 2026-11-17 | Filename collision on move: stop and ask, never overwrite. | Safe fallback for a rare case. |
| 2026-11-17 | Access to `docs/completed/` is limited to destination existence checks. Completed tasks are never listed or parsed. | Keeps the task-input boundary strict. |
| 2026-11-17 | Discovery includes subfolders of `docs/to-do/`. Moves mirror the subfolder path. | Subfolders work as lists, and grouping is kept after completion. |
| 2026-11-17 | `next_action` lives in front matter, with a "Resume note" section for context. | Easy to show in the UI, with room to explain where work stopped. |
| 2026-11-17 | Lifecycle stages may be skipped, with the reason recorded. | Keeps small tasks light without losing the record. |
| 2026-11-17 | Relative links inside a moved file are rewritten. Inbound links are reported, not edited. | Avoids editing files outside the task boundary. |
| 2026-11-17 | Move with `git mv` when tracked, a plain move otherwise. | Clear rename history. |
| 2026-11-17 | Extension language: TypeScript. | Standard VS Code extension toolchain. |
| 2026-11-17 | Storage for everyday tasks is deferred to the 0.5.0 go/no-go. | Decide with real usage data. |
| 2026-11-17 | **Core principle: build only what AI coding tools can't do.** The extension is the attention layer (visibility, start and resume, break mode, reminders, nudges). The workflow, file edits, moves, and commits belong to the AI tool, following a Markdown workflow kit. | Developer direction. AI tools already do the heavy lifting. The extension's value is acting before a session starts and after a break. |
| 2026-11-17 | The extension is read-only on repository files. | Follows from the core principle. Also removes a class of approval and conflict risks. |
| 2026-11-17 | ~~Malformed front matter gets an approval-gated fix action in the extension.~~ Superseded: the extension offers "Copy fix prompt", and the AI tool makes the fix. | The extension is read-only. |
| 2026-11-17 | ~~"Start session" writes `.repo-flow/session-brief.md` and copies a prompt.~~ Superseded: "Start session" only copies a short prompt naming the task file and the kit. | The AI tool can read the task file itself, so a generated brief duplicates work and writes to the repository. |
| 2026-11-17 | The workflow kit (Markdown, tool-neutral, with thin per-tool adapters) comes before the extension MVP. The MVP moves to 0.3.0. | The kit is cheap, usable at once, and settles the conventions the extension reads. |
| 2026-11-17 | Model routing (0.8.0) is conditional and likely dropped. | AI tools already provide model selection. |
| 2026-11-17 | Flow scope: a repository flow file plus an optional per-task "## Flow" override, where the task wins. No personal default for now. | Simple, and it still fits tasks that need a different shape. |
| 2026-11-17 | Flow format: a numbered Markdown list with optional notes after a colon (for example "Committing: approval required"). | Easy to edit by hand and reliably read by AI tools. |
| 2026-11-17 | The extension may read the flow file (as configuration) and a task's "## Flow" section, to show step progress. This is a named exception to reading only `docs/to-do/`. | A "Step 3 of 5, next: …" hint helps with resuming. Limited to exactly one non-task file. |
| 2026-11-17 | The task flow is user-defined and editable. The six steps are only the default. The AI tool follows the developer's flow, and nothing hardcodes step names. Required approvals can't be removed by a flow. | Developer direction. People work differently, and the flow should fit the individual. |
| 2026-11-17 | ~~Kit location: `docs/repo-flow/` (the flow file is `docs/repo-flow/flow.md`).~~ Superseded the same day: see the install-once decision below. | Visible, next to `docs/to-do/`, and committed with the repository. |
| 2026-11-17 | ~~Kit installation: an extension command, "Repo Flow: Set up this repository", copies the kit after showing the files and getting approval. A named exception to the read-only rule.~~ Superseded the same day. | One deliberate, approval-gated step is more predictable than asking an AI tool to recreate the kit. |
| 2026-11-17 | `start_at` (planned start) lives in task front matter. | Visible and tool-neutral, and the extension stays read-only. |
| 2026-11-17 | Status values: `todo`, `in-progress`, `blocked`, `done`. | `blocked` explains why a task isn't the next one to pick. |
| 2026-11-17 | Duplicate IDs: show both files with a conflict warning and a fix prompt. | Nothing is hidden. |
| 2026-11-17 | `README.md` files in `docs/to-do/` are ignored. | Lets lists be documented without noise. |
| 2026-11-17 | `due` and `start_at` accept a date, or a date and time, in the local time zone. | Needed for start-time reminders, and simple to type. |
| 2026-11-17 | `stage` matching ignores case and extra spaces. Otherwise the extension warns "not in flow". | Tolerates small edits and flags real mismatches. |
| 2026-11-17 | Approved idle signals: window focus, editor and terminal activity (timestamps only), task file changes, and AI tool session events (only if verified). Each signal is opt-in and local. | The developer chose all four. Combined, they reduce false "idle" readings. |
| 2026-11-17 | Retention: 90 days for activity and feedback data, with automatic deletion and a "Delete all" command. | Covers the 0.5.0 validation and 0.7.0 history needs. |
| 2026-11-17 | Minimum VS Code version: the version current at scaffolding, minus about 3 months. Tooling: the official extension generator defaults, verified at scaffolding. | Latest APIs and the least custom setup. |
| 2026-11-17 | Malformed fields: a missing `id` → "needs setup"; an unknown `status` → "Needs attention"; an invalid date → a warning and no reminder. | Nothing is hidden, and only real setup problems keep a task out of the normal list. |
| 2026-11-17 | Task files over about 1 MB: only the front matter is read, with a "large file" note. | Keeps VS Code responsive and still shows the next action. |
| 2026-11-17 | The install command never overwrites the developer's `flow.md`. (Originally written for the per-repo set-up command.) | Protects the developer's edited `flow.md`. |
| 2026-11-17 | Kit files: `workflow.md` (instructions for the AI tool) and `flow.md` (the editable steps), now in `~/.claude/skills/repo-flow/`. | Keeping the developer's flow separate from the kit's instructions makes kit upgrades safe. |
| 2026-11-17 | A missing `title` falls back to the first `#` heading, then to the filename without the ID prefix. | The task still looks normal, and a title isn't worth a warning. |
| 2026-11-17 | Overlap audit, first pass: "session waiting for input" notifications and the multi-session overview are removed from Repo Flow (Claude Code provides them). Task visibility, resume cards, break mode, and planned-start reminders and nudges are confirmed as gaps. | Evidence in the "Overlap audit" section. |
| 2026-11-17 | Start session for Claude Code: open a tab with the prompt pre-filled through the documented URI handler. Clipboard as the fallback. Depends on experiment E1. | One fewer step to start, and the developer still presses Enter. |
| 2026-11-17 | How session events reach the extension is decided after experiment E2. | The developer wants to see the hook approach working first. |
| 2026-11-17 | Phase 0.6.0 first tests Claude Code cloud routines for laptop-off pushes (E3) before evaluating any hosted service. | Avoids building a service if the AI tool already covers it. |
| 2026-11-17 | Experiments E1 and E2 done. E1: pre-filled launch works (PowerShell; `cmd.exe` breaks the encoding), with VS Code's allow-link dialog. E2: hooks log events with no content; no idle `Notification` seen. | Evidence in "Overlap audit" → Experiments. |
| 2026-11-17 | Session events reach the extension through an opt-in kit hook that writes event name, time, and session ID to a local file in the home folder. The extension reads it. Idle time = time since the last `Stop` with no later `UserPromptSubmit`. | Proven by E2, keeps content out, and doesn't depend on `Notification`. |
| 2026-11-17 | E3 failed: the cloud routine fired with the laptop asleep and requested a push twice, but nothing reached the phone. Phase 0.6.0 moves to step 2 (evaluate hosted services). Routine pushes are retested in the 0.5.0 audit. | Laptop-off reminders can't depend on a research-preview feature that didn't deliver. |
| 2026-11-17 | Codex is out of scope. The developer won't use it. The kit stays tool-neutral Markdown, but only the Claude Code adapter is built. | Avoids work for a tool that won't be used. Another adapter can be added later if needed. |
| 2026-11-17 | Phase 0.1.0 complete. Phase 0.2.0 (workflow kit) started. | All nine 0.1.0 success criteria met. The baseline caveat is noted on its criterion. |
| 2026-11-17 | **Install once, not per repo.** The kit (skill, workflow, template, and the personal default flow) is installed once per machine in `~/.claude/skills/repo-flow/`, from its source in `kit/repo-flow/`. A repository needs only `docs/to-do/`, plus an optional `docs/repo-flow/flow.md` override. Effective flow: task `## Flow`, then repository flow, then personal default. The extension becomes fully read-only on repositories. Its install command writes only to the home folder. Supersedes the per-repo kit and set-up command, and resolves the deferred "personal default flow" item. | No copies drifting apart between repositories, and no Repo Flow files committed into work repositories. Claude Code supports personal skills. Trade-off: the kit doesn't travel with a repository, which is fine for a single-developer tool. |

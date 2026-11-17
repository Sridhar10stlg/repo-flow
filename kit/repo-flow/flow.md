# Flow

This is a flow: the steps a task goes through. Edit it freely. Rename, reorder, add, or remove steps.

Where flows live, highest priority first:
1. A task's own `## Flow` section, for that task only.
2. `docs/repo-flow/flow.md` in a repository, for that repository only. Optional.
3. `~/.claude/skills/repo-flow/flow.md`, the personal default for every repository.

- One step per numbered line.
- An optional note after a colon adds an instruction or an approval checkpoint for that step.
- Required approvals in `workflow.md` apply whatever this list says. A flow can add checkpoints but can't remove those.

## Flow
1. Impact analysis
2. Documentation
3. Questions and answers
4. Recording decisions
5. Building: approval before changing project files
6. Committing: approval required

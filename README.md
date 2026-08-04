# ticket-workflow

A Claude Code skill that turns a ticket into shipped, verified work through six sequential phases:

1. **Analyze** — parse the ticket into a stable spec table (`SPEC-1`, `SPEC-2`, …), list ambiguities with the assumption chosen and the risk if wrong, and map each requirement to real files in the repo.
2. **Plan** — an ordered, SPEC-mapped implementation plan covering migrations, permissions, performance and rollback.
3. **Implement** — direct edits following the repo's own conventions, with a running summary of files created/modified/deleted.
4. **Validate** — three separate passes (spec compliance, hostile code review, security review) plus every linter, type check and test suite the repo provides, with real command output.
5. **Manual test plan** — delivered interactively, **one check per message**, never advancing until the tester reports the result. Grouped by the ticket's own issue/AC numbering.
6. **PR description** — the repo's required format: Jira link, Motivation and Context, Description of changes, including migration runtime and performance impact.

Phases 1-4 run without stopping for approval. Phase 5 stops at every single check.

## Install

Drop the directory into your skills folder:

```bash
git clone <this repo> ~/.claude/skills/ticket-workflow
```

Then invoke it with `/ticket-workflow <ticket-url-or-key>`, or just paste a ticket — the skill's description triggers on ticket URLs, ticket keys, "work on this ticket", and similar.

## Layout

| Path | Purpose |
|------|---------|
| `SKILL.md` | The workflow: non-negotiable rules plus the six phases |
| `templates/spec-table.md` | Phase 1 spec + ambiguity tables |
| `templates/coverage-matrix.md` | Phase 4 spec-compliance matrix |
| `templates/manual-test-plan.md` | Phase 5 interactive check-by-check protocol |
| `templates/pr-description.md` | Phase 6 PR body format |

## Notes

- Codebase-agnostic: conventions (test commands, linters, file layout) are discovered from the repo at runtime rather than hardcoded.
- `templates/pr-description.md` references a Jira base URL — change it to your own instance.

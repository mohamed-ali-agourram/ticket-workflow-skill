# ticket-workflow

A Claude Code skill for storiq that turns a Jira ticket into shipped, verified work:

0. **Intake**: fetches the ticket (description, comments, sub-tasks, parent, attachments) from Jira and finds any existing branch or PR. It then picks a mode: **New**, **Rework** (QA sent it back), **Review** (PR review comments) or **Resume**.
1. **Analyze**: a Sonnet `Explore` agent maps the code and every entry point to the behaviour. The output is a compact `SPEC-n → files` list plus assumptions.
2. **Plan**: one line per step, plus a risks line when relevant.
3. **Implement**: direct edits that follow repo conventions, including RSpec/Jest tests for the change.
4. **Validate**: changed-files-only linters and specs run in the background while a fresh-context agent does a hostile code and security review. The output is a short coverage, review and checks summary.
5. **Manual test**: one check per message, verified against the real UI, with the data setup done through the Rails console. Lives in `phases/5-manual-test.md`.
6. **Ship**: automatic once every check passes. It does a safety check, a delta review, commits by mode, pushes, creates or updates the PR, monitors CI until the OBT is deployed (never approving QA gates) and moves Jira to Code review once tests are green. Lives in `phases/6-ship.md`.

Phases 0-4 run without approval stops. Phase 5 stops at every check. Phase 6 runs by itself.

## Layout

| Path | Purpose |
|------|---------|
| `SKILL.md` | Rules and Phases 0-4 (always loaded) |
| `phases/5-manual-test.md` | Interactive manual test protocol (read on reaching Phase 5) |
| `phases/6-ship.md` | Commit, push, CI, Jira and PR steps (read on reaching Phase 6) |
| `templates/pr-description.md` | PR body format |

## Install

```bash
git clone <this repo> ~/.claude/skills/ticket-workflow
```

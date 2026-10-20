# ticket-workflow

A project-agnostic Claude Code skill that turns a ticket into shipped, verified work.

Usage: `/ticket-workflow <TICKET-KEY>`, or paste the ticket or requirements.

0. **Intake**: fetches the ticket (description, comments, sub-tasks, parent, attachments) from the tracker and finds any existing branch or PR. It then picks a mode: **New**, **Rework** (QA sent it back), **Review** (PR review comments) or **Resume**.
1. **Analyze**: a Sonnet `Explore` agent maps the code and every entry point to the behaviour. The output is a compact `SPEC-n → files` list plus assumptions.
2. **Plan**: one line per step, plus a risks line when relevant.
3. **Implement**: direct edits that follow repo conventions, including tests for the change.
4. **Validate**: changed-files-only linters and tests run in the background while a fresh-context agent does a hostile code and security review.
5. **Manual test**: one check per message, verified against the real UI, with the data setup done by Claude.
6. **Ship**: automatic once every check passes. It does a safety check, a delta review, commits by mode (no Claude signature), pushes, creates or updates the PR, moves the ticket to Code review once tests are green, and monitors CI until the review environment is deployed. It never approves QA gates.

Phases 0-4 run without approval stops. Phase 5 stops at every check. Phase 6 runs by itself.

## Project bindings

Project facts are not hardcoded. Claude reads them from `CLAUDE.md` and its memory:

- tracker access, statuses and transitions
- branch and commit naming
- check commands
- dev URL and user switching
- console
- PR conventions
- CI jobs and gates

When a binding is missing, Claude works it out once from the repo and saves it to memory.

## Layout

| Path | Purpose |
|------|---------|
| `SKILL.md` | Rules, project bindings, Phases 0-4 (always loaded) |
| `phases/5-manual-test.md` | Interactive manual test protocol (read on reaching Phase 5) |
| `phases/6-ship.md` | Commit, push, PR, CI and tracker steps (read on reaching Phase 6) |
| `templates/pr-description.md` | PR creation/update and body format |

## Install

```bash
git clone <this repo> ~/.claude/skills/ticket-workflow
```

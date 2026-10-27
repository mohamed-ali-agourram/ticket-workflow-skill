---
name: ticket-workflow
description: >
  End-to-end ticket workflow. Usage: `/ticket-workflow <TICKET-KEY>` (or paste the ticket or requirements).
  It fetches the ticket from the tracker, finds any existing branch or PR, and picks a mode: new ticket, QA rework,
  PR review comments, or resume. Then it runs on its own: analyzes the code, plans, implements with tests, and
  validates with linters, specs and an independent code/security review. The first stop for the user is the
  manual test, given one check per message and waiting for pass/fail; failures are fixed and re-checked. Once
  every check passes it ships without asking: final spec/code check, commit (no Claude signature), push,
  create or update the PR in the repo's format, ticket to Code review, and CI monitored until the review/QA
  environment is deployed. Use when the user shares a ticket key, pastes requirements, or says QA sent it back
  or there are review comments to address.
---

# Ticket Workflow

Phases run in order: **0 Intake → 1 Analyze → 2 Plan → 3 Implement → 4 Validate → 5 Manual test → 6 Ship.**
Phases 5 and 6 have their own files; read each one only when you reach it.

## Project bindings

The skill is project-agnostic. Project facts come from **CLAUDE.md and memory**, never from guesses:

- tracker access, ticket statuses and transitions
- branch naming, commit prefix, the no-ticket key used for CI-only commits
- how to run the console, linters and tests (host or container)
- dev base URL and how to switch users
- PR conventions: title, base, labels, draft, and grouping branches for epics
- CI pipeline: test jobs, approval gates, deploy-to-review-env job, audit ignore files

If a binding is missing, work it out once from the repo (CLAUDE.md, CI config, PR template, recent PRs and commits) and **save it to memory** so later runs don't re-derive it.

## Rules

1. **No approval stops before Phase 5.** Go from Phase 0 straight through Phase 4. The user's first stop is the Phase 5 test. Ask earlier only for a genuine blocker (no reasonable default exists) or a destructive / hard-to-reverse step.
2. **Phase 5 is one check per message**, gated on the user's pass/fail reply. **Phase 6 runs by itself** once every check has passed or been skipped. That includes commit, push, PR, ticket status and CI.
3. **Keep chat output short.** The user reads results, not bookkeeping. Spec lists, plans, coverage and reviews are compact: one line per item, no long tables, no restating the ticket. Never paste code blocks as the deliverable; edit files.
4. **Evidence, not claims.** Never say a check passed without running it. Show the command and its result line (e.g. `12 examples, 0 failures`), not the full log.
5. **SPEC-IDs in Phases 1-4 only.** Phase 5 and the PR use the ticket's own Issue / AC numbering.
6. **Assumptions are flagged, never silently resolved.** Pick the lowest-risk default, state it with its risk, and keep going.
7. **Close every path to the ticket's behaviour in the same change.** That means UI buttons, alternate entry points, direct URLs (server-side guard), API endpoints/mutations and the service/use-case layer. Never write "out of scope / follow-up ticket" for a hole that defeats the ticket's goal.
8. **Scope growth: ask first.** If you find a gap that is related to the ticket but not part of it, or the work grows well beyond the plan, report it in one line and ask before starting. Don't run silently for a long time; give a one-line status at each milestone.
9. **Reuse before you add.** Before adding a filter, flag, helper or component, grep for an existing one and use it.
10. **Never** disable lint rules, bump dependencies, add Claude signatures to commits/PRs, push from a subagent, or approve CI approval gates that belong to QA or other people.
11. **Second agents are Codex, not Claude.** Any delegated or parallel work (code mapping, independent review, side investigations, parallel fixes) runs as a background `codex exec --yolo -C <dir> -o <scratchpad>/<name>.txt "<task>" </dev/null` (stdin must be closed or it hangs), then read the result file. Codex starts cold: put the full context and the user's rules in the task (no commit/push/env changes, git inside the container if the project says so, read-only unless editing is the task). Give parallel editors their own worktree. If Codex fails (usage/quota limit, auth error, non-zero exit, empty output), fall back to a Claude agent for that task and say so in one line. Split work across whatever agents are actually available; never block on a missing one. Verify every agent's output yourself before it reaches the user.
12. **Get a Codex second opinion whenever it's worth it**, without being asked: root-cause diagnoses, fix designs and plans, risky changes (migrations, permissions, performance rewrites), expected values in the Phase 5 checks, and any "done/correct" claim. Run it in parallel with your own work, report agreement or the disagreement in one line, and resolve a disagreement before acting.

---

## Phase 0 — Intake (fetch and choose the mode)

1. **Fetch the ticket** from the tracker, using the access in memory (e.g. Jira REST + stored credentials; flatten Jira's ADF description to plain text with Python). Get the description, status, comments, sub-tasks, parent and attachments. Read the parent too if this is a sub-task (QA sub-tasks hold the original requirements in their parent). For image attachments that matter, download them to the scratchpad and look at them.
2. **Look for existing work:** `git branch -a --list "*<n>*" "*<parent-n>*"`, `git log <base>..<branch> --oneline` and `gh pr list --head <branch> --json number,url,reviewDecision`.
3. **Pick the mode:**

   | Situation | Mode | What to do |
   |---|---|---|
   | No branch yet, or its PR is already merged | **New** | If the ticket's epic has a grouping branch (per the PR conventions in memory), work on that branch. Otherwise create a branch per the project's naming convention off the base branch. Then Phase 1. |
   | QA sub-task, or ticket in a rework status, with an existing branch | **Rework** | Check out the existing branch. Phase 1 covers only the reported issues. Fixes are folded into that ticket's commit (Phase 6). |
   | Open PR with unresolved review comments the user wants addressed | **Review** | Fetch them with `gh api repos/{owner}/{repo}/pulls/<pr>/comments`. Each comment is a SPEC. Fixes go in a **separate** commit. Phase 5 tests only behaviour that changed. |
   | Branch with uncommitted work for this ticket | **Resume** | Find out which phase was reached from the diff and continue from there. |

4. **Before switching branches:** if the working tree has another ticket's uncommitted work, don't stash or discard it. Ask, offering a git worktree.
5. **Tracker:** in New mode, if the ticket is still in a to-do status, move it to the in-progress status.

## Phase 1 — Analyze

1. **Delegate the code mapping** to a read-only Codex agent (Rule 11; fallback an `Explore` agent with `model: "sonnet"`) so file dumps stay out of the main context. Give it the ticket text and ask it to return, with `file:line`:
   - the code that implements each requirement today
   - **every entry point** that reaches the affected behaviour: UI pages/links, routes, controllers/handlers, API queries/mutations, service layer, background jobs
   - existing patterns, filters, flags or hooks that can be reused
   - existing tests that cover the area

   For a small ticket (one or two obvious files) skip the agent and read them directly.
2. Split the ticket into requirements `SPEC-1, SPEC-2…` (stable for the rest of the conversation). Include the implicit ones: edge cases, permissions, empty states.
3. **Output (compact):**
   ```
   SPEC-1 <requirement> (AC 1) → path/to/file.rb, path/to/Component.tsx
   SPEC-2 …
   Assumptions: A1 <assumption chosen> — risk <low|med|high>
   ```

## Phase 2 — Plan

Write an ordered list of steps, one line each: `<n>. (SPEC-x) <change> — <files>`. Add a single `Risks:` line only when something applies: migration (and its rollback), permissions/tenant scoping, N+1/indexes, backwards compatibility. Then go straight into Phase 3.

## Phase 3 — Implement

1. Edit files directly. Follow CLAUDE.md and the surrounding code: architecture layers, naming, i18n, UI components, data-fetching patterns, comment density.
2. **Write tests for the change** with the repo's test frameworks. Cover backend behaviour (services, models, endpoints, permissions), plus frontend logic and components where non-trivial. In Rework/Review mode, add a test that reproduces the reported bug when that's feasible.
3. After each logical unit, write one line: `Modified path/to/x — <what>`.

## Phase 4 — Validate

Run these in parallel:

- **Automated checks**, started with `run_in_background`, using the repo's lint/test commands (CLAUDE.md, CI config) and limited to what changed:
  - linters on the changed files
  - new and related tests
  - when a migration was added: migrate, roll back, migrate again (note the runtime; follow any schema-file rule in memory)
- **Independent review:** start one Codex agent in the background (Rule 11; fallback a fresh `general-purpose` agent, default model). Pass it the ticket text, the SPEC list and the branch name, and tell it to review `git diff <base>...HEAD` plus the uncommitted diff as a hostile senior reviewer. It should cover:
  1. every SPEC implemented
  2. every entry point from Phase 1 guarded (Rule 7)
  3. correctness and edge cases
  4. authz and tenant/data scoping, IDOR, mass assignment, injection, XSS
  5. N+1 and work repeated per render
  6. dead code, duplication, reuse of existing helpers
  7. missing tests

  Ask for real findings only, each with `file:line`, severity and a fix, and no praise or style nitpicks covered by linters. It must not edit files.

Fix the confirmed findings and re-run only the affected checks. **Output:**
```
Coverage: SPEC-1 ✓ foo.rb:42 · SPEC-2 ✓ Bar.tsx:88 · …
Review: <n> findings fixed — <one line each> (or "no findings")
Checks: lint ✓ (3 files) · backend tests 14 examples, 0 failures · frontend tests 6 passed
Verdict: PASS
```
Any SPEC not covered, or any red check → back to Phase 3.

## Phase 5 — Manual test

Read `phases/5-manual-test.md` and follow it.

## Phase 6 — Ship

Once every check has passed or been skipped, read `phases/6-ship.md` and follow it.

---
name: ticket-workflow
description: >
  Takes a storiq Jira ticket from start to merged-ready PR: implements, tests, validates, walks the user through
  manual checks, then commits, pushes, watches CI and updates Jira and the PR. Use when the user shares a ticket
  key (e.g. STOR-1234), pastes requirements, or says QA sent it back or there are review comments to address.
---

# Ticket Workflow (storiq)

Phases run in order: **0 Intake → 1 Analyze → 2 Plan → 3 Implement → 4 Validate → 5 Manual test → 6 Ship.**
Phases 5 and 6 have their own files; read each one only when you reach it.

## Rules

1. **No approval stops before Phase 5.** Go from Phase 0 straight through Phase 4. The user's first stop is the Phase 5 test. Ask earlier only for a genuine blocker (no reasonable default exists) or a destructive / hard-to-reverse step.
2. **Phase 5 is one check per message**, gated on the user's pass/fail reply. **Phase 6 runs by itself** once every check has passed or been skipped. That includes commit, push, CI, Jira and PR.
3. **Keep chat output short.** The user reads results, not bookkeeping. Spec lists, plans, coverage and reviews are compact: one line per item, no long tables, no restating the ticket. Never paste code blocks as the deliverable; edit files.
4. **Evidence, not claims.** Never say a check passed without running it. Show the command and its result line (e.g. `12 examples, 0 failures`), not the full log.
5. **SPEC-IDs in Phases 1-4 only.** Phase 5 and the PR use the ticket's own Issue / AC numbering.
6. **Assumptions are flagged, never silently resolved.** Pick the lowest-risk default, state it with its risk, and keep going.
7. **Close every path to the ticket's behaviour in the same change.** That means UI buttons, alternate entry points, direct URLs (controller guard), GraphQL mutations and use cases (`validate!`). Never write "out of scope / follow-up ticket" for a hole that defeats the ticket's goal.
8. **Scope growth: ask first.** If you find a gap that is related to the ticket but not part of it, or the work grows well beyond the plan, report it in one line and ask before starting. Don't run silently for a long time; give a one-line status at each milestone.
9. **Reuse before you add.** Before adding a filter, flag, helper or component, grep for an existing one and use it.
10. **Never** disable lint/RuboCop rules, bump dependencies, add Claude signatures to commits/PRs, push from a subagent, or approve CircleCI approval jobs.

---

## Phase 0 — Intake (fetch and choose the mode)

1. **Fetch the ticket** from Jira and flatten the ADF description to plain text with Python:
   ```bash
   source ~/.config/jira/credentials
   curl -s -u "${JIRA_EMAIL}:${JIRA_API_TOKEN}" "$JIRA_SITE/rest/api/3/issue/STOR-<n>?fields=summary,description,status,parent,subtasks,issuelinks,comment,attachment"
   ```
   Read the comments too, and the parent ticket if this is a sub-task (QA sub-tasks hold the original requirements in their parent). For image attachments that matter, download them to the scratchpad and look at them.
2. **Look for existing work:** `git branch -a --list "*<n>*" "*<parent-n>*"`, `git log develop..<branch> --oneline` and `gh pr list --head <branch> --json number,url,reviewDecision`.
3. **Pick the mode:**

   | Situation | Mode | What to do |
   |---|---|---|
   | No branch yet, or its PR is already merged | **New** | If the ticket belongs to an epic that has a grouping branch (see the PR conventions memory, e.g. Smart Reports → `STOR-9815-smart-reports`), work on that branch. Otherwise branch `STOR-<n>-<short-desc>` (≤ 25 chars) off `develop`. Then Phase 1. |
   | QA sub-task, or ticket in `QA Rework` / `Code Rework`, with an existing branch | **Rework** | Check out the existing branch. Phase 1 covers only the reported issues. Fixes are amended into that ticket's commit (Phase 6). |
   | Open PR with unresolved review comments the user wants addressed | **Review** | Fetch them with `gh api repos/cowcross/storiq/pulls/<pr>/comments`. Each comment is a SPEC. Fixes go in a **separate** commit. Phase 5 tests only behaviour that changed. |
   | Branch with uncommitted work for this ticket | **Resume** | Find out which phase was reached from the diff and continue from there. |

4. **Before switching branches:** if the working tree has another ticket's uncommitted work, don't stash or discard it. Ask, offering a worktree via `/wt`.
5. **Jira:** in New mode, if the ticket is `Open`, `Staged` or `Reopened`, move it to in progress (the transition named `In Progress…`). Find the id with `GET .../issue/STOR-<n>/transitions` and `POST` it.

## Phase 1 — Analyze

1. **Delegate the code mapping** to an `Explore` agent with `model: "sonnet"` so file dumps stay out of the main context. Give it the ticket text and ask it to return, with `file:line`:
   - the code that implements each requirement today
   - **every entry point** that reaches the affected behaviour: React pages/links, routes, controllers, GraphQL queries/mutations, use cases, jobs
   - existing patterns, filters, flags or hooks that can be reused
   - existing specs that cover the area

   For a small ticket (one or two obvious files) skip the agent and read them directly.
2. Split the ticket into requirements `SPEC-1, SPEC-2…` (stable for the rest of the conversation). Include the implicit ones: edge cases, permissions, empty states.
3. **Output (compact):**
   ```
   SPEC-1 <requirement> (AC 1) → app/…/foo.rb, app/javascript/…/Bar.tsx
   SPEC-2 …
   Assumptions: A1 <assumption chosen> — risk <low|med|high>
   ```

## Phase 2 — Plan

Write an ordered list of steps, one line each: `<n>. (SPEC-x) <change> — <files>`. Add a single `Risks:` line only when something applies: migration (and its rollback), permissions/tenant scoping, N+1/indexes, backwards compatibility. Then go straight into Phase 3.

## Phase 3 — Implement

1. Edit files directly. Match the surrounding code and CLAUDE.md conventions: use cases in `app/src/services`, i18n for every string, CDS components, the data hooks, no new comments.
2. **Write tests for the change.** Use RSpec for backend behaviour (use cases, models, controllers/resolvers, permissions) and Jest for non-trivial pure functions or components. In Rework/Review mode, add a spec that reproduces the reported bug when that's feasible.
3. After each logical unit, write one line: `Modified app/…/x.rb — <what>`.
4. Rails console data checks: `docker exec storiq-web-1 bin/rails runner "…"`.

## Phase 4 — Validate

Run these in parallel:

- **Automated checks**, started with `run_in_background`, limited to what changed. Run them inside `storiq-web-1` like the rest of the dev env:
  - `bundle exec rubocop <changed .rb files>`
  - `yarn eslint <changed .ts/.tsx files>`
  - `bin/rspec <new + related specs>`
  - `yarn jest <new + related specs>`
  - `bin/rails db:migrate` + rollback + migrate when a migration was added (note the runtime; see the schema.rb memory)
- **Independent review:** spawn one fresh `general-purpose` agent (default model). Pass it the ticket text, the SPEC list and the branch name, and tell it to review `git diff develop...HEAD` plus the uncommitted diff as a hostile senior reviewer. It should cover:
  1. every SPEC implemented
  2. every entry point from Phase 1 guarded (Rule 7)
  3. correctness and edge cases
  4. authz and tenant/org-unit scoping, IDOR, mass assignment, injection, XSS
  5. N+1 and work repeated per render
  6. dead code, duplication, reuse of existing helpers
  7. missing tests

  Ask for real findings only, each with `file:line`, severity and a fix, and no praise or style nitpicks covered by linters. It must not edit files.

Fix the confirmed findings and re-run only the affected checks. **Output:**
```
Coverage: SPEC-1 ✓ foo.rb:42 · SPEC-2 ✓ Bar.tsx:88 · …
Review: <n> findings fixed — <one line each> (or "no findings")
Checks: rubocop ✓ (3 files) · eslint ✓ · rspec 14 examples, 0 failures · jest 6 passed
Verdict: PASS
```
Any SPEC not covered, or any red check → back to Phase 3.

## Phase 5 — Manual test

Read `phases/5-manual-test.md` and follow it.

## Phase 6 — Ship

Once every check has passed or been skipped, read `phases/6-ship.md` and follow it.

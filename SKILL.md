---
name: ticket-workflow
description: >
  Start this skill whenever the user pastes a Jira ticket, shares a ticket ID/key (e.g. STOR-1234),
  says "work on this ticket", "implement this", "pick up this story", "here are the requirements",
  or provides written requirements for any dev task. Codebase-agnostic — discovers conventions
  from the current repo at runtime.
---

# Ticket Workflow Skill

## Rules (non-negotiable)

1. **Never skip a phase.** Phases are sequential. Do not enter a phase before the previous one is complete and its output is written in full.
2. **Run Phases 1 through 4 without stopping for approval.** Unless the user says otherwise, produce the Phase 1 spec table and the Phase 2 plan, then proceed straight through Phase 3 (implement) and Phase 4 (validate) without pausing to ask "should I continue?". The first stop for the human is Phase 5.
2a. **Phase 5 stops at every check.** The manual test plan is delivered interactively — one check per message, and the next check is not sent until the user reports the result of the current one. Never paste the whole plan at once.
3. **Never claim validation without running commands.** Report the actual output of every linter, type check, and test run. No hypothetical success.
4. **Always reference SPEC-IDs — in Phases 1 to 4 only.** Every plan step, every code change note and every coverage row must cite the SPEC-IDs it satisfies. Phase 5 is the exception: the manual test plan is written for a human and uses the ticket's own issue / AC numbering, never SPEC-IDs or TC-IDs.
5. **Always flag assumptions.** Never silently resolve an ambiguity. State the assumption chosen and the risk if it's wrong. If the ambiguity is a genuine blocker (no reasonable default exists), ask before proceeding; otherwise pick the lowest-risk assumption, note it, and keep going.
6. **Scope discipline.** If a needed change falls outside the plan, note it explicitly in the running summary (what, why, which files) and keep going — don't silently expand scope, but don't stop the run for it either unless it's a genuinely destructive or hard-to-reverse change.

---

## Phase 1 — Analyze the ticket

**Goal:** produce a complete, stable spec before any planning begins.

Steps:
1. Check the current git branch. If it is `develop`/`main` or a branch belonging to a different ticket, create (or switch to an existing) branch named `STOR-<ticket-number>-<short-description>` — no more than 25 characters total — for this ticket before doing anything else.
2. Parse the ticket into: goal, functional requirements, acceptance criteria, implicit expectations, edge cases, explicit non-goals / out of scope.
3. Assign each distinct requirement a stable ID: `SPEC-1`, `SPEC-2`, … Do not renumber or reuse IDs across the conversation.
4. List every ambiguity and unstated assumption separately (see template). State the assumption chosen and the risk if wrong.
5. Explore the repo to map specs to concrete code: read relevant files, grep for related symbols, check existing patterns and prior art. Name the actual files, models, migrations, API types/resolvers/endpoints, services, components, and tests likely involved.
6. Output the completed spec table (template: `templates/spec-table.md`).

**Gate:** Phase 1 is done when the spec table is written out in full. Do not proceed to Phase 2 until then.

### Example output shape (Phase 1)

**Goal:** Allow store managers to archive products.

| SPEC-ID | Requirement | Source | Affected code |
|---------|-------------|--------|---------------|
| SPEC-1 | Store manager can archive a product via a button on the product detail page | Ticket AC #1 | `app/javascript/react/pages/product_detail/`, `ProductsController` |
| SPEC-2 | Archived products are hidden from the storefront by default | Ticket description | `Product` model scope, `ProductsController#index` |
| SPEC-3 | Admin can toggle archived products visible via a filter | Inferred from existing filter pattern | `app/javascript/react/components/filters/` |

**Ambiguities:**

| # | Ambiguity | Assumption chosen | Risk if wrong |
|---|-----------|-------------------|---------------|
| A1 | Does archiving cascade to variants? | No cascade — not mentioned | Medium |

---

## Phase 2 — Plan

**Goal:** produce an ordered, SPEC-mapped implementation plan, then move straight into Phase 3.

Steps:
1. Derive an ordered list of implementation steps from the spec table. Each step must:
   - Reference the SPEC-ID(s) it satisfies.
   - Name the exact files it touches and the type of change (add / modify / delete).
2. Cover explicitly: data/migration impact, backwards compatibility, permissions/authorization impact, performance concerns (N+1 risk, index needs), rollback considerations.
3. Call out risks and anything that could break existing behavior.
4. **Present the plan, then continue directly into Phase 3** unless the user has asked to review plans before implementation on this task.

### Example output shape (Phase 2)

**Step 1** _(SPEC-1, SPEC-2)_ — Add `archived` boolean column to `products` table.
- Files: `db/migrate/YYYYMMDDHHMMSS_add_archived_to_products.rb`
- Migration adds column with default `false`; no data backfill needed.

**Step 2** _(SPEC-2)_ — Add `active` default scope to `Product` model.
- Files: `app/models/product.rb`
- Risk: any code doing `Product.unscoped` will still work; callers relying on default scope will be unaffected.

_(…)_

> Proceeding directly to Phase 3.

---

## Phase 3 — Implement

**Goal:** make direct edits to the codebase, scoped exactly to the approved plan.

Steps:
1. Edit files directly. Never dump implementation code blocks in chat as the deliverable.
2. Follow the repo's existing conventions: naming, file structure, error handling, i18n, testing style. Discover these by reading the surrounding code, not by assuming.
3. Report a running summary after each logical unit of work:
   - Created: `path/to/new_file.rb`
   - Modified: `path/to/existing.tsx` — added ArchiveButton component
   - Deleted: _(if any)_
4. If a needed change is discovered that is outside the plan, state it explicitly in the running summary and keep going, unless it's destructive or hard to reverse — in that case, pause and check with the user.

---

## Phase 4 — Validate

**Goal:** independent verification that the implementation is correct, complete, and safe.

Three mandatory passes — do not merge them:

### Pass 1 — Spec compliance
Output the coverage matrix (template: `templates/coverage-matrix.md`).
- Every SPEC-ID must appear.
- Any row with "No" in "Implemented?" → return to Phase 3. No partial completion accepted.

### Pass 2 — Code review
Review as a hostile senior reviewer, not as the author. Cover:
- Correctness and logic errors
- Edge cases and missing guards
- Error handling
- Dead code, duplication, unnecessary abstraction
- Naming and readability
- N+1 queries and other performance traps
- Missing or inadequate tests
- Breakage of existing behavior

### Pass 3 — Security review
- Authentication and authorization checks present
- Tenant / store / data scoping — no cross-tenant data leaks
- Mass assignment protection
- SQL/GraphQL injection vectors
- Unsafe deserialization
- XSS and unescaped output
- IDOR (insecure direct object references)
- Secrets or credentials in code
- Over-permissive API responses
- Input validation and sanitization
- Rate limiting where relevant

### Automated checks
Detect what the repo provides (linters, formatters, type checks, test suite) by reading `package.json`, `Gemfile`, `Makefile`, CI config, etc. Run every applicable check. Report the actual command and its output. Never summarize as "tests pass" without showing evidence.

**End of Phase 4:** explicit verdict — **PASS** (all three passes clean, all checks green) or **FAIL** (itemized reasons, return to Phase 3).

### Example output shape (Phase 4)

**Coverage matrix:** _(see templates/coverage-matrix.md)_

**Code review findings:**
- `product.rb:55` — `archived` scope missing index on column. Added `add_index` to migration.
- No other issues.

**Security review findings:**
- `ProductsController#archive` — missing `authorize!` call before update. Fixed.

**Automated checks:**
```
$ bin/rspec spec/models/product_spec.rb
...........
11 examples, 0 failures
$ yarn eslint app/javascript/react/pages/product_detail/
No ESLint warnings or errors.
```

**Verdict: PASS**

---

## Phase 5 — Manual test plan (interactive, one check at a time)

**Goal:** walk the human tester (the user) through verification **one check at a time**, never advancing until they confirm the result.

This phase is written for a human, not for the process. Drop the internal bookkeeping — SPEC-IDs, TC-IDs and coverage matrices belong to Phases 1-4 and must not appear here.

**Never dump the whole plan in one message.** Build the full plan internally, then release it check by check.

### Definition of a check

One check = the smallest block of actions that ends in a single observable outcome. Actions with nothing to observe are not a check on their own — fold them into the check whose outcome they set up. A check is usually 1-5 steps and always has exactly one **Expected:** line.

### Protocol

1. **Message 1 — setup only.** Give the login/preconditions block (real emails, roles, record ids, URLs from the environment — never placeholders), any caveat that would make a retest misleading (e.g. "this fix only applies to records created after this build; on an older record set X first"), and a one-line outline naming the issues to be covered *and how many checks each has*. No steps yet. Ask them to confirm setup is ready.
2. **Then one message per check.** Each message contains, and contains only:
   - a heading line: `**Issue <n> — <ticket's wording> · check <i> of <n>**`
   - the numbered steps for that check
   - one bold **Expected:** line
   - a closing prompt to reply pass/fail (with what they saw if it fails)
3. **Always give full, copy-pasteable URLs — never bare paths.** Write `https://dev.example.localhost/reports/33/edit`, not `/reports/33/edit`, and append the login parameter whenever the step involves a different user (e.g. `?mcp_user_email=store@manager.com`). Discover the environment's real base URL from the repo or the user rather than assuming one.
4. **Do not send the next check until the user replies.** No batching, no "and then also try…", no previewing later checks. Their reply is the gate.
5. **On pass:** record it and send the next check.
6. **On fail:** stop advancing. Diagnose, fix, run the automated checks again, then re-issue *that same check* for retest. Only move on once it passes or the user explicitly says to skip it.
7. **Order:** follow the ticket's own structure in the ticket's exact order — numbered issues → `Issue <n>`, numbered acceptance criteria → `AC <n>`. Never invent your own ordering or numbering.
8. **Coverage per issue:** the failure the ticket reported, the happy path, then invalid input, permission/role variations, and empty or boundary states where they apply. Finish with a short set of regression checks for adjacent existing behaviour.
9. **No redundant checks.** Before writing a check, confirm no earlier check already proved it. If the user points out an overlap, drop the duplicate and renumber that issue's checks rather than defending it.
10. **Self-contained checks:** if a check needs setup that another fix provides, write that setup out as ordinary steps rather than pointing at another section.
11. **Final message:** once every check has passed (or been skipped), give a short recap — checks passed, anything skipped, anything fixed mid-run.

### Example output shape (Phase 5)

**Message 1**

```markdown
**Login:** append `?mcp_user_email=<email>` to any URL on `https://dev.example.localhost`.
- Admin: `admin@example.com`
- Store manager: `store@manager.com` (org unit `Frank Canyon`)

**Note:** the archive default only applies to products created after this build.

Coverage: Issue 1 — Archive button (2 checks) · Issue 2 — Storefront visibility (1 check) · Regression (1 check).

Ready when you are — tell me when you're logged in and I'll send the first check.
```

**Message 2 (after they confirm)**

```markdown
**Issue 1 — Archive button missing on product detail · check 1 of 2**

1. As admin, go to `https://dev.example.localhost/products/1`
2. Click **Archive**
3. Confirm the dialog

**Expected:** toast "Product archived"; the button changes to **Unarchive**.

Pass or fail? If it fails, tell me what you saw.
```

**Message 3 (only after their reply)**

```markdown
**Issue 1 — Archive button missing on product detail · check 2 of 2**

1. Click **Unarchive**

**Expected:** toast "Product restored"; the button changes back to **Archive**.

Pass or fail?
```

---

## Phase 6 — Update the PR description

**Goal:** leave the PR describing the shipped work in the repo's required format.

**Gate:** enter this phase only once every Phase 5 check has passed (or been explicitly skipped). Never write the PR description while checks are still outstanding.

Steps:
1. Find the PR for the branch: `gh pr view --json number,title,url`. If there is no PR, do not open one — say so and hand the drafted body to the user to paste.
2. Draft the body using the template (template: `templates/pr-description.md`): the `[JIRA ticket](…)` link with the real ticket key, then `## Motivation and Context` and `## Description of changes`, keeping the repo's HTML comment prompts and writing real prose under each.
3. Group the change bullets by the ticket's own issue/AC numbering, in the ticket's order — the same structure Phase 5 used — and state the root cause of each fix, not just the symptom.
4. Answer the template's prompts explicitly: performance impact (or "No performance impact.") and migrations (what it changes plus the runtime you observed, or "No migration.").
5. Note in one line anything deliberately left out of scope, e.g. a security finding filed for a follow-up ticket.
6. Post it with `gh pr edit <number> --body-file <file>`, then show the exact body that was posted in your reply.

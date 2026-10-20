# Phase 5 — Manual test (one check per message)

This is written for a human tester. Leave out SPEC-IDs, TC-IDs, tables and coverage bookkeeping.

## Before the first message

1. **Build the whole plan internally**, but release it one check at a time.
2. **Verify every step against the code.** Read the page/component for its real labels, defaults and required inputs (e.g. a field that must be filled before a query runs). Inexact steps waste a test round.
3. **Do the data setup yourself.** If a check needs data or config the project console can create or change (feature flags, tenant/account settings, records, form submissions), do it there and say what changed in one line. Leave it in place afterwards; never revert test data unless asked.
4. **Use real data.** Query for real emails, record ids, names and URLs. Never use placeholders.

## What counts as a check

A check is the smallest set of actions that ends in one observable outcome: usually 1–5 steps and exactly one **Expected:** line. Setup actions with nothing to observe belong to the check they lead into.

- **Order:** follow the ticket's own structure and order (`Issue <n>` / `AC <n>`). In Review mode, group by review comment. End with a few regression checks for nearby existing behaviour.
- **Coverage per item:** the reported failure first, then the happy path, then invalid input, other roles, and empty/boundary states where they apply.
- **No redundant checks.** If an earlier check already proved it, drop it. If the user points out an overlap, drop it and renumber.
- **Self-contained.** If a check depends on setup from another fix, spell that setup out as steps.
- **Terse steps for UI the user knows.** For builders and forms, list what to fill (e.g. `Data: Tasks · Count · Creation Date — Layout: Line`) instead of narrating every click.

## URLs and login

- Every URL is full and copy-pasteable, using the dev base URL from CLAUDE.md/memory, never a bare path.
- **The default user is already logged in.** Add the project's user-switch mechanism (e.g. a login query param) only when the check needs a different user.

## Protocol

1. **Message 1:** preconditions (non-default users with their role/scope, what you changed in the data) and any caveat that would make a retest misleading (e.g. "only records created after this build"). Add a one-line outline, `Issue 1 — <title> (2 checks) · Issue 2 — … · Regression (1)`, then **check 1 in the same message**. Never ask whether the user is ready.
2. **Each later message is exactly one check:**
   ```markdown
   **Issue <n> — <ticket wording> · check <i> of <total>**

   1. Go to `https://<dev-host>/exact/path`
   2. Click **Exact Label**

   **Expected:** exact text / UI state.

   Pass or fail?
   ```
3. **Send the next check only after the user replies.** Don't batch or preview later checks. If the user asks for the whole plan in one go this session, do that instead.
4. **On fail:** stop. Diagnose, fix, re-run the affected automated checks (show the result lines), then re-issue **the same check**. Move on only after a pass or an explicit skip.
5. **On a requirement added during testing:** treat it as a new SPEC. Implement it, check it, and add it to the outline.
6. **Final message:** `All checks done: <n> passed, <n> skipped. Fixed mid-run: <…>.` Then continue straight into Phase 6.

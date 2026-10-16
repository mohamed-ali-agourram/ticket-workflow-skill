# Manual Test Plan Template (interactive)

Written for a human tester, delivered **one check at a time**. No SPEC-IDs, no TC-IDs, no tables, no "Actual result" column — that bookkeeping stays in Phases 1-4.

Build the whole plan internally first, then release it check by check. Never paste it all in one message.

## What counts as a check

One check = the smallest block of actions ending in a single observable outcome (usually 1-5 steps, exactly one **Expected:** line). Setup actions with nothing to observe belong to the check they enable, not to a check of their own.

## Ordering

Follow the ticket's own structure, in the ticket's exact order:
- ticket lists numbered issues → `Issue <n> — <ticket wording>`
- ticket lists numbered acceptance criteria → `AC <n> — <ticket wording>`
- single feature → one group per user-facing behaviour

Then a final group of regression checks. Never renumber or reorder the ticket's items.

---

## Message 1 — setup + check 1 (never ask if they're ready)

```markdown
**Login:** append `?mcp_user_email=<email>` to any URL on `https://dev.example.localhost`.
- Admin: `admin@example.com` (peak org unit `Domain`)
- Store manager: `store@manager.com` (org unit `Frank Canyon`)

**Note:** <caveat that would make a retest misleading, e.g. "this fix only applies to records
created after this build; on an older record, set X first">

Coverage: Issue 1 — <title> (N checks) · Issue 2 — <title> (N checks) · Regression (N checks).

<check 1, in the format below>
```

## Every following message — exactly one check

```markdown
**Issue <n> — <ticket wording> · check <i> of <total>**

1. As <role>, go to `https://dev.example.localhost/exact/url`
2. Click **Exact Button Label**
3. Type `exact value` into **Field Name**

**Expected:** exact text / exact UI state.

Pass or fail? If it fails, tell me what you saw.
```

## Final message — recap

```markdown
All checks done: <n> passed, <n> skipped.
Fixed mid-run: <what changed, if anything>.
```

---

## Rules

- One check per message. The user's reply is the gate — never send the next check before it arrives.
- On fail: stop, diagnose, fix, re-run the automated checks, then re-issue **the same check**. Advance only on pass or an explicit skip.
- Real logins, real record ids, real captions from the environment — query the database for them rather than writing placeholders.
- Every step names an exact URL, element label, or typed value. Never "verify it works correctly".
- URLs are always full and copy-pasteable — scheme + host + path, plus the login parameter when the step switches user. Never a bare path like `/reports/33/edit`.
- No redundant checks: if an earlier check already proved it, drop it and renumber.
- Each check stands alone: if it needs setup another fix provides, write that setup out as ordinary steps.
- Across the checks for an issue, cover the reported failure, the happy path, and then invalid input, permission/role variations, and empty or boundary states where they apply.

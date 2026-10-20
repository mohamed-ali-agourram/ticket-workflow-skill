# PR Description

1. Find the PR: `gh pr view --json number,title,url,commits,body`. If there is none, don't open one; hand the body to the user.
2. Re-read the existing body. Correct any statement a later commit made untrue.
3. A PR can hold several tickets. Keep one section per ticket, plus a table of every commit with its ticket. Every ticket commit gets its part. Review-response commits get one line, and their effect is folded into the sections they change.
4. Write to a file in the scratchpad and post it with `gh pr edit <n> --body-file <file>`. If that errors (e.g. the "Projects (classic)" deprecation), use `gh api -X PATCH repos/cowcross/storiq/pulls/<n> -F body=@<file>`. Confirm with `gh pr view <n> --json body`.

## Shape (per ticket)

```markdown
[JIRA ticket](https://storiq.atlassian.net/browse/STOR-<n>)

## Motivation and Context
<!-- Are you introducing a new feature? Provide some context -->
<!-- Why is this change required? What problem does it solve? -->

<what the ticket reported and why it matters: one short paragraph, or one bullet per reported issue>

## Description of changes
<!-- Describe your changes in detail -->
<!-- Any noticeable performance change? -->
<!-- Did you add a migration? How long did take the migration to run? -->

<bullets grouped by the ticket's own Issue/AC order, each giving the root cause and the fix>

<"No performance impact." or the impact>

<"No migration." or what it changes and the runtime you observed>
```

## Rules

- Use the real ticket key in the link, never a placeholder.
- Give the root cause, not the symptom. "Reports were created with the DB column default, which hid them from descendant org units" beats "fixed permissions".
- Optional template sections (Rollbar Item, Request Metrics, Screenshots, Feedback) are **deleted entirely** when there's nothing to put in them. Never write "N/A". Delete the "Checklist before requesting a review" section.
- Don't put out-of-scope notes in the PR. Mention them in chat.

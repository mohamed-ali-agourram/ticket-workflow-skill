# PR Description Template

Post this as the PR description once every Phase 5 check has passed. Keep the structure exactly as below — the JIRA link line first, then both `##` headings, with the repo's HTML comment prompts left in place (they are invisible when rendered) and real prose written under each.

```markdown
[JIRA ticket](https://storiq.atlassian.net/browse/<TICKET-KEY>)

## Motivation and Context
<!-- Are you introducing a new feature? Provide some context -->
<!-- Why is this change required? What problem does it solve? -->

<what the ticket reported and why it mattered — one short paragraph, or one bullet per
reported issue when the ticket lists several>

## Description of changes
<!-- Describe your changes in detail -->
<!-- Any noticeable performance change? -->
<!-- Did you add a migration? How long did take the migration to run? -->

<one bullet per change, grouped by the ticket's own issue numbering, each naming the root
cause and the fix — file paths where useful>

<performance note, or "No performance impact.">

<migration note: what it changes and how long it took to run, or "No migration.">
```

## Rules

- `<TICKET-KEY>` is the real ticket key (e.g. `STOR-10681`) — never a placeholder.
- Group the change bullets by the ticket's issue/AC numbering, in the ticket's order, matching the Phase 5 structure.
- State the root cause, not just the symptom: "reports were created with the DB column default, which hid them from descendant org units" beats "fixed permissions".
- Always answer the migration prompt explicitly, including the runtime you observed.
- Always answer the performance prompt explicitly, even when the answer is "none".
- Mention anything deliberately left out of scope (e.g. a finding filed for a follow-up ticket) in one line at the end.
- Post with `gh pr edit <number> --body-file <file>`, then show the exact posted body in the reply. If the branch has no PR yet, say so and hand the body over for pasting instead of opening a PR uninvited.

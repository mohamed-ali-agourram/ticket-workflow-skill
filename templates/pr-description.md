# PR Description

1. Find the PR: `gh pr view --json number,title,url,commits,body`.
2. **No PR yet → create one** following the PR conventions in memory (title format, base, labels, draft or not). Write the body (below) to a scratchpad file first, then:
   ```bash
   gh pr create --base <base> --title "[<TICKET-KEY>] <imperative summary>" --body-file <file> [--label <label>]
   ```
   Don't set reviewers or assignees unless the conventions say so.
3. **PR exists → update it.** Re-read the existing body and correct any statement a later commit made untrue.
4. A PR can hold several tickets (e.g. an epic's grouping PR). Keep one section per ticket, plus a table of every commit with its ticket. Every ticket commit gets its part. Review-response commits get one line, and their effect is folded into the sections they change.
5. Post it with `gh pr edit <n> --body-file <file>`. If that errors (e.g. the "Projects (classic)" deprecation), use `gh api -X PATCH repos/{owner}/{repo}/pulls/<n> -F body=@<file>`. Confirm with `gh pr view <n> --json body`.

## Body

Use the repo's PR template (e.g. `.github/PULL_REQUEST_TEMPLATE.md`): same headings in the same order, with its HTML comment prompts left in place and answered. If the repo has no template, use this:

```markdown
[Ticket](<tracker URL for TICKET-KEY>)

## Motivation and Context
<what the ticket reported and why it matters: one short paragraph, or one bullet per reported issue>

## Description of changes
<bullets grouped by the ticket's own Issue/AC order, each giving the root cause and the fix>

<"No performance impact." or the impact>

<"No migration." or what it changes and the runtime you observed>
```

## Rules

- Use the real ticket key and link, never a placeholder.
- Give the root cause, not the symptom. "Reports were created with the DB column default, which hid them from descendant org units" beats "fixed permissions".
- Always answer the performance and migration prompts explicitly.
- Optional template sections are **deleted entirely** when there's nothing to put in them. Never write "N/A". Delete checklist sections the template says to delete.
- Don't put out-of-scope notes in the PR. Mention them in chat.

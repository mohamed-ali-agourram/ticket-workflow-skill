# Phase 6 — Ship (automatic once every check passed or was skipped)

Run every step in order without asking. Stop only if a step fails and the fix isn't obvious.

1. **Safety check.** Re-read the ticket and every requirement added during testing. List each one in a single line with its proof (check number or spec). If anything is missing, go back to Phase 3.
2. **Delta review.** Review only what changed since Phase 4 (fixes made during testing). If nothing changed, skip this. If it did, review it hostile-reviewer style and re-run the affected checks, showing the result lines.
3. **Commit.** Stage only this ticket's files. Subject is `[STOR-<n>] <imperative subject>`. The body gives the root cause and the fix. No Claude signature or co-author lines.
   - **New:** one commit for the ticket.
   - **Rework, or a follow-up for the same ticket:** amend/fixup into that ticket's existing commit, even when it's already pushed.
   - **Review:** a **separate** new commit (e.g. `[STOR-<n>] Address review comments`). Never amend it.
   - **Migration:** commit `db/schema.rb` with only the version line bumped (see the schema/Rails-version memory).
4. **Push.** Use `--force-with-lease` when you rewrote history. If an approved PR loses its approval, say whose in one line afterwards.
5. **Monitor CI** in the background (Monitor/Bash, polling with Python since there's no `jq`) until it's green or parked at the QA gate. Never approve any CircleCI approval job. On a failure:
   - **Our code caused it** (lint, spec, snapshot, an advisory from a package we added): amend the relevant commit and force-push.
   - **Upstream advisory unrelated to us:** add it to the ignore list (`.iyarc` / `.bundler-audit.yml`, with a reason comment) in a standalone `[STOR-0000] Fix audit` commit. Never bump or upgrade the package.
   - **A job stuck as `not_running`:** rerun the workflow through the API.

   Then monitor again.
6. **Jira:** move the ticket to **Code review** (transition `Code review`).
7. **PR description:** read `templates/pr-description.md` and follow it.
8. **Report** in a few lines: commit hash(es), CI state, Jira status, PR link, test data left changed, and anything deliberately left out (with the reason).

# Phase 6 — Ship (automatic once every check passed or was skipped)

Run every step in order without asking. Stop only if a step fails and the fix isn't obvious.

1. **Safety check.** Re-read the ticket and every requirement added during testing. List each one in a single line with its proof (check number or spec). If anything is missing, go back to Phase 3.
2. **Delta review.** Review only what changed since Phase 4 (fixes made during testing). If nothing changed, skip this. If it did, review it hostile-reviewer style and re-run the affected checks, showing the result lines.
3. **Commit.** Stage only this ticket's files. Subject is `[STOR-<n>] <imperative subject>`. The body gives the root cause and the fix. **No Claude signature or co-author lines.**
   - **New:** one commit for the ticket.
   - **Rework, or a follow-up for the same ticket:** fold into that ticket's existing commit, even when it's already pushed. If that commit isn't HEAD, use `git commit --fixup <sha>` + `GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash <sha>~1`.
   - **Review:** a **separate** new commit (e.g. `[STOR-<n>] Address review comments`). Never amend it.
   - **Migration:** commit `db/schema.rb` with only the version line bumped (see the schema/Rails-version memory).
4. **Push.** Use `--force-with-lease` when you rewrote history. If an approved PR loses its approval, say whose in one line afterwards.
5. **PR.** Read `templates/pr-description.md` and follow it. It covers creating the PR when there is none and updating it when it already exists.
6. **Monitor CI until the OBT is deployed.** Run a background Monitor (polling with Python since there's no `jq`; the token is in the CircleCI memory) on the branch's latest `Pull Request` workflow.
   - **Tests** (`Check branch is up to date`, Jest/audit, Rspec, Cucumber): poll every ~2 min. When they're all green, **move the Jira ticket to Code review** (transition `Code review`).
   - **`Wait for QA capacity`:** an approval gate that belongs to QA. **Never approve it**, or any other approval job (`Passing QA`, `Failing QA`, `Finished testing`). Keep polling slowly (~10 min) while it's on hold and re-arm the monitor if it expires.
   - **Done** when `Deploy to OBT` succeeds. Report the OBT (its URL if the job or `storiq obt list` shows it) and the Smoke Test result.

   On a failure, fix it and push again, then monitor again:
   - **Our code caused it** (lint, spec, snapshot, an advisory from a package we added): amend the relevant commit and force-push.
   - **`Check branch is up to date` failed:** rebase onto `origin/develop` and force-push. On a shared grouping branch, or a ticket already in the Jira QA column, ask first.
   - **Upstream advisory unrelated to us:** add it to the ignore list (`.iyarc` / `.bundler-audit.yml`, with a reason comment) in a standalone `[STOR-0000] Fix audit` commit. Never bump or upgrade the package.
   - **A job stuck as `not_running`:** rerun the workflow through the API.
7. **Report** in a few lines: commit hash(es), PR link, CI/OBT state, Jira status, test data left changed, and anything deliberately left out (with the reason).

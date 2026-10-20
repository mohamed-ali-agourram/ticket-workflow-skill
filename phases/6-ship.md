# Phase 6 — Ship (automatic once every check passed or was skipped)

Run every step in order without asking. Stop only if a step fails and the fix isn't obvious.

1. **Safety check.** Re-read the ticket and every requirement added during testing. List each one in a single line with its proof (check number or test). If anything is missing, go back to Phase 3.
2. **Delta review.** Review only what changed since Phase 4 (fixes made during testing). If nothing changed, skip this. If it did, review it hostile-reviewer style and re-run the affected checks, showing the result lines.
3. **Commit.** Stage only this ticket's files. The subject uses the project's commit prefix (e.g. `[<TICKET-KEY>]`) and an imperative summary. The body gives the root cause and the fix. **No Claude signature or co-author lines.**
   - **New:** one commit for the ticket.
   - **Rework, or a follow-up for the same ticket:** fold into that ticket's existing commit, even when it's already pushed. If that commit isn't HEAD, use `git commit --fixup <sha>` + `GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash <sha>~1`.
   - **Review:** a **separate** new commit (e.g. `[<TICKET-KEY>] Address review comments`). Never amend it.
   - **Migration:** commit the schema file following any schema-file rule in memory.
4. **Push.** Use `--force-with-lease` when you rewrote history. If an approved PR loses its approval, say whose in one line afterwards.
5. **PR.** Read `templates/pr-description.md` and follow it. It covers creating the PR when there is none and updating it when it already exists.
6. **Monitor CI until the review/QA environment is deployed.** Run a background Monitor on the branch's latest PR pipeline, using the CI access and job names in memory (poll with Python if `jq` is missing).
   - **Test jobs:** poll every ~2 min. When they're all green, **move the ticket to Code review** in the tracker.
   - **Approval gates** (e.g. QA capacity, passing/failing QA): they belong to QA or others. **Never approve one.** Keep polling slowly (~10 min) while the pipeline is on hold, and re-arm the monitor if it expires.
   - **Done** when the deploy-to-review-env job succeeds. Report the environment (with its URL if available) and the smoke test result.

   On a failure, fix it and push again, then monitor again:
   - **Our code caused it** (lint, test, snapshot, an advisory from a package we added): amend the relevant commit and force-push.
   - **Branch-up-to-date check failed:** rebase onto the base branch and force-push. On a shared grouping branch, or a ticket already in QA, ask first.
   - **Upstream advisory unrelated to us:** add it to the audit ignore list (with a reason comment) in a standalone commit using the no-ticket key. Never bump or upgrade the package.
   - **A job stuck and not running:** rerun the workflow through the CI API.
7. **Report** in a few lines: commit hash(es), PR link, CI/environment state, ticket status, test data left changed, and anything deliberately left out (with the reason).

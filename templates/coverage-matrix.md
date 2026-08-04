# Coverage Matrix Template

Used in Phase 4 spec-compliance pass. Every SPEC-ID from the spec table must appear.
Anything in the "Implemented?" column that is not "Yes" sends the agent back to Phase 3.

| SPEC-ID | Implemented? | Where (file:line) | Evidence |
|---------|-------------|-------------------|----------|
| SPEC-1  | Yes         | `app/models/foo.rb:42` | `validates :name, presence: true` added |
| SPEC-2  | No          | —                 | Discovered missing during review — returning to Phase 3 |

**Verdict:** PASS / FAIL (reason)

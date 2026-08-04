# Spec Table Template

Produce one row per distinct requirement. Assign IDs sequentially and never reuse them.

| SPEC-ID | Requirement | Source | Affected code |
|---------|-------------|--------|---------------|
| SPEC-1  | _what must be true_ | ticket line / inferred | `path/to/file.rb`, `path/to/component.tsx` |
| SPEC-2  | … | … | … |

**Ambiguities / Assumptions** (list separately, never silently resolve):

| # | Ambiguity | Assumption chosen | Risk if wrong |
|---|-----------|-------------------|---------------|
| A1 | _e.g. "Should this also apply to archived records?"_ | Excluded — not mentioned in ticket | Low |

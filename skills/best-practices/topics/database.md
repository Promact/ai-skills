# Database Access

- Wrap multi-step writes in a transaction; don't leave data in a
  partially-updated state on failure.
- Index columns used in `WHERE`/`JOIN`/`ORDER BY` on tables large enough to
  matter; watch for N+1 query patterns from ORMs.
- Write migrations that are reversible and safe to run against production
  data (avoid destructive changes without a backfill/rollback plan).
- Keep schema changes backward-compatible with in-flight application code
  during a deploy (expand-then-contract, not drop-then-add).
- Don't store secrets or PII in plaintext columns without a documented
  reason and encryption-at-rest.

<!-- TODO: replace/extend with Promact's internal database standards -->

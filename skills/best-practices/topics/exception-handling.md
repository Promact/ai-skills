# Exception Handling

- Catch only what you can meaningfully handle; let unexpected errors
  propagate rather than swallowing them silently.
- Never catch-and-ignore (empty `catch` blocks) — at minimum log with enough
  context to debug later.
- Fail fast on programmer errors (bad arguments, invariant violations)
  instead of defensively working around them.
- Don't use exceptions for normal control flow.
- Translate internal errors into safe, user-facing messages at the boundary
  — don't leak stack traces or internal details to callers/clients.
- Ensure resources (connections, file handles, transactions) are released on
  the error path, not just the happy path.

<!-- TODO: replace/extend with Promact's internal error-handling standards -->

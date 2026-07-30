# Node.js Best Practices

- Prefer `async`/`await` over raw callbacks/`.then()` chains; always handle
  promise rejections (no floating unhandled promises).
- Validate environment/config at startup and fail fast rather than
  discovering a missing var mid-request.
- Use a proper logging library with levels, not `console.log`, in service
  code.
- Isolate CPU-bound work from the event loop (worker threads or a separate
  service) — don't block it with synchronous heavy computation.
- Prefer ESM for new code; keep module boundaries and circular-dependency-free.

<!-- TODO: replace/extend with Promact's internal Node.js standards -->

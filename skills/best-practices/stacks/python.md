# Python Best Practices

- Use type hints on public functions/methods; run a type checker (mypy/pyright)
  in CI rather than relying on hints as documentation only.
- Prefer `dataclasses`/`pydantic` models over raw dicts for structured data
  crossing a boundary (API, DB, config).
- Use context managers (`with`) for anything acquiring a resource
  (files, connections, locks).
- Avoid mutable default arguments; avoid wildcard imports.
- Use `asyncio` consistently within an async codebase — don't mix blocking
  I/O calls into async request handlers.

<!-- TODO: replace/extend with Promact's internal Python standards -->

# Python Coding Guidelines

- Follow PEP 8: `snake_case` for functions/variables, `PascalCase` for
  classes, `UPPER_SNAKE_CASE` for constants.
- One module per cohesive responsibility; avoid god-modules that accumulate
  unrelated helpers.
- Tests live in `tests/` mirroring the package structure, named
  `test_<module>.py`; test functions named `test_<behavior>`.
- Keep `__init__.py` files minimal — re-export a package's public API there,
  don't hide logic in them.

<!-- TODO: replace/extend with Promact's internal Python style guide -->

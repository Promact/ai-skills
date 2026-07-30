# Testing Structure

- Mirror the source tree in the test tree (or colocate per the stack's own
  convention — see `stacks/*.md`) so a test's location is predictable from
  its subject's location.
- Name tests to describe the behavior under test, not the method name alone
  (`returns 404 when order is missing`, not `test_getOrder2`).
- One logical assertion focus per test; use setup/fixtures instead of
  repeating arrange code across tests.
- Keep unit tests free of real network/database calls; use integration
  tests (clearly named/located as such) for those.

<!-- TODO: replace/extend with Promact's internal testing standards -->

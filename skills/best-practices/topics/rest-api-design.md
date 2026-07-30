# REST API Design

- Use nouns for resource paths and HTTP verbs for actions
  (`POST /orders`, not `POST /createOrder`).
- Return the correct status code for the outcome (`201` on create, `204` on
  empty success, `400` on client error, `404` on missing resource, `422` on
  validation failure) — don't collapse everything to `200`/`500`.
- Version breaking changes explicitly (URL or header versioning) rather than
  changing a shipped contract in place.
- Validate and sanitize all request input at the boundary before it reaches
  business logic.
- Paginate any list endpoint that can return an unbounded result set.
- Keep response shapes consistent across endpoints (envelope, error format,
  casing) within the same service.

<!-- TODO: replace/extend with Promact's internal API standards -->

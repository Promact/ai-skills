# .NET Best Practices

- Prefer `async`/`await` all the way down; avoid blocking on async code
  (`.Result`, `.Wait()`).
- Use dependency injection via the built-in container rather than static
  singletons or `new`-ing up services deep in call chains.
- Use `IHttpClientFactory` rather than instantiating `HttpClient` directly
  per request (socket exhaustion).
- Favor `record` types for immutable DTOs; use nullable reference types
  intentionally rather than suppressing warnings.
- Use EF Core migrations for schema changes; avoid manual schema drift
  between environments.

<!-- TODO: replace/extend with Promact's internal .NET standards -->

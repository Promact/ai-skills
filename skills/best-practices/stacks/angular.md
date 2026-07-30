# Angular Best Practices

- Use the `HttpClient` with typed responses and interceptors for
  cross-cutting concerns (auth headers, error handling) rather than
  repeating logic per call site.
- Unsubscribe from long-lived observables (or use `async` pipe /
  `takeUntilDestroyed`) to avoid memory leaks.
- Prefer reactive forms over template-driven forms for anything beyond a
  trivial form.
- Keep components thin; push business logic into services.
- Use `OnPush` change detection where component inputs are immutable, to
  avoid unnecessary check cycles.

<!-- TODO: replace/extend with Promact's internal Angular standards -->

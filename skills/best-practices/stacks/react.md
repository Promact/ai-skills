# React Best Practices

- Keep server state (API data) separate from local UI state; use a data
  library (React Query/SWR/RTK Query) rather than hand-rolled `useEffect`
  fetching where the project already has one.
- Don't call hooks conditionally; keep dependency arrays accurate rather
  than suppressing the lint rule.
- Lift state up only as far as needed; avoid prop-drilling past 2–3 levels
  by reaching for context or composition instead.
- Memoize (`useMemo`/`useCallback`/`React.memo`) only where profiling shows
  it matters — don't apply it reflexively.
- Handle loading/error/empty states explicitly for any async data.

<!-- TODO: replace/extend with Promact's internal React standards -->

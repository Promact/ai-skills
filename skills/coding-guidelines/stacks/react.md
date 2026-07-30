# React Coding Guidelines

- `PascalCase` for component files and component names
  (`OrderSummary.tsx`), `camelCase` for hooks (`useOrderSummary.ts`) and
  regular functions.
- One component per file; colocate a component's styles/tests/stories with
  it in a folder when it has more than one related file.
- Custom hooks live in a `hooks/` folder (or colocated with their feature)
  and always start with `use`.
- Test files colocated as `Component.test.tsx` next to the component.

<!-- TODO: replace/extend with Promact's internal React style guide -->

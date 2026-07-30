# Angular Coding Guidelines

- `kebab-case` for file names (`order-summary.component.ts`), `PascalCase`
  for class names, suffixed by role (`Component`, `Service`, `Module`,
  `Guard`, `Pipe`).
- Organize by feature module (`orders/orders.module.ts` with its components,
  services, and routing colocated) rather than one flat `components/`
  folder.
- Test files colocated as `<name>.spec.ts` next to the file under test.
- Keep services stateless where possible; hold state in a dedicated
  store/service, not scattered across components.

<!-- TODO: replace/extend with Promact's internal Angular style guide -->

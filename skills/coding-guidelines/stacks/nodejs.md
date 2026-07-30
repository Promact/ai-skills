# Node.js Coding Guidelines

- `camelCase` for variables/functions, `PascalCase` for classes, `kebab-case`
  for filenames.
- Group by feature (`orders/orders.controller.ts`, `orders/orders.service.ts`)
  rather than a global `controllers/`/`services/` split, unless the project
  already uses the latter.
- Colocate unit tests next to source (`orders.service.test.ts`) unless the
  project already has a separate `test/` tree.
- Keep `index.ts`/`index.js` files as thin re-export barrels, not logic
  containers.

<!-- TODO: replace/extend with Promact's internal Node.js style guide -->

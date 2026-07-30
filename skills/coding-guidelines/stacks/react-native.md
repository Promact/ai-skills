# React Native Coding Guidelines

- Same naming conventions as React (`PascalCase` components, `camelCase`
  hooks/functions).
- Organize by feature/screen (`screens/OrderDetails/`) with the screen
  component, its styles, and local hooks colocated.
- Platform-specific files use the `.ios.tsx`/`.android.tsx` suffix
  convention rather than inline `Platform.OS` branching scattered through a
  shared file.
- Keep navigation route names and screen component names aligned so a route
  is easy to trace back to its component.

<!-- TODO: replace/extend with Promact's internal React Native style guide -->

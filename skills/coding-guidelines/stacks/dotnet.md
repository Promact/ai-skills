# .NET Coding Guidelines

- `PascalCase` for classes, methods, properties, and public members;
  `camelCase` for local variables and private fields (commonly prefixed
  `_camelCase`).
- One public type per file, file named after the type.
- Organize by feature folder (e.g. `Orders/`) containing its controller,
  service, and DTOs, unless the project already uses a layered
  (`Controllers/`, `Services/`) layout.
- Test project mirrors the main project's namespace structure, suffixed
  `.Tests`; test files named `<ClassUnderTest>Tests.cs`.

<!-- TODO: replace/extend with Promact's internal .NET style guide -->

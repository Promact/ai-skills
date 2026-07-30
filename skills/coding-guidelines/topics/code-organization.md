# Code Organization

- Group by feature/domain rather than by technical layer where the project
  already does so (don't introduce a `controllers/`, `services/`, `models/`
  split into a codebase organized by feature, or vice versa).
- One primary export/class/component per file, named to match the file.
- Keep a consistent place for shared/cross-cutting code (`shared/`,
  `common/`, `lib/`) and don't duplicate a utility that already exists there.
- Configuration and secrets stay out of source-controlled code files — use
  env vars or a config file that's gitignored where appropriate.

<!-- TODO: replace/extend with Promact's internal organization standards -->

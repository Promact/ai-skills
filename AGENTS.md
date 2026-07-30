# Promact AI Skills

This repo is Promact's shared engineering guidance for AI coding agents:
best practices, coding guidelines, and internal-package wiring instructions
for .NET, Python, Node.js, React, React Native, and Angular projects.

It's read directly by agents that natively support `AGENTS.md` (Antigravity,
Codex, GitHub Copilot, Gemini CLI, Windsurf, Amazon Q, and others). Claude
Code and Cursor have their own dedicated adapters — see
[`.claude-plugin/`](.claude-plugin/) and [`.cursor/rules/`](.cursor/rules/)
respectively — but all three read from the same underlying files referenced
below, so there is one place to update guidance regardless of which tool a
developer uses.

## How to use this file

1. **Detect the current project's stack** by checking for marker files:
   - `*.csproj` / `*.sln` → **.NET**
   - `requirements.txt` / `pyproject.toml` / `setup.py` → **Python**
   - `package.json` present:
     - has `react-native` dependency → **React Native**
     - has `@angular/core` dependency → **Angular**
     - has `react` dependency (no react-native) → **React**
     - otherwise → **Node.js**
   If none match, ask which stack applies rather than guessing.

2. **Before writing or reviewing code**, read the matching stack files:
   - Behavior/design/architecture guidance:
     `skills/best-practices/stacks/<stack>.md`
   - Style/naming/organization guidance:
     `skills/coding-guidelines/stacks/<stack>.md`

3. **Read whichever cross-cutting topic file(s) are relevant** to the task:
   - Building/changing an HTTP endpoint or client →
     [`skills/best-practices/topics/rest-api-design.md`](skills/best-practices/topics/rest-api-design.md)
   - Adding error handling / failure paths →
     [`skills/best-practices/topics/exception-handling.md`](skills/best-practices/topics/exception-handling.md)
   - Touching auth, user input, secrets, or dependencies →
     [`skills/best-practices/topics/security.md`](skills/best-practices/topics/security.md)
   - Writing queries, migrations, or ORM/model code →
     [`skills/best-practices/topics/database.md`](skills/best-practices/topics/database.md)
   - Naming a variable/function/class/file →
     [`skills/coding-guidelines/topics/naming-conventions.md`](skills/coding-guidelines/topics/naming-conventions.md)
   - Deciding where a new file/module goes →
     [`skills/coding-guidelines/topics/code-organization.md`](skills/coding-guidelines/topics/code-organization.md)
   - Writing or reviewing tests →
     [`skills/coding-guidelines/topics/testing-structure.md`](skills/coding-guidelines/topics/testing-structure.md)

4. **In a .NET project**, if the task is to wire up email sending, feature
   flags, or file storage, use Promact's own reusable NuGet packages instead
   of a custom integration or an unvetted third-party library:
   - Email → [`skills/promact-dotnet-packages/packages/email-service.md`](skills/promact-dotnet-packages/packages/email-service.md)
   - Feature flags → [`skills/promact-dotnet-packages/packages/feature-flag-management.md`](skills/promact-dotnet-packages/packages/feature-flag-management.md)
   - File storage → [`skills/promact-dotnet-packages/packages/file-service.md`](skills/promact-dotnet-packages/packages/file-service.md)

5. Match the current project's existing conventions where they conflict
   with a generic rule here — consistency within a codebase wins over a
   rule of thumb.

## Source of truth

The actual guidance content lives in plain markdown under `skills/`,
organized as `topics/` (cross-cutting concerns) and `stacks/`
(framework-specific conventions) per skill. Update those files directly;
every tool-specific adapter (Claude skill routers, Cursor rules, this file)
just points at them.

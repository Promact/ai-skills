---
name: best-practices
description: Use whenever writing, reviewing, or designing code in a Promact project — any .NET, Python, Node.js, React, React Native, or Angular codebase. Covers architecture and design judgment calls — API design, exception handling, security, and database access — not just style. Trigger on requests to add an endpoint, design a data model, handle errors, review a PR for correctness/security, or "is this the right way to do X".
---

# Promact Best Practices

This skill answers "is this the *right* way to build it" — architecture,
error handling, security, data access, and API design. For naming/formatting/
file-structure conventions, see the sibling `coding-guidelines` skill instead.

## How to use this skill

1. **Detect the project's stack** by checking for marker files in the repo
   root (and nearest ancestor if not found there):
   - `*.csproj` / `*.sln` → `.NET`
   - `requirements.txt` / `pyproject.toml` / `setup.py` → `Python`
   - `package.json` present:
     - has `react-native` dependency → `React Native`
     - has `@angular/core` dependency → `Angular`
     - has `react` dependency (no react-native) → `React`
     - otherwise → `Node.js`
   If none match, ask the user which stack applies rather than guessing.

2. **Read the matching stack file** for framework-specific guidance:
   - `stacks/dotnet.md`, `stacks/python.md`, `stacks/nodejs.md`,
     `stacks/react.md`, `stacks/react-native.md`, `stacks/angular.md`

3. **Read whichever topic file(s) are relevant** to the task at hand — don't
   load all of them, just the ones that apply:
   - Building/changing an HTTP endpoint or client → `topics/rest-api-design.md`
   - Adding try/catch, error responses, or failure handling → `topics/exception-handling.md`
   - Touching auth, user input, secrets, or dependencies → `topics/security.md`
   - Writing queries, migrations, or ORM/model code → `topics/database.md`

4. Apply the guidance while writing or reviewing code; call out violations
   you notice in existing code even if it's outside the immediate task.

## Topics
- [REST API design](topics/rest-api-design.md)
- [Exception handling](topics/exception-handling.md)
- [Security](topics/security.md)
- [Database access](topics/database.md)

## Stacks
- [.NET](stacks/dotnet.md)
- [Python](stacks/python.md)
- [Node.js](stacks/nodejs.md)
- [React](stacks/react.md)
- [React Native](stacks/react-native.md)
- [Angular](stacks/angular.md)

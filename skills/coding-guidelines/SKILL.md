---
name: coding-guidelines
description: Use whenever writing or reviewing code in a Promact project — any .NET, Python, Node.js, React, React Native, or Angular codebase. Covers naming, formatting, file/folder organization, and test structure conventions. Trigger on requests to write new code, rename things, organize files, or "does this follow our style".
---

# Promact Coding Guidelines

This skill answers "does this *look* right" — naming, formatting, file/folder
organization, and test structure. For architecture/design/security/error-
handling judgment calls, see the sibling `best-practices` skill instead.

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

2. **Read the matching stack file** for framework-specific conventions:
   - `stacks/dotnet.md`, `stacks/python.md`, `stacks/nodejs.md`,
     `stacks/react.md`, `stacks/react-native.md`, `stacks/angular.md`

3. **Read whichever topic file(s) are relevant**:
   - Naming a variable/function/class/file → `topics/naming-conventions.md`
   - Deciding where a new file/module goes → `topics/code-organization.md`
   - Writing or reviewing tests → `topics/testing-structure.md`

4. Match the existing project's conventions where they conflict with a
   generic rule here — consistency within a codebase wins over a rule of
   thumb.

## Topics
- [Naming conventions](topics/naming-conventions.md)
- [Code organization](topics/code-organization.md)
- [Testing structure](topics/testing-structure.md)

## Stacks
- [.NET](stacks/dotnet.md)
- [Python](stacks/python.md)
- [Node.js](stacks/nodejs.md)
- [React](stacks/react.md)
- [React Native](stacks/react-native.md)
- [Angular](stacks/angular.md)

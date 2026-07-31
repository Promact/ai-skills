# ai-skills

Promact's shared AI coding-agent guidance: best practices, coding
guidelines, and internal-package wiring instructions so any developer's AI
tool automatically applies Promact's standards, whatever stack the project
is in — .NET, Python, Node.js, React, React Native, or Angular.

Supports three tools from one canonical set of content files:

| Tool | Adapter | Mechanism |
|---|---|---|
| **Claude Code** | [`.claude-plugin/`](.claude-plugin/) + [`skills/*/SKILL.md`](skills/) | Installable plugin; skills auto-trigger by description |
| **Cursor** | [`.cursor/rules/*.mdc`](.cursor/rules/) | Glob-triggered (stacks with a reliable file signature) or agent-requested (cross-cutting topics, ambiguous stacks) rules, each pulling in the matching content file via `@`-reference |
| **Antigravity, Codex, Copilot, Gemini CLI, and other [AGENTS.md](https://agents.md)-native tools** | [`AGENTS.md`](AGENTS.md) | Read at session start; routes to the same content files by instruction |

The actual guidance lives once, in plain markdown under `skills/*/topics/`
and `skills/*/stacks/` — every adapter above just points at those files, so
there's one place to update regardless of which tool a developer uses.

Note: unlike Claude Code's plugin marketplace, Cursor and Antigravity have
no built-in org-wide install mechanism — how individual project repos pull
in `AGENTS.md`/`.cursor/rules` from this repo (submodule, copy, or
otherwise) is not yet solved here.

## Claude Code skills

- **best-practices** — architecture and design judgment: REST API design,
  exception handling, security, database access. See
  [`skills/best-practices/SKILL.md`](skills/best-practices/SKILL.md).
- **coding-guidelines** — naming, formatting, file/folder organization, and
  test structure. See
  [`skills/coding-guidelines/SKILL.md`](skills/coding-guidelines/SKILL.md).
- **promact-dotnet-packages** — wires up Promact's internal reusable NuGet
  packages ([Promact/reusable-components](https://github.com/Promact/reusable-components))
  for email sending, feature flags, and file storage in .NET projects
  instead of a custom integration. See
  [`skills/promact-dotnet-packages/SKILL.md`](skills/promact-dotnet-packages/SKILL.md).

The first two skills auto-detect the current project's stack and pull in
the matching guidance from their `stacks/` and `topics/` subfolders.

## Install — manually, or per project

From any Claude Code session:

```
/plugin marketplace add promact/ai-skills
/plugin install promact-ai-skills@promact-ai-skills
```

To make this automatic for every developer working in a given project (but
not elsewhere), add to that project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "promact-ai-skills": {
      "source": { "source": "github", "repo": "promact/ai-skills" }
    }
  },
  "enabledPlugins": {
    "promact-ai-skills@promact-ai-skills": true
  }
}
```

## Install — org-wide (every developer, every project)

Project-level settings above only apply inside repos that commit that
file. For every developer to get this automatically regardless of which
project they're in, an org Owner/Primary Owner configures it once via
**managed settings**, which take precedence over user and project settings
and can't be overridden locally.

**If Promact is on Claude for Teams/Enterprise** (the case today): go to
`claude.ai/admin-settings/claude-code` and paste:

```json
{
  "extraKnownMarketplaces": {
    "promact-ai-skills": {
      "source": { "source": "github", "repo": "Promact/ai-skills" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "promact-ai-skills@promact-ai-skills": true
  }
}
```

Every developer's Claude Code fetches this at next login (and polls
hourly after) — no manual `/plugin install` needed. `autoUpdate: true`
means bumping `version` in `.claude-plugin/plugin.json` and pushing is
enough to roll updates out org-wide.

Note: the admin console validates the settings on save — if it rejects
`enabledPlugins` as an object map, try the array form
(`["promact-ai-skills@promact-ai-skills"]`) instead; both shapes have
turned up in different docs.

**Without a Teams/Enterprise admin console**, the same effect can be
achieved via IT/MDM pushing a `managed-settings.json` file (same JSON
shape as above) to a fixed OS path on every machine:
- macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
- Linux: `/etc/claude-code/managed-settings.json`
- Windows: `C:\Program Files\ClaudeCode\managed-settings.json`

## Contributing

Guideline content lives in `skills/<skill>/topics/*.md`,
`skills/<skill>/stacks/*.md`, and `skills/<skill>/packages/*.md` as plain
markdown — edit those directly. That's enough for `AGENTS.md`-native tools
and Cursor to pick up automatically. For Claude Code, also bump `version`
in `.claude-plugin/plugin.json` and push — developers get the update next
time their marketplace refreshes (`/plugin marketplace update
promact-ai-skills`).

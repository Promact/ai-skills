# ai-skills

Promact's shared Claude Code plugin: org-wide skills so every developer's
Claude Code session automatically applies Promact's best practices and
coding guidelines, whatever stack the project is in — .NET, Python,
Node.js, React, React Native, or Angular.

## Skills

- **best-practices** — architecture and design judgment: REST API design,
  exception handling, security, database access. See
  [`skills/best-practices/SKILL.md`](skills/best-practices/SKILL.md).
- **coding-guidelines** — naming, formatting, file/folder organization, and
  test structure. See
  [`skills/coding-guidelines/SKILL.md`](skills/coding-guidelines/SKILL.md).

Each skill auto-detects the current project's stack and pulls in the
matching guidance from its `stacks/` and `topics/` subfolders.

## Install (org-wide)

From any Claude Code session:

```
/plugin marketplace add promact/ai-skills
/plugin install promact-ai-skills@promact-ai-skills
```

To make this automatic for every developer on a given project, add to that
project's `.claude/settings.json`:

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

## Contributing

Guideline content lives in `skills/<skill>/topics/*.md` and
`skills/<skill>/stacks/*.md` as plain markdown — edit those directly, bump
`version` in `.claude-plugin/plugin.json`, and push. Developers get the
update next time their marketplace refreshes
(`/plugin marketplace update promact-ai-skills`).

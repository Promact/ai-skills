# Proposal: Extending `ai-skills` with Commands and Agents

**Prepared by:** Roshni Shah
**Repo:** `promact/ai-skills`
**Context:** The `ai-skills` plugin (Skills: `best-practices`, `coding-guidelines`,
`promact-dotnet-packages`) is being rolled out org-wide via Claude for
Teams/Enterprise managed settings, so every developer has it installed
by default.

## 1. Why Skills alone aren't enough

Skills are passive: Claude loads a skill's guidance automatically when it
judges the current task relevant, and that guidance only shapes code
**Claude writes from scratch, in the moment**. Two structural gaps remain
once this is used org-wide:

1. **No retroactive check.** Skills don't scan code that already exists —
   legacy files, a human-written PR, code merged from another tool, or
   anything written before the plugin was installed. There's nothing for a
   skill to "trigger" on there.
2. **Triggering is inference, not a guarantee.** A skill fires based on
   Claude's read of task relevance. A vague ask ("fix this bug", "add a
   field to this form") may never surface `security` or
   `naming-conventions`, even though the resulting diff touches them.

Skills make Claude's own fresh output *more likely* to be compliant.
**Commands and agents are how we verify compliance on demand** — for code
Claude didn't just write, or didn't think to check.

## 2. Proposed Commands

Commands are explicit, developer-invoked checklists (`.claude/commands/*.md`).
They don't run arbitrary automation — they're canned prompts that make
Claude apply the existing skills deterministically, on request, rather than
hoping the right skill fires organically.

| Priority | Command | What it does | Why |
|---|---|---|---|
| **1** | `/guideline-check [path]` | Detects the stack (dotnet/react/angular/nodejs/python/react-native) from the diff, loads the matching `best-practices` + `coding-guidelines` stack files, and reports violations in the changed files only. Read-only. | Lets any developer self-check before opening a PR — the single highest-leverage, lowest-friction addition. |
| **2** | `/pre-pr-review` | Broader pass: pulls in `naming-conventions`, `code-organization`, `testing-structure`, `exception-handling`, `security`, and `rest-api-design` as relevant, and gives a go/no-go summary. | Catches cross-cutting issues `/guideline-check` might not, right before a PR is opened. |
| **3** | `/wire-package <email\|feature-flags\|file-storage>` | Scaffolds using the org's sanctioned NuGet packages from `promact-dotnet-packages`, instead of a developer defaulting to SendGrid/AWS SDK/etc. directly. | Prevents a specific, common wrong default rather than relying on a developer remembering the internal package exists. |

**Deprioritized for now:** authoring-side commands (`/new-stack`,
`/new-topic`, `/check-skill-links`, `/sync-agents-md`). These help
whoever maintains the skill content, but don't affect the developers
consuming the plugin — worth revisiting once the commands above are in use
and we're iterating on the guidance itself.

## 3. Proposed Agents

Agents are isolated reviewer personas (`.claude/agents/*.md`) with
restricted tools and no memory of the authoring session — useful
specifically because they review with fresh eyes, unbiased by whatever
assumptions the developer's own session made while writing the code.

| Priority | Agent | Scope | Why an agent, not a command |
|---|---|---|---|
| **1** | `promact-code-reviewer` | Read-only (`Read`/`Grep`/`Bash`, no `Edit`). Loads the relevant stack + topic skill files and reviews a diff/PR for guideline violations. | A second, independent opinion — same reason a human wants a second reviewer, not the author re-reading their own diff. Can be run manually or wired into CI against PRs for actual enforcement, not just self-serve checking. |
| **2** | `dotnet-package-auditor` | Narrower: flags hand-rolled email/feature-flag/file-storage code and points to the matching package in `promact-dotnet-packages`. | This is a checkable yes/no rule ("uses the sanctioned package or not"), distinct enough from general judgment-call review to warrant its own narrow agent rather than folding it into the general reviewer. |

**Deprioritized:** a conversational "onboarding guide" agent. Skills
already answer "why do we do X" on demand when relevant — a dedicated
agent for that would be redundant.

## 4. MCP servers — revisited: org-level troubleshooting skill

Update: a concrete ask has emerged. An org-level skill to troubleshoot
**security, performance, and infrastructure** issues needs live access to
real systems (logs, metrics, scan results) — static guidance alone can't
do this, so MCP is now warranted (superseding the "not recommended"
verdict originally in this section).

**Current stack (per Roshni):** AWS/Azure native monitoring (CloudWatch,
Azure Monitor) + Sentry.

**Known MCP coverage today:**

| Area | Tool | MCP server available? |
|---|---|---|
| Errors / perf | Sentry | Yes — official Sentry MCP server (already connected in this session) |
| Infra / logs (AWS) | CloudWatch | Yes — AWS Labs publish official MCP servers, incl. one for CloudWatch |
| Infra / logs (Azure) | Azure Monitor | Yes — Microsoft's Azure MCP server covers Monitor + most Azure services |
| Security | — | No sanctioned tool identified yet — needs a decision before an MCP server can be picked or built |

**Approach:**

- Reuse the official Sentry, AWS, and Azure MCP servers — no custom build
  needed for those three.
- Decide on a sanctioned security scanning/reporting tool first; only
  then evaluate whether an existing MCP server covers it or one must be
  built in-house.
- Scope access per server to read-only (logs/metrics/scan results) — no
  write access into cloud accounts or Sentry from this skill.
- Pair each MCP server with a troubleshooting skill (runbook per area:
  security / speed / infra) that tells Claude how to use the data, not
  just that it can fetch it.
- Pilot with Sentry + one cloud provider first (narrowest access,
  clearest existing MCP support) before adding security tooling.
- Requires sign-off from whoever owns infra/security access before any
  server is wired into a real environment, since this expands what
  Claude can read org-wide.

## 5. Rollout principle

Because this plugin is pushed via managed settings, developers can't opt
out of it. Everything we add should be:

- **Read-only / report-only by default** — flag issues, don't silently
  rewrite code. Enforcement developers can't see or dispute erodes trust
  fast.
- **Opt-in to invoke** (commands are typed deliberately; agents are spawned
  deliberately or via CI) — nothing runs against a developer's code without
  them asking, at least in this first phase.

## 6. Recommended first step

Ship `/guideline-check` and `promact-code-reviewer` first — one gives
developers a self-serve check, the other gives the org a real
enforcement mechanism if wired into CI. Everything else in this document
can follow once we see how those two land.

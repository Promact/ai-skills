# Proposal: `infra-troubleshoot` Skill Backed by MCP Servers

**Prepared by:** Roshni Shah
**Repo:** `promact/ai-skills`
**Context:** Follow-up to §4 ("MCP servers — revisited") of
`commands-and-agents-proposal.md`. That section identified a concrete need
for an org-level troubleshooting skill with live access to real systems.
This document inventories the MCP servers available for that skill and
proposes how the skill should route a symptom to the right server(s).

## 1. Goal

A developer or support engineer says something like *"the system is
running slow"* or *"customers are getting errors"*, and Claude — via a
skill — knows which third-party services to query (logs, metrics, error
tracker, database) to diagnose the likely cause, without the developer
having to know which tool to check first.

**Scope:** org-wide/generic. Different client projects use different
combinations of these tools, so the skill should detect or ask which
stack a project uses, then route to the matching MCP server(s) rather
than assuming one fixed stack.

**Tools in scope (per known usage across Promact projects):** AWS
(CloudWatch, RDS), Azure (Monitor, Azure Database for PostgreSQL),
Sentry, Cloudflare, PostgreSQL (generic), MongoDB.

## 2. MCP server inventory

| Area | Server | Maintainer | Read/Write | Install |
|---|---|---|---|---|
| AWS CloudWatch | CloudWatch MCP Server | AWS Labs (`awslabs/mcp`) | Read-only | pip/uvx, needs AWS creds |
| AWS APM | CloudWatch Application Signals MCP Server | AWS Labs | Read-only | Same monorepo |
| AWS RDS/Aurora Postgres | Postgres MCP Server | AWS Labs (`awslabs/mcp`) | Query execution via RDS Data API — not confirmed strictly read-only, scope IAM carefully | Docker image |
| Azure (unified) | Azure MCP Server | Microsoft (`Azure/azure-mcp`) | Mixed — mostly read/query, some management (e.g. workbooks) | npm/`npx` |
| — Azure Monitor | tool group within Azure MCP Server | Microsoft | Read + some management | same |
| — Azure DB for PostgreSQL | tool group within Azure MCP Server (**Preview**) | Microsoft | Read-only queries | same |
| Sentry | `sentry-mcp` | Sentry (official) | Both — has write scopes (`event:write`, `project:write`) | Remote OAuth (`mcp.sentry.dev`) or `npx @sentry/mcp-server` |
| Cloudflare | Cloudflare MCP suite (Observability, DNS Analytics, Logpush, Audit Logs, GraphQL Analytics) | Cloudflare (official) | Read-only for the troubleshooting-relevant servers listed; other bundled servers (Workers, main API) are read/write | Remote, OAuth via `mcp.cloudflare.com` |
| PostgreSQL (generic) | Postgres MCP Pro | Crystal DBA | Explicit **Restricted Mode** (read-only, blocks COMMIT/ROLLBACK tricks) | pipx/uv/Docker |
| MongoDB | `mongodb-mcp-server` | MongoDB (official) | Defaults to read-only via `--readOnly` flag | `npx mongodb-mcp-server --readOnly` |

**Conclusion: no in-house MCP server needs to be built for this scope.**
Every tool named has an existing official or well-maintained server.

### Gaps flagged (informational, not blocking)

- **Cloudflare WAF-specific logs** — no dedicated WAF server found; likely
  reachable via Observability/GraphQL/main API instead of a purpose-built
  tool.
- **AWS RDS instance-level management** (start/stop/reboot/modify, not
  query) — not confirmed as an official AWS Labs server; only the
  Postgres *query* server was verified.
- **Security scanning tool** — still an open decision from
  `commands-and-agents-proposal.md` §4; no MCP server can be picked until
  a sanctioned tool is chosen. Out of scope for this document.

## 3. Symptom → server routing

The skill's job is to map a reported symptom to the servers worth
querying, roughly in the order a human would check them. This is a
starting draft — refine as real incidents validate or contradict it.

| Symptom | Check first | Then check | Notes |
|---|---|---|---|
| System running slow | CloudWatch/Azure Monitor metrics (CPU, memory, latency) | Postgres Pro `analyze_db_health` / `get_top_queries`, or Mongo `db-stats` + slow query log; Sentry performance/traces | If cloud metrics are flat but DB is slow, it's almost always a query/index/connection-pool issue, not infra capacity |
| High error rate / 5xx spike | Sentry issues (correlate with deploy time) | CloudWatch/Azure Monitor logs around the same window | Check for a recent deploy first — cheapest signal |
| Database connection pool exhausted | Postgres Pro `analyze_db_health` (connection utilization) or Mongo `db-stats` | CloudWatch RDS connection metrics / Azure DB metrics | App-level pool config vs. DB-level max_connections — check both |
| Outage / service down | CloudWatch/Azure Monitor alarms + resource health | Cloudflare Observability (edge reachability) | Distinguish "app is down" from "can't reach app" (DNS/CDN/edge) |
| Memory leak / OOM kills | CloudWatch/Azure Monitor memory metrics over time (trend, not snapshot) | Sentry (crash reports if app-level) | Needs a time-series view, not a point-in-time check |
| Slow/timeout-ing queries | Postgres Pro `explain_query` / `get_top_queries`; Mongo `explain` | — | Direct DB-level diagnosis, usually doesn't need cloud metrics |
| Deployment-related regression | Sentry (error rate before/after deploy timestamp) | CloudWatch/Azure Monitor logs at deploy time | Time-boxing around the deploy is the key move |
| CDN/edge issues | Cloudflare Observability + DNS Analytics | Cloudflare GraphQL Analytics (cache hit rate) | Check cache hit rate and DNS resolution before blaming origin |
| Cost/usage anomaly | CloudWatch/Azure Monitor (resource usage spike) | Correlate with deploy or traffic spike | Sometimes the "incident" is a runaway process burning cost, not user-facing yet |
| Third-party API degradation | Sentry (outbound call errors/timeouts) | CloudWatch/Azure Monitor (if calls are logged) | Confirm it's *their* latency, not misattributed local slowness |

## 4. Access & safety posture

Consistent with the rollout principle in `commands-and-agents-proposal.md`
§5 (read-only/report-only by default, opt-in to invoke):

- Use **restricted/read-only modes** wherever the server offers one:
  Postgres Pro `--access-mode=restricted`, Mongo `--readOnly`, the
  read-only Cloudflare tool groups (Observability, DNS Analytics,
  Logpush, Audit Logs, GraphQL).
- **Sentry and parts of Azure/Cloudflare can write** (event/project
  write scopes, workbook management, main Cloudflare API). These need
  scope restriction at config/token level, not code — issue read-only
  tokens/scopes for this skill's use.
- **AWS RDS/Aurora Postgres MCP server** executes queries via the RDS
  Data API — scope the IAM role to a read-only DB user before wiring it
  in.
- Requires sign-off from whoever owns infra/security access before any
  server is connected to a real environment (per the existing proposal).

## 5. Open questions

1. Which specific client projects/environments should pilot this first?
   (Narrows which of the 6 tool categories actually need wiring up now
   vs. later.)
2. Do we need per-project config (which stack a given project uses) or
   should the skill ask the developer at invocation time?
3. Confirm acceptable credential storage/handling for each service
   (Sentry tokens, AWS IAM roles, DB read-only users, etc.) — likely
   needs infra/security sign-off per §4 above.

## 6. Recommended first step

Pilot with **CloudWatch or Azure Monitor + Sentry + Postgres Pro**
(narrowest, all read-only-capable, and covers the most common "slow
system" / "high error rate" symptoms) before wiring in Cloudflare and
MongoDB.

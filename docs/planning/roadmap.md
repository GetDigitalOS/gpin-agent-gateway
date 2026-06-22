# Next Session Handoff

**Last updated:** 2026-06-08 (stub — no sessions recorded yet)
**Update cadence:** At the end of every session that lands material work — on `main`, or on the active feature branch while work-in-flight hasn't merged yet.

This doc is a paste-ready prompt for starting the next Claude Code session, and a human-readable status of what's open. `CLAUDE.md` directs every new session here as the first read.

---

## How to start the next session

Paste the block below into a fresh Claude Code session at `C:\dev\business\getdigital\products\gpin-agent-gateway`.

```
I'm picking up GPIN Agent Gateway. No prior session recorded — start fresh:
1. Read `CLAUDE.md` (project conventions and two-layer rule if applicable)
2. Run `git log --oneline -10` (recent commits)
3. Summarize current state and propose where to start.
```

## Repo state

No sessions recorded yet. Run `git log --oneline -10` to see current state.

### Status notes (not counted toward the bar)

- Project is in **scaffold phase**: no `src/` directory, `wrangler.toml` is a 0-byte file, `package.json` has no build/test/dev scripts.
- Compliance audit (2026-05-23, UWP v4.03.01): 16% of applicable principles met or partial (11/69 applicable); 4 Met, 7 Partial, 58 Not Met, 18 N/A.
- Tier 3 — Application. Backend-only Cloudflare Worker (no UI surface).

## Roadmap

Derived from `docs/architecture/COMPLIANCE_REPORT.md` prioritized recommendations. All items are not-started (scaffold phase, no application code yet).

### Critical (Security / Deploy / Compliance Risk)

- [ ] Reconcile or delete the empty `wrangler.toml` (restore from `config/wrangler.toml.backup` with secrets moved to `wrangler secret put`, or document scaffold-only/no-live-deploy state)
- [ ] Move all API secrets (`OPENAI_API_KEY`, `NOTION_SECRET`, `NOTION_DB_ID`, `PINATA_JWT`) from `[vars]` to `wrangler secret put`; document rotation pattern
- [ ] Fix the registry stack profile (`auth: none`, `database: cloudflare-d1`, `seo: n/a`) via a project-hub PR
- [ ] Implement inbound authentication (Bearer token or HMAC-signed request) before any code that calls a paid API; write the ADR first
- [ ] Add `npm audit` + dependency check + a real `build` script (`tsc --noEmit` + esbuild bundle) to CI

### Important (Quality / Reliability Risk)

- [ ] Write the Tier 3 documentation backlog (`api-spec.md`, `data-model.md`, `system-architecture.md`, `auth-architecture.md`, `security-architecture.md`, `error-handling.md`, `testing-plan.md`, `deployment.md`, `monitoring.md`)
- [ ] Capture the deferred ADRs (Tier classification, CF Workers vs Railway, D1+KV vs Postgres+Redis, approval model, inbound auth scheme, prompt management, SEO-n/a — 8 to 12 minimum)
- [ ] Add rate limiting (per-caller KV token bucket) + idempotency (Idempotency-Key header) on side-effecting routes
- [ ] Wire `@getdigitalos/observability` (`@sentry/cloudflare`) with `SENTRY_DSN` secret and `SENTRY_AUTH_TOKEN` in CI for source maps
- [ ] Implement the audit log (D1 schema + migration + write path on every action)

### Recommended (Best Practice)

- [ ] Add preview deployments via `wrangler --env preview` on PRs
- [ ] Add Dependabot config (weekly; npm + github-actions ecosystems)
- [ ] Add PR template + CODEOWNERS for the gateway repo
- [ ] Update CLAUDE.md (fix mojibake, fill the empty Overview section)
- [ ] Add AI cost controls (per-request `max_tokens`, per-caller rate cap, spend alert, KV-backed response cache)
- [ ] Add a `GET /health` endpoint (JSON liveness + downstream-service ping status)

## Open follow-ups

None recorded yet.

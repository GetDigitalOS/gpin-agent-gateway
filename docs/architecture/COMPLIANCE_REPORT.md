# Compliance Report — GPIN Agent Gateway

**Date:** 2026-06-25
**Tier:** 3 — Application
**Framework Version:** Universal Web Development Principles v4.06.00 (canonical, dated 2026-06-21)
**Auditor:** Claude Code (automated audit)
**Previous Audit:** 2026-05-23 (v4.03.01) — re-run against current v4.06.00 canonical

> **Note on framework version:** The `/compliance` skill template still references "UWP v2," but the single canonical document at `C:/dev/project-hub/canonical/references/Universal_Web_Development_Principles.md` is now **v4.06.00**. This audit runs against the current canonical, as the prior re-audit did. The deltas from v4.03.01 → v4.06.00 are additive/patch only (External Spec Mirrors pattern, the additive-tier-law note, the `NEW_PROJECT_CREATION.md` pointer, and a compliance-score-interpretation note) — none change the Tier 3 principle set, so the substantive findings below are unchanged from the prior audit.

---

## Summary

| Status | Count |
|--------|:-----:|
| ✅ Met | 4 |
| ⚠️ Partial | 7 |
| ❌ Not Met | 58 |
| ⬜ Not Applicable | 18 |

**Overall Compliance:** 16% of applicable principles met or partially met (11/69 applicable)

> **Context:** This project remains in **scaffold phase** at the time of audit. There is still no `src/` directory, `wrangler.toml` is still a 0-byte file, and `package.json` still has no build/test/dev scripts beyond the failing `test` placeholder. The active deploy config that would back a `wrangler deploy` is empty; the only infrastructure intent lives in `config/wrangler.toml.backup`. Most ❌ findings reflect the absence of any application code, **not** flawed design — per UWP v4.04.01, compliance % is a *launch-readiness* signal, not a development-progress metric; a pre-feature-build project is expected to score low and this report is its build punch list. That said, the scaffold itself has concrete, fixable gaps that must close before any code ships, and the registry profile is still misaligned with reality (claims Better Auth + Railway Postgres; the project uses Cloudflare D1 + KV with no auth wired).
>
> **What changed since 2026-05-23:** Nothing in code — no `src/`, `wrangler.toml` still empty, `package.json` unchanged, `config/wrangler.toml.backup` unchanged. Documentation moved slightly: `docs/planning/roadmap.md` now exists (a 5th permanent doc), CLAUDE.md gained the hub-owned Session Continuity block and had its mojibake fixed, and `.github/workflows/agent-spawn.yml` references a `docs/ADR-004-agent-spawn-trigger.md` that does **not** exist (new dangling-reference finding, see Structural Integrity). `npm audit` re-run today: still **0 vulnerabilities across 29 deps**.

---

## Critical Gaps (Fix Before Launch)

Security risks, deploy-blocking issues, or compliance violations that block production readiness — in rough fix order:

1. **No source code exists.** `src/index.ts` is absent; the worker cannot deploy in any meaningful sense. Every Tier 3 principle dependent on application behavior (auth, validation, audit logging, error handling, rate limiting, prompt injection defense, structured logging) is unimplemented by absence.

2. **`wrangler.toml` is empty (0 bytes).** The active deploy config is blank. All KV/D1 bindings, env vars, and observability config defined in `config/wrangler.toml.backup` are NOT active. `wrangler deploy` against this repo deploys nothing — or fails. Either restore `wrangler.toml` from the backup (sanitized) or delete the misleading empty file and document the deploy state honestly.

3. **API secrets stored as `[vars]` (plaintext) in backup config.** `OPENAI_API_KEY`, `NOTION_SECRET`, `NOTION_DB_ID`, `PINATA_JWT` are declared in the `[vars]` block of `config/wrangler.toml.backup`. Cloudflare `[vars]` are visible in the dashboard and surface in `wrangler` output as plaintext. These MUST move to `wrangler secret put`. Current values are placeholders (`"sk-..."`, `"secret_..."`), but the *pattern* leaks the first real key dropped in.

4. **No inbound authentication or API-key validation.** No implementation authenticates callers to the gateway. Any actor who reaches the worker URL could trigger agent actions that spend OpenAI tokens, write to Notion, pin to IPFS, or query ENS. For a gateway whose entire job is governance, this is the central security gap. A Bearer token (or HMAC-signed request) check at every route is the minimum.

5. **No input validation.** No Zod or equivalent schema validation exists. Event-queue payloads, webhook bodies, and action requests are all unvalidated. Combined with #4, this is the prompt-injection attack surface for the OpenAI integration — the escalation trigger that forced Tier 3.

6. **No audit logging implementation.** The `AUDIT_D1` binding is declared as infrastructure intent in the backup config, but no schema, migrations, write logic, or read paths exist. "Full audit trail" is a CLAUDE.md claim, not a deliverable. For a gateway that governs autonomous agent actions, the audit trail *is* the product.

7. **Registry stack profile is wrong.** `registry/projects.json` (lines 1406–1416) declares `auth: "better-auth"`, `database: "railway-postgres"`, `seo: "required"`. The actual design (per CLAUDE.md, classification doc, and `config/wrangler.toml.backup`) uses Cloudflare D1 + KV with no Better Auth, no Postgres, and no public web surface for SEO. `/scaffold` and `/compliance` read this profile to decide what to enforce — a wrong profile silently disables relevant checks. Reconcile via a hub PR (see Hub Registration).

8. **No test suite.** `package.json` test script is `echo "Error: no test specified" && exit 1`. Zero unit, integration, or contract tests on a Tier 3 system that proxies four external paid APIs. CI runs no tests before deploying.

9. **No rate limiting.** No inbound request rate limiting on a worker that calls OpenAI and other usage-billed APIs. A loose-loop caller could rack up OpenAI cost in minutes.

10. **CI is non-functional.** `deploy.yml` runs `npm ci`, `npm run build`, `npx wrangler deploy` — but `package.json` has **no `build` script** (CI fails on `npm run build` today), there is no `npm audit`, no SAST, no dependency scanning, and the wrangler deploy targets an empty config (#2). The deploy pipeline is a placeholder, not a pipeline.

11. **AI/LLM concerns are entirely undocumented.** Tier 3 escalation was triggered by the OpenAI integration. UWP requires prompt injection defense, output guardrails, content filtering, fallback behavior, cost controls, prompt versioning, an evaluation framework, transparency disclosure, DPAs with AI providers, and caching. None documented, none implemented.

12. **Documentation stack is ~5% complete for Tier 3.** UWP requires 30–36 docs at Tier 3 (permanent + conditional + ADRs). Present: `CLAUDE.md`, `PROJECT_CLASSIFICATION.md`, `ADR-000-template.md` (a template, not a real ADR), `roadmap.md`, and this report. Missing: `site-spec.md`/`api-spec.md`, `data-model.md`, `system-architecture.md`, `integration-specs.md`, `auth-architecture.md`, `launch-checklist.md`, `testing-plan.md`, `coding-standards.md`, `performance-budget.md`, `security-architecture.md`, `error-handling.md`, `deployment.md`, `monitoring.md`, plus 8–12 ADRs. Most are not "n/a" — they're missing.

---

## Partial Items

1. **HTTPS Everywhere** — Cloudflare Workers enforces TLS at the edge by default; `BASE_URL` in the backup config is `https://`. Partial only because no `src/` exists to confirm no mixed-content/insecure-redirect bugs land later.

2. **Security Headers** — Cloudflare auto-attaches `Strict-Transport-Security` at the edge, but the application has no code to set `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, or `Permissions-Policy`. For a JSON-only API gateway with no HTML output the CSP/X-Frame surface is reduced — but `X-Content-Type-Options: nosniff` and `Referrer-Policy: no-referrer` should still be set on every response once code exists.

3. **Observability Foundation** — `config/wrangler.toml.backup` declares `[observability.logs]` with `enabled = true`, `head_sampling_rate = 1`, `invocation_logs = true`, `persist = true` — a credible scaffold for Cloudflare Workers logs. But no structured (JSON) logging, no correlation IDs, no `@sentry/cloudflare` SDK, no Better Stack uptime monitor. The portfolio-default `@getdigitalos/observability` is not installed.

4. **Secrets Management** — `.gitignore` correctly excludes `.env`, `.dev.vars`, and `config/.dev.vars.backup` (duplicated entries are harmless). However, `config/wrangler.toml.backup` is checked in and declares secrets in `[vars]` (wrong pattern; Critical #3). Net: gitignore hygiene is okay, secret *configuration* pattern is wrong.

5. **Automated Deployment** — `.github/workflows/deploy.yml` triggers on push to `main`, sets up Node 20, runs `npm ci` → `npm run build` → `npx wrangler deploy` with `CLOUDFLARE_API_TOKEN`. **But** there is no `build` script (CI fails), no preview/staging environment, and the deploy targets an empty `wrangler.toml`. Pipeline exists in name only.

6. **Version Control / Git Conventions** — `.gitignore` present and reasonable. CLAUDE.md documents git remote, author email, and branch. Commit history is sensible (conventional-ish). No branch protection rule, no PR template, no CODEOWNERS.

7. **ADR Process** — `ADR-000-template.md` is present and structurally correct. **But** zero actual ADRs exist (Tier 3 requires 8–12), and `agent-spawn.yml` even references a `docs/ADR-004-agent-spawn-trigger.md` that was never written. Decisions already made implicitly (D1 vs KV vs Postgres, approval-mode design, Workers vs Railway, no-auth-yet stance) need capturing.

---

## Detailed Results

### Tier 1: Security Fundamentals

| | Principle | Status | Evidence |
|---|---|---|---|
| ⚠️ | HTTPS Everywhere | Partial | CF Workers enforces TLS; no code yet to verify no mixed content. |
| ⚠️ | Security Headers | Partial | CF edge sets HSTS; no application code sets CSP/X-Frame-Options/X-Content-Type-Options/Referrer-Policy. |
| ❌ | Input Validation | Not Met | No `src/`; no validation logic anywhere. |
| ✅ | Dependency Hygiene | Met | `npm audit` returned 0 vulnerabilities across 29 deps (re-run 2026-06-25). Lockfile committed. Devdeps only (esbuild, typescript, workers-types). |

### Tier 1: Design System Basics

⬜ **Not Applicable** — Backend-only gateway with no UI. All design-token, typography, icon, spacing, breakpoint, and motion principles are n/a. Should be documented explicitly in `component-map.md` ("no UI").

### Tier 1: Performance

| | Principle | Status | Evidence |
|---|---|---|---|
| ⬜ | Core Web Vitals | N/A | No browser surface. |
| ⬜ | Asset Optimization | N/A | No assets. |
| ⬜ | Lazy Loading | N/A | No assets. |
| ⚠️ | CDN Delivery | Partial | Cloudflare Workers run at the edge by definition; no application code yet to confirm no upstream-only patterns. |

### Tier 1: Accessibility

⬜ **Not Applicable** — No UI. Document explicitly ("no rendered surface; consumers are programmatic").

### Tier 1: SEO Fundamentals + SEO Operations

⬜ **Not Applicable** for the gateway itself (no public-facing pages). The registry profile has `seo: required`, which is **wrong** for a backend-only worker. Change to `seo: n/a` and note in an ADR.

### Tier 1: Observability & Diagnostics

| | Principle | Status | Evidence |
|---|---|---|---|
| ⬜ | Plausible Analytics | N/A | No public web surface. |
| ❌ | Pre-Launch DNS/Email Diagnostic Gate | Not Met | No custom domain; deploy targets `*.workers.dev`. If a custom domain is attached, MXToolbox + Workspace Toolbox checks become mandatory. |

### Tier 1: Code Quality

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Separation of Concerns | Not Met | No code. |
| ❌ | DRY Principle | Not Met | No code. |
| ❌ | Semantic Naming | Not Met | No code. |
| ⬜ | Mobile-First CSS | N/A | No CSS. |

### Tier 1: DevOps Basics

| | Principle | Status | Evidence |
|---|---|---|---|
| ⚠️ | Automated Deployment | Partial | `deploy.yml` exists but fails on missing `npm run build`; deploys against empty `wrangler.toml`. |
| ❌ | Preview Deployments | Not Met | No preview/staging flow. CF Workers supports `--env preview`; not used. |
| ❌ | Environment Separation | Not Met | No staging vs prod separation. Single deploy target. |
| ⚠️ | Domain & DNS Management | Partial | Only `*.workers.dev`; no custom domain, no DNS docs. Acceptable for current phase but undocumented. |
| ⚠️ | Version Control | Partial | Git + .gitignore present; no branch protection or PR template. |

### Tier 1: UX Fundamentals

⬜ **Not Applicable** — No UI.

### Tier 2: Enhanced Security

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | CSRF Protection | Not Met | API-only worker; less relevant under bearer tokens — but no auth scheme exists, so this still fails. Address via inbound auth (Critical #4). |
| ❌ | Rate Limiting | Not Met | None. Critical for a gateway calling paid APIs. Use Cloudflare rate limiting or a KV-backed token bucket. |
| ⬜ | Honeypot Fields | N/A | API only. |
| ❌ | Output Encoding | Not Met | No code. Once code exists: JSON-encode all D1/KV reads echoed to callers; never interpolate into HTML/log strings. |
| ⬜ | Content Security Policy | N/A | No HTML responses (JSON API). |
| ❌ | Dependency Scanning in CI | Not Met | No `npm audit` step in `deploy.yml`. No Dependabot config. Add both. |

### Tier 2: Data Handling

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Client+Server Validation | Not Met | No validation anywhere. |
| ⚠️ | Data Minimization | Partial | By scope the gateway should store only action metadata — but no data model exists to verify. Write `data-model.md`. |
| ✅ | Secure Transmission | Met | HTTPS-only via CF Workers; design uses JSON bodies, no sensitive fields in URLs. |
| ❌ | Privacy Compliance | Not Met | No privacy policy, cookie disclosure, or data-handling docs. The OpenAI/Notion/Pinata data flow needs a DPA paragraph somewhere. |

### Tier 2: Design System Extension

⬜ **Not Applicable** (no UI).

### Tier 2: Frontend Architecture Basics

⬜ **Not Applicable** (no frontend).

### Tier 2: Architecture

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Separation of Concerns | Not Met | No code. |
| ❌ | Configuration Externalized | Not Met | Intent (env vars via wrangler) is in backup config but no live config exists. |
| ❌ | Error Handling Strategy | Not Met | No code. Tier 3 requires `error-handling.md` (taxonomy + response shape). |
| ❌ | State Management | Not Met | Worker is stateless by design (state in D1/KV) — needs explicit documentation. |

### Tier 2: Error & Edge Case UX

⬜ **Not Applicable** for empty-state/timeout/form-error UX (no UI). API-side equivalents (4xx/5xx, retry semantics, idempotency) belong under Tier 3 Resilience — all ❌.

### Tier 2: Performance Enhancement

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Code Splitting | Not Met | No code. |
| ❌ | Debouncing/Throttling | Not Met | Inbound rate limiting absent (also Tier 2 Enhanced Security). |
| ❌ | Request Cancellation | Not Met | No code. |
| ❌ | Caching Strategy | Not Met | No code. AI request caching absent. |

### Tier 2: Testing

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Form Validation Testing | Not Met | No tests. |
| ❌ | Calculator Logic Testing | Not Met | No tests; no domain logic yet. |
| ⬜ | Cross-Browser Testing | N/A | No browser. |
| ⬜ | Accessibility Testing | N/A | No UI. |
| ⬜ | Visual Regression Testing | N/A | No UI. |

### Tier 2: Observability

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Client-Side Error Capture | Not Met | No client; worker-side equivalent (Sentry via `@sentry/cloudflare`) also absent. |
| ❌ | Source Maps in Production | Not Met | No Sentry config, no `SENTRY_AUTH_TOKEN` in `deploy.yml`. |
| ⬜ | Basic Analytics | N/A | No web surface to track. |
| ❌ | Uptime Monitoring | Not Met | No Better Stack monitor; no status-page component. |

### Tier 2: Content Management

⬜ **Not Applicable** (no editorial content).

### Tier 3: Security (Critical)

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Authentication Architecture | Not Met | No inbound auth. Registry says `auth: better-auth` but Better Auth assumes Node/Express patterns inconsistent with Workers + JSON API. Decide: HMAC-signed requests? Bearer tokens issued by another GPIN service? Document in ADR + `auth-architecture.md`. |
| ❌ | Session Management | Not Met | API-key model preferred (stateless). Document why no sessions. |
| ⬜ | RBAC Implementation | N/A (current scope) | Single principal (GPIN orchestrator) — but the approval workflow's manual/auto mode implies a "human approver" role. Capture in spec. |
| ⚠️ | Principle of Least Privilege | Partial | OpenAI/Notion/Pinata tokens grant whatever scope the provider gave. Document desired scopes per integration. |
| ❌ | Secrets Management | Not Met | See Critical #3 — `[vars]` instead of `wrangler secret put`. |
| ⬜ | JWT Best Practices | N/A | No JWT planned. |
| ⬜ | Parameterized Queries | N/A (current) | When D1 writes start, mandatory — D1 supports prepared statements; never string-interpolate user input. |
| ❌ | API Authentication | Not Met | See Critical #4. |
| ❌ | Audit Logging | Not Met | See Critical #6. The audit trail IS the product, and it doesn't exist. |
| ❌ | Security Testing | Not Met | No SAST, no DAST, no dependency-block in CI. |

### Tier 3: AI/LLM Integration Security

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Prompt Injection Defense | Not Met | No code. Auto-escalation trigger; cannot be deferred. Document structured prompt templates with explicit system/user boundaries; never concat caller input into the system role. |
| ❌ | Output Guardrails | Not Met | No code. Use OpenAI `response_format: json_schema` / function calling; validate against a Zod schema before any downstream effect (Notion write, Pinata pin, ENS query). |
| ❌ | Content Filtering | Not Met | No filter on input or output. Define out-of-scope categories in `security-architecture.md`. |
| ❌ | Fallback Behavior | Not Met | No code path for OpenAI rate-limit, 5xx, timeout, or schema-mismatch. Approval workflow should fail closed, not open. |
| ❌ | Cost Controls | Not Met | No per-request `max_tokens` budget, no per-caller rate limit, no spend alerts. A misconfigured caller can run the OpenAI bill to four figures in an afternoon. |

### Tier 3: Database & Data

| | Principle | Status | Evidence |
|---|---|---|---|
| ⚠️ | ACID Compliance | Partial | D1 (SQLite) provides ACID for single-row + small-tx — fine for the audit-log use case. **But** registry says `database: railway-postgres` — reconcile (Critical #7). |
| ❌ | Referential Integrity | Not Met | No schema exists. |
| ❌ | Indexing Strategy | Not Met | No schema. |
| ❌ | Migration Strategy | Not Met | No `migrations/` folder. Use `wrangler d1 migrations create`. |
| ⬜ | Soft Deletes | N/A (current) | Append-only audit trail by design — document. |
| ❌ | Backup & Recovery | Not Met | D1 backup is Cloudflare-managed; document RPO/RTO assumptions explicitly. |
| ⬜ | Connection Pooling | N/A | D1 access is via binding; no pooling concept. |
| ⬜ | Cache & Queue: Railway Redis | N/A | Worker uses CF KV; document the deviation in an ADR. |

### Tier 3: Architecture (SOLID + 12-Factor)

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Single Responsibility | Not Met | No code. |
| ❌ | Open/Closed | Not Met | No code. |
| ❌ | Dependency Inversion | Not Met | No code. |
| ⚠️ | Stateless Processes | Partial | CF Workers are stateless by definition; design intent consistent. No code to verify. |
| ⚠️ | Config in Environment | Partial | Intent in `wrangler.toml.backup`; live `wrangler.toml` empty. |
| ⚠️ | Backing Services as Attachments | Partial | D1/KV bindings declared in backup config. |
| ⚠️ | Dev/Prod Parity | Partial | `wrangler dev` documented in CLAUDE.md; no `.dev.vars` template committed. |
| ❌ | API-First Design | Not Met | No `api-spec.md`. |
| ❌ | API Versioning | Not Met | No versioning scheme defined. |

### Tier 3: Frontend Architecture

⬜ **Not Applicable** (no frontend).

### Tier 3: Design System Maturity

⬜ **Not Applicable** (no UI).

### Tier 3: Error & Edge Case UX (Application-Level)

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Conflict Resolution UX | Not Met | (n/a for end-user UX; equivalent: define behavior when two callers submit conflicting actions on the same resource — optimistic locking via D1 row version.) |
| ❌ | Rate Limit UX | Not Met | When OpenAI rate-limits us, define response shape to caller (`Retry-After`, queued vs rejected). |
| ⬜ | Offline-First Patterns | N/A | No mobile/client surface. |
| ❌ | Undo/Redo | Not Met | (n/a for end-user UX; equivalent: append-only audit log makes "undo" definitional — document.) |
| ⬜ | Session Expiration | N/A | No sessions. |

### Tier 3: AI/LLM Application Patterns

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Prompt Management | Not Met | No prompts yet, but the design (route agent actions through OpenAI) demands a versioned prompt store. KV is fine; document the pattern. |
| ❌ | Evaluation Framework | Not Met | No eval suite. Boundary tests (injection attempts, empty input, oversized input) belong in `testing-plan.md`. |
| ⬜ | Streaming UX | N/A | No end-user UX. If the gateway exposes streamed responses to callers, document; otherwise n/a. |
| ⬜ | Transparency & Disclosure | N/A (currently) | If outputs ever surface to end users via a downstream system, add disclosure metadata to the audit record. |
| ❌ | Data Privacy with AI Providers | Not Met | No DPA paragraph anywhere. OpenAI API tier assumed non-training but unverified. Document in `security-architecture.md`. |
| ❌ | Caching & Deduplication | Not Met | No caching. KV is well-suited (key = hash of prompt + params, value = response). |

### Tier 3: Resilience

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Graceful Degradation | Not Met | No code. |
| ❌ | Circuit Breaker | Not Met | No code. |
| ❌ | Retry with Backoff | Not Met | No code. |
| ❌ | Timeout Configuration | Not Met | No code. CF Workers has a 30s wall-clock by default — document. |
| ❌ | Health Checks | Not Met | No `/health` or `/ready` endpoint. |
| ⬜ | Graceful Shutdown | N/A | Workers are request-scoped; no long-running process. |

### Tier 3: Concurrency

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Idempotency Keys | Not Met | Critical for a gateway triggering external side-effects (Pinata pin, Notion write). Standard: caller-supplied `Idempotency-Key` header, dedup window in KV. |
| ❌ | Optimistic Locking | Not Met | D1 row version. |
| ❌ | Race Condition Awareness | Not Met | No code. |
| ⬜ | Distributed Locks | N/A | Single-instance worker model. |

### Tier 3: Testing Maturity

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Testing Pyramid | Not Met | No tests at any level. |
| ❌ | Test Isolation | Not Met | No tests. |
| ❌ | Test Data Factories | Not Met | No tests. |
| ❌ | CI Integration | Not Met | `deploy.yml` runs no tests. |
| ❌ | Coverage Thresholds | Not Met | N/A until tests exist. |
| ❌ | Load & Performance Testing | Not Met | No baselines. |
| ❌ | Contract Testing | Not Met | No OpenAPI/Pact contract between the gateway and its callers. |
| ❌ | Security Testing in CI | Not Met | No SAST. |
| ⬜ | Accessibility Testing | N/A | No UI. |

### Tier 3: Observability (Role Split)

| | Principle | Status | Evidence |
|---|---|---|---|
| ❌ | Structured Logging | Not Met | No Pino/JSON logger, no correlation IDs. CF Workers' built-in logs are line-oriented unless code formats them. |
| ❌ | Metrics Collection | Not Met | No Sentry custom metrics, no `captureBusinessError`. |
| ❌ | Distributed Tracing | Not Met | No `@sentry/cloudflare` instrumentation. UWP platform-constraint note explicitly calls out the wrong-SDK risk on Workers. |
| ❌ | Error Triage (Sentry) | Not Met | No Sentry DSN, no `initSentry` call, no PII scrubbing config. |
| ❌ | External Uptime + Status Page | Not Met | No Better Stack monitor. |
| ❌ | Alerting on Symptoms | Not Met | No alerts. |
| ❌ | Dashboards | Not Met | None. |
| ⬜ | RUM | N/A | No browser surface. |
| ⬜ | Session Replay | N/A | No browser surface. |
| ⬜ | Email Deliverability Audit | N/A | No transactional email sent. |

### Tier 3: Operational Maturity

| | Principle | Status | Evidence |
|---|---|---|---|
| ⚠️ | CI/CD Pipeline | Partial | Pipeline file exists but is non-functional (Critical #10). |
| ❌ | Infrastructure as Code | Not Met | `wrangler.toml` would BE the IaC; currently empty. |
| ❌ | Feature Flags | Not Met | No flag system. `APPROVAL_MODE` in backup config is a single env-var toggle, not a flag system. |
| ❌ | Rollback Capability | Not Met | CF Workers has built-in rollback (`wrangler rollback`); not documented in a runbook. |
| ❌ | Runbooks | Not Met | No `runbooks.md`. For a worker proxying four external APIs, the "OpenAI is down, what now?" runbook is overdue. |

### Tier 3: Internationalization

⬜ **Not Applicable** (no end-user output).

---

### Cross-Cutting: Privacy & Data Protection (Tier 3 checklist)

| | Item | Status | Evidence |
|---|---|---|---|
| ⬜ | Privacy policy | N/A | No public-facing collection. **However**, document the data-flow paragraph (what user data, if any, flows to OpenAI/Notion/Pinata/ENS) in `security-architecture.md`. |
| ⬜ | Cookie disclosure | N/A | No cookies. |
| ⚠️ | Secure form submission | Partial | HTTPS-only at edge. |
| ⚠️ | Data minimization | Partial | Design intent — no schema yet. |
| ❌ | Data subject rights | Not Met | If human data is ever logged in D1, define DSR purge path. |
| ❌ | Consent management | Not Met | n/a if no user data; explicitly say so. |
| ❌ | DPAs with all processors (including AI) | Not Met | Not documented. OpenAI, Notion, Pinata all require DPA awareness. |
| ❌ | Access logging for personal data | Not Met | No logging. |
| ❌ | Retention schedules | Not Met | D1 audit-log retention undefined. |
| ❌ | Privacy policy covers all processing (incl. AI) | Not Met | No privacy policy exists. |
| ❌ | Breach notification procedure | Not Met | Not documented. |

### Cross-Cutting: AI/LLM Integration

Tier 3 escalation was driven by the OpenAI integration. Every AI principle in Tier 3 (above) applies. All ❌ except the "n/a for end-user UX" rows. The lack of any AI-specific implementation or documentation is the single largest cluster of unmet principles in this audit.

### Cross-Cutting: Dependency Management

| | Item | Status | Evidence |
|---|---|---|---|
| ✅ | Lock file committed | Met | `package-lock.json` present. |
| ✅ | `npm audit` clean | Met | 0 vulnerabilities, 29 deps total (mostly devdeps), re-run 2026-06-25. |
| ⚠️ | Evaluation criteria documented | Partial | No `coding-standards.md` capturing the dependency-add policy. |
| ❌ | License audit | Not Met | Not run. ISC/MIT-dominated devdeps make this low-risk but undocumented. |
| ❌ | Dependabot config | Not Met | No `.github/dependabot.yml`. |
| ⚠️ | Minimal dependencies | Partial | Genuinely minimal today (3 devdeps + transitives). Risk is what gets added once `src/` lands. |

### Cross-Cutting: Structural Integrity (v4.01)

| | Item | Status | Evidence |
|---|---|---|---|
| ❌ | No swallowed errors | Not Met | No code yet. Document the rule in `coding-standards.md`. |
| ❌ | No manual sync steps | Not Met | The "empty `wrangler.toml`, real config in `config/wrangler.toml.backup`" split is itself a manual-sync trap. Restore the live config or delete the backup. |
| ❌ | Actionable error messages | Not Met | No code. |
| ⬜ | CLI commands support headless | N/A | No CLI surface. |
| ❌ | Singular source of truth | Not Met (Tier 2+) | The empty `wrangler.toml` + populated `wrangler.toml.backup` split **violates** this directly. **New this audit:** `.github/workflows/agent-spawn.yml` references `docs/ADR-004-agent-spawn-trigger.md` which does not exist — a dangling-reference drift. |
| ❌ | No workaround comments | Not Met (Tier 3+) | n/a today (no code). |
| ❌ | Explicit API contracts | Not Met (Tier 3+) | No `api-spec.md`, no OpenAPI/JSON-Schema. |
| ⚠️ | Commit gate hermetic and fast | Partial (Tier 2+) | No commit gate exists, so it trivially can't be infra-coupled. Becomes live once tests are added. |

### Cross-Cutting: Project Hub Registration

| | Item | Status | Evidence |
|---|---|---|---|
| ✅ | Registered in Project Hub | Met | Entry at `registry/projects.json` lines 1384–1417. |
| ✅ | Tier matches classification | Met | Registry `tier: 3` aligns with `PROJECT_CLASSIFICATION.md` Tier 3 result. |
| ✅ | Git remote configured | Met | `https://github.com/GetDigitalOS/gpin-agent-gateway.git`. |
| ✅ | Dev port assigned | Met | `3018` (in getdigital range). |
| ⚠️ | Status is current | Partial | `status: "development"` is honest; `last_audited` was `2026-05-23T19:18:07Z` and **could not be updated by this audit** — the hub repo blocks edits to `main` from this session (PreToolUse hook). The bump to `2026-06-25` must be made via a feature-branch PR on the hub repo. |
| ❌ | Stack profile matches reality | **Not Met** | **Most important hub finding (unchanged since last audit).** Registry declares `auth: better-auth`, `database: railway-postgres`, `seo: required`. Actual design (CLAUDE.md, backup wrangler config, classification doc) uses D1 + KV, no Better Auth, no Postgres, no public web surface for SEO. `/scaffold` and `/compliance` mis-target this project until reconciled. |

---

## Recommendations (Prioritized)

### 🔴 Critical (Security / Deploy / Compliance Risk)

1. **Reconcile or delete the empty `wrangler.toml`.** Either restore from `config/wrangler.toml.backup` (secrets moved to `wrangler secret put`) or delete both files and document the "scaffold-only, no live deploy" state in CLAUDE.md. Today's repo would `wrangler deploy` an empty worker.
2. **Move all API secrets to `wrangler secret put`.** Remove `OPENAI_API_KEY`, `NOTION_SECRET`, `NOTION_DB_ID`, `PINATA_JWT` from `[vars]` in any committed config. Document the rotation pattern in `security-architecture.md`.
3. **Fix the registry stack profile (hub PR).** Change `auth` → `none` (or document the real auth design), `database` → `cloudflare-d1`, `seo` → `n/a`, and bump `last_audited` → `2026-06-25`. Open a feature-branch PR against project-hub (main is hook-blocked).
4. **Implement inbound authentication before any code that calls a paid API.** Bearer token or HMAC-signed request — write the ADR, then the code, in that order.
5. **Make CI functional.** Add a real `build` script (`tsc --noEmit` + esbuild bundle), an `npm audit` step, and a test step to `deploy.yml`. Today `npm run build` fails on every push.

### 🟡 Important (Quality / Reliability Risk)

6. **Write the Tier 3 documentation backlog.** Minimum to unblock implementation: `api-spec.md`, `data-model.md` (D1 audit-log schema), `system-architecture.md`, `auth-architecture.md`, `security-architecture.md` (DPAs, secret rotation, prompt-injection defense), `error-handling.md`, `testing-plan.md`, `deployment.md`, `monitoring.md`.
7. **Capture the deferred ADRs** — and write the **missing ADR-004 that `agent-spawn.yml` already references**. At least: ADR-001 Tier classification, ADR-002 CF Workers vs Railway, ADR-003 D1 + KV vs Postgres + Redis, ADR-004 agent-spawn trigger (dangling reference), ADR-005 inbound auth scheme, ADR-006 prompt management & versioning, ADR-007 SEO-not-applicable. Tier 3 minimum is 8–12.
8. **Add rate limiting + idempotency.** Inbound per-caller token bucket in KV; `Idempotency-Key` header for any side-effecting route.
9. **Wire `@getdigitalos/observability`** (the `@sentry/cloudflare` path per UWP platform constraint). Add `SENTRY_DSN` as a secret, `SENTRY_AUTH_TOKEN` to CI for source maps.
10. **Implement the audit log.** D1 schema + migration + write path on every action. This is the product, not an afterthought.

### 🟢 Recommended (Best Practice)

11. **Preview deployments** via `wrangler --env preview` on PRs.
12. **Dependabot** config (weekly; ecosystems: npm + github-actions).
13. **PR template + CODEOWNERS** for the gateway repo.
14. **Fill the empty CLAUDE.md Overview section** and the empty `README.md` (currently one line); verify the recent mojibake cleanup is reflected in CLAUDE.md.
15. **AI cost controls** — per-request `max_tokens`, per-caller rate cap, spend alert in the OpenAI dashboard, KV-backed response cache keyed by (prompt-hash, params).
16. **Health endpoint** — `GET /health` returning JSON liveness + downstream-service ping status.

---

## Phase Readiness

- **Development:** Not blocked, but the scaffold is misleading (empty active config alongside a populated backup config invites drift). Closing 🔴 #1 makes the repo honest.
- **Staging:** Blocked by 🔴 #1, #2, #4, #5.
- **Launch:** Blocked by 🔴 #1–#5, plus 🟡 #6, #8, #10. The audit-logging gap (#10) is the deal-breaker for a system whose explicit purpose is governance of autonomous agent actions.

## Next Review

Recommended next compliance check: **2026-07-30**, or sooner once `src/` lands meaningful code — at that point the audit shifts from "scaffold gaps" to "implementation gaps" and many ⬜/n/a items become live ✅/❌ checks.

# GPIN Agent Gateway

## Docs-First Rule (MANDATORY)

This block is the **standing instruction** for every Claude Code session, every sub-agent, and every /command invocation in this workspace.

### Core Rule (enforced on every single interaction)

**For every plan, build, feature, refactor, bugfix, or code change request:**

1. **Immediately and fully read the entire `/docs` folder** before doing anything else.
   - Treat `/docs` as the single source of truth for architecture, decisions, specifications, planning, examples, and reference material.
   - Respect this exact layout (do not create folders outside this structure without explicit approval):
     ```
     docs/
     ├── architecture/
     ├── adr/               (Architecture Decision Records — Tier 1+)
     ├── examples/
     ├── operations/        (deployment, monitoring, runbooks, incident-response — Tier 3+)
     ├── planning/
     ├── reference/
     ├── specifications/
     └── (any .md files explicitly added at top level)
     ```
   - If a `/docs` folder does not yet exist, note that and proceed — but flag it as needing backfill.

2. **Maintain documentation as you go — lean & zero-bloat rule**
   - Only update or add to `/docs` when the change is **meaningful** for a new developer or future team handoff.
   - Prefer **editing an existing file** over creating a new one.
   - Never add filler, repetition, or "nice-to-have" sections.
   - Keep every file concise, scannable, and accurate.
   - If something becomes outdated → mark it `[DEPRECATED — see new location]` or move it to a `docs/archive/` subfolder. Never delete history.
   - After any code change that affects design, specs, architecture, or workflow, **always include** in your response:
     > "Documentation update required: [exact file(s) + 1-sentence summary of what changed]"
   - **Operational capture (Tier 3+):** after a deploy, a production-incident debug, or an outage, record the playbook in `docs/operations/runbooks.md` (detection signal → first response → diagnosis → remediation → escalation), and note post-incident learnings in `docs/operations/incident-response.md`. Runbooks are **designed to accrete from real operational work** — capture the entry in the same session you did the ops work; don't defer it to a cold backfill pass.

3. **Documentation philosophy (enforced)**
   - Living, minimal, high-signal only.
   - One source of truth per topic.
   - No duplicate information between files.
   - Update immediately when the project evolves — never let docs drift from reality.
   - Goal: A new senior developer could be fully productive in <2 hours just by reading `/docs`.

### Canonical References (portfolio-wide standards)

Do NOT copy canonical documents into this project. Instead, read them from the hub:
- **Universal Web Development Principles**: `C:/dev/project-hub/canonical/references/Universal_Web_Development_Principles.md`
- **Versioning Standards**: `C:/dev/project-hub/canonical/references/VERSIONING_STANDARDS.md`

These are the single source of truth. If you need to reference a principle, point to the canonical path — never create a local copy.

### How this interacts with hub sync

- This `Docs-First Rule` block is **hub-canonical** — `hub sync` may re-insert it if removed.
- Everything else in this `CLAUDE.md` is **project-owned** — hub will never overwrite it.
- If hub updates this block, accept the update. It will never conflict with project-specific content.


## Overview

## Session Continuity — READ FIRST

At the start of every session, read `docs/planning/NEXT_SESSION.md` before doing anything else — it carries the paste-ready handoff prompt and the current open follow-ups. If that file does not exist yet, create it at the end of your first session that lands material work, using the template in `C:/dev/project-hub/canonical/standards/session-continuity.md`. Update it at the end of every session that lands material work (on `main`, or on the active feature/integration branch when work-in-flight hasn't merged yet). If it is stale — its last-updated date is older than the most recent commit on the branch it describes — say so and update it as part of the session.

*(This block is hub-owned and Tier 3+ only; `hub sync` may re-insert it. Everything else in this CLAUDE.md is project-owned.)*

Cloudflare Worker acting as an agent gateway for GPIN autonomous agent infrastructure. Routes and governs agent actions across OpenAI, Notion, Pinata (IPFS), and Ethereum ENS, with an approval workflow system (manual/auto modes) and full audit trail via D1 and KV.

## Classification

- **Tier:** 3 — Application
- **Framework:** Universal Web Development Principles
- **Classification:** See `docs/architecture/PROJECT_CLASSIFICATION.md`

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Cloudflare Workers |
| Language | TypeScript / JavaScript |
| Database | Cloudflare D1 (audit trail) |
| KV | Cloudflare KV (event queue, outbox) |
| Integrations | OpenAI API, Notion API, Pinata (IPFS), Ethereum ENS RPC |
| Package Manager | npm |

## Key Commands

```bash
wrangler dev          # Start local dev server
wrangler deploy       # Deploy to Cloudflare Workers
wrangler tail         # Live log streaming
```

## Architecture

All agent actions flow through the gateway's approval workflow:
- **Manual mode** — Actions queue for human approval before execution
- **Auto mode** — Actions execute immediately with audit logging

### External Integrations
- **OpenAI**: LLM completions and embeddings for agent intelligence
- **Notion**: Document and knowledge base read/write operations
- **Pinata (IPFS)**: Decentralized file storage for agent artifacts
- **Ethereum ENS**: Decentralized identity resolution

### Storage
- **D1 database** — Persistent audit trail of all agent actions
- **KV (events)** — Incoming event queue for agent requests
- **KV (outbox)** — Outgoing action queue for processed results

## Git Conventions

- **Remote**: `https://github.com/GetDigitalOS/gpin-agent-gateway`
- **Email**: `45609802+getdigital2020@users.noreply.github.com`
- **Branch**: `main`

## Canonical References

Do NOT copy these into this project. Read from the hub:
- **Universal Web Development Principles**: `C:/dev/project-hub/canonical/references/Universal_Web_Development_Principles.md`
- **Versioning Standards**: `C:/dev/project-hub/canonical/references/VERSIONING_STANDARDS.md`

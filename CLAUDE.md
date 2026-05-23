# GPIN Agent Gateway

## Overview

## Session Continuity — READ FIRST

At the start of every session, read `docs/planning/NEXT_SESSION.md` before doing anything else — it carries the paste-ready handoff prompt and the current open follow-ups. If that file does not exist yet, create it at the end of your first session that lands material work, using the template in `C:/dev/project-hub/canonical/standards/session-continuity.md`. Update it at the end of every session that lands material work (on `main`, or on the active feature/integration branch when work-in-flight hasn't merged yet). If it is stale — its last-updated date is older than the most recent commit on the branch it describes — say so and update it as part of the session.

*(This block is hub-owned and Tier 3+ only; `hub sync` may re-insert it. Everything else in this CLAUDE.md is project-owned.)*

Cloudflare Worker acting as an agent gateway for GPIN autonomous agent infrastructure. Routes and governs agent actions across OpenAI, Notion, Pinata (IPFS), and Ethereum ENS, with an approval workflow system (manual/auto modes) and full audit trail via D1 and KV.

## Classification

- **Tier:** 3 â€” Application
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
- **Manual mode** â€” Actions queue for human approval before execution
- **Auto mode** â€” Actions execute immediately with audit logging

### External Integrations
- **OpenAI**: LLM completions and embeddings for agent intelligence
- **Notion**: Document and knowledge base read/write operations
- **Pinata (IPFS)**: Decentralized file storage for agent artifacts
- **Ethereum ENS**: Decentralized identity resolution

### Storage
- **D1 database** â€” Persistent audit trail of all agent actions
- **KV (events)** â€” Incoming event queue for agent requests
- **KV (outbox)** â€” Outgoing action queue for processed results

## Git Conventions

- **Remote**: `https://github.com/GetDigitalOS/gpin-agent-gateway`
- **Email**: `45609802+getdigital2020@users.noreply.github.com`
- **Branch**: `main`

## Canonical References

Do NOT copy these into this project. Read from the hub:
- **Universal Web Development Principles**: `C:/dev/project-hub/canonical/references/Universal_Web_Development_Principles.md`
- **Versioning Standards**: `C:/dev/project-hub/canonical/references/VERSIONING_STANDARDS.md`

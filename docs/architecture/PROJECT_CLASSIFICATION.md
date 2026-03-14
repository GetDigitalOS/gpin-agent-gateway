# GPIN Agent Gateway - Project Classification

## Universal Web Development Principles Assessment

Classified by: Project Hub (automated)
Date: 2026-03-14

---

## Classification Questionnaire

| # | Question | Answer | Notes |
|---|----------|--------|-------|
| 1 | **User Authentication**: Does the app require user accounts, login, or personalized data? | **YES** | API keys for OpenAI, Notion, Pinata; approval workflow governance |
| 2 | **Data Persistence**: Is there a database with user-generated content that must be stored/retrieved? | **YES** | D1 database (AUDIT_D1) for audit trail; KV stores for events/outbox |
| 3 | **Multi-Role Access**: Are there different user types with different permissions? | **YES** | APPROVAL_MODE (manual/auto) indicates role-based agent governance |
| 4 | **Third-Party Integrations**: Does it connect to external APIs or services? | **YES** | OpenAI, Notion, Pinata (IPFS), Ethereum ENS RPC |
| 5 | **Real-Time Features**: Are there live updates, websockets, or collaborative features? | **MAYBE** | KV event/outbox pattern suggests async event processing |
| 6 | **Transaction Sensitivity**: Does it handle payments, PII, health data, or financial info? | **YES** | API secrets, audit trail data |
| 7 | **Scale Expectations**: Will this serve >10,000 users? | **NO** | Internal agent gateway |
| 8 | **Team Size**: Will multiple developers work on this codebase? | **NO** | Single developer |
| 9 | **Longevity**: Is this expected to be maintained for 2+ years? | **YES** | Strategic infrastructure for GPIN agent orchestration |
| 10 | **AI Features**: Does it use AI/ML models or LLM integrations? | **YES** | OpenAI API integration for agent functionality |

**Total "YES" answers: 7**

---

## Auto-Escalation Rules

- AI/LLM integration → Escalates to Tier 3+
- External API secrets management → Escalates security requirements
- Audit trail (D1) → Compliance and data integrity requirements

---

## Project Tier: **Tier 3 - Application**

This project requires all principles from:
- Tier 1: Static/Marketing (baseline)
- Tier 2: Interactive (forms, external integrations)
- **Tier 3: Application** (auth, database, external APIs, audit trail)

**Rationale:** Cloudflare Worker acting as an agent gateway with D1 database for audit trails, KV namespaces for event processing and outbox patterns, and integrations with OpenAI, Notion, Pinata (IPFS), and Ethereum ENS. Includes approval workflow governance (manual/auto modes). Currently in scaffold phase — config defined but no source code implemented yet.

---

## Technical Stack

| Component | Technology | Tier Compliance |
|-----------|-----------|-----------------|
| Runtime | Cloudflare Workers | Tier 3 |
| Language | TypeScript 5.9 | Tier 3 |
| Database | Cloudflare D1 (SQLite) | Tier 3 |
| KV Storage | Cloudflare KV (EVENTS_KV, OUTBOX_KV) | Tier 3 |
| AI | OpenAI API | Tier 3 |
| Integrations | Notion, Pinata (IPFS), Ethereum ENS | Tier 3 |
| Deploy | Cloudflare Workers (GitHub Actions) | Tier 3 |

---

## Development Status

**Scaffold phase** — `config/wrangler.toml.backup` contains full infrastructure config (D1, KV, secrets) but no source code exists yet. The main `wrangler.toml` is empty and `src/index.ts` has not been created.

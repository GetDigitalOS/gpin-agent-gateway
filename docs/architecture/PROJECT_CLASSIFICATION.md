# Project Classification — gpin-agent-gateway

| Field | Value |
|-------|-------|
| **Generated** | 2026-03-22 |
| **Method** | Auto-classified from project documentation |
| **Result** | **Tier 3 — Application** (4/10 yes) |

## Assessment

| # | Question | Answer | Rationale |
|---|----------|--------|-----------|
| auth | User Authentication | No | No user accounts or login — this is an agent gateway with an approval workflow, not a user-facing authenticated application. |
| data | Data Persistence | **Yes** | Uses Cloudflare D1 for a persistent audit trail of all agent actions and KV for event/outbox queues. |
| roles | Multi-Role Access | No | No multi-role access described — the gateway operates in manual or auto approval modes, not per-user role-based access. |
| integrations | Third-Party Integrations | **Yes** | Integrates with four external services: OpenAI API, Notion API, Pinata (IPFS), and Ethereum ENS RPC. |
| realtime | Real-Time Features | No | No websockets or live collaborative features — it uses async event queues (KV) for request/response processing. |
| sensitive | Transaction Sensitivity | No | No payments, PII, health data, or financial transactions — it handles agent actions and audit logs. |
| scale | Scale Expectations | No | No indication of >10,000 users or heavy concurrent load — this is an internal agent infrastructure gateway. |
| team | Team Size | No | No evidence of multiple developers — minimal package.json with no test framework or CI setup suggests a single-developer project. |
| longevity | Longevity | **Yes** | Part of the GPIN autonomous agent infrastructure with audit trail and governance, indicating long-term operational intent. |
| ai | AI/LLM Features | **Yes** | Integrates OpenAI for LLM completions and embeddings powering agent intelligence. |

## Auto-Escalation Triggers

- **ai** → Prompt injection defense requires Tier 3 rigor

## Tier Determination

**Tier 3 — Application**

- Yes count: 4/10
- Count-based tier: 3
- Escalation-based tier: 3
- Final tier: 3 (max of count + escalation)

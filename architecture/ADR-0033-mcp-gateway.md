# ADR-0033 — racunAI MCP Gateway Architecture

```text
Status: Proposed
Date: 2026-11-27
Type: Integration
Supersedes: —
Related: ADR-0010, ADR-0011, ADR-0013, ADR-0014, ADR-0020, ADR-0021, ADR-0022, ADR-0025, ADR-0027
```

## Status
**Proposed** — architecture owner approval required; no implementation or deployment authorized.

## Context
Users should be able to query authorized ERP data conversationally in ChatGPT without screenshots or manual exports. The existing REST/OpenAPI contract and domain services are the source of business truth. The [MCP plan](../integrations/RACUNAI-MCP-01-PLAN.md) inventories 27 candidate GET tools against API OpenAPI blob 6bb55c144aa8c1140030e3ad0eb2907bd0870cb8. Endpoint presence is not proof of runtime authorization or side-effect freedom. This cross-cutting external integration introduces an identity and data disclosure boundary, warranting an ADR without changing frozen domain architecture.

## Decision
The following decisions are proposed as a single architecture baseline.

| ID | Decision |
|---|---|
| D01 | Separate gateway repository racunai-hr/mcp; stack owns deployment; docs owns architecture. |
| D02 | Existing Django REST API and canonical domain services remain authoritative. No direct DB/ORM access or duplicated accounting, tax or banking logic. |
| D03 | Python, official MCP SDK, pinned dependencies, typed HTTP client and Docker. |
| D04 | Remote Streamable HTTP over TLS; proposed production endpoint https://mcp.racunai.hr/mcp. DNS and deployment require separate approval. |
| D05 | OAuth 2.1 authorization code with PKCE, protected resource metadata and delegated user identity. Existing JWT endpoints are not evidence of OAuth compatibility. |
| D06 | Explicit tenant per business call; membership and role checked server-side at gateway and upstream. No default tenant fallback. |
| D07 | V1 read-only; only verified side-effect-free GET handlers. No POST, PATCH, PUT or DELETE. |
| D08 | Curated typed tool allowlist mapped to OpenAPI operations. No automatic exposure of all endpoints, arbitrary HTTP, SQL or shell. |
| D09 | Auditable calls with actor, tenant, tool, timestamp, correlation ID, outcome and latency; redact sensitive content. |
| D10 | Future writes require separate authorization, exact action confirmation and canonical domain services. |
| D11 | Standard MCP protocol independent of AI vendor; ChatGPT is first target client, client-specific compatibility stays at adapter boundary. |
| D12 | Separate development, stage and production identities, credentials, audiences and configuration. No production data in development tests. |

### Architecture and ownership
```text
ChatGPT / compatible MCP client
    -> HTTPS / OAuth -> MCP Gateway
    -> delegated authenticated REST -> Django API
    -> canonical domain services -> tenant-scoped persistence
```
Gateway owns transport, schema validation, authorization orchestration, response shaping, rate limits and audit. API owns tenant enforcement, business authorization, invariants and authoritative data. No new Django domain, event or persistence model is implied. Existing OpenAPI generation remains canonical; generated files must not be edited manually.

### Identity and trust boundary
Validate issuer, audience, expiry, scopes and revocation. MCP and API audiences are separate; no blind token passthrough. Verify delegated identity/token exchange or design backend-enforced scoped delegation before rollout. No global service account, shared user identity or ERP passwords in chat. Resolve tenant origin from trusted registry, never arbitrary URLs. Recheck access for references, pagination and retries; deny foreign tenant IDs even when local IDs collide. Fail closed if membership cannot be established. Proposed read scopes: partners:read, documents:read, finance:read, banking:read, tax:read, assets:read. Identity introspection must not expose credentials.

### V1 capabilities and data contract
Initial pilot: whoami, partner and document search/detail, journal list/detail, partner subledger, bank transactions, VAT period. Expand to remaining verified candidates in plan only after tests. Tool input has typed bounded filters and explicit tenant for business data. Output includes tenant, source IDs, verified authorized links, retrieval timestamp, pagination/completeness; preserve decimal strings, currencies, dates and statuses. Distinguish missing, zero, incomplete and forbidden; do not infer authoritative balances from partial results. Tool annotations are hints, not security controls. Support search/fetch compatibility if required by target client; opaque IDs must be tenant-bound and reauthorized.

No dedicated tenant-list, account balance, aging or standalone GET detail for expense, bank transaction or vehicle was identified in the plan snapshot. Verify canonical alternatives before proposing new API contracts. PDF/UBL/XML, attachments, bulk exports, HR/payroll/JOPPD, partner contacts/bank accounts and payment orders are excluded from V1. Posting preview requires side-effect inspection before inclusion. Document text is untrusted data, never instructions. Bound response size, pagination, timeouts and outbound destinations; avoid existence leaks, raw tracebacks and sensitive logs. Privacy, retention and consent review precede real financial data disclosure.

### Future write boundary
Drafts that persist are writes. No implicit authorization from conversation, tool annotations or prior read access. A separately approved write design must bind explicit confirmation to actor, tenant, exact payload, resource version, expiry and idempotency key; revalidate at execution. Posting, reversal, reconciliation, payments and tax submission each require individual risk gates. No direct edits to posted journals, submitted XML or payload files. Development diagnostics are optional and separately scoped; reuse GitHub connector for repository work.

### Alternatives considered
- Direct DB access: rejected; bypasses domain and tenant authorization.
- MCP inside Django: not selected; couples external protocol and security lifecycle to ERP core. Reconsider only through amended ADR if operational evidence warrants.
- Generic OpenAPI auto-exposure: rejected; expands attack surface and exposes unintended mutations.
- ChatGPT-only proprietary integration: not selected; standard protocol reduces client lock-in, but compatibility is not guaranteed.

## Consequences
### Benefits
- Reuse existing business contracts, preserve accounting invariants and audit.
- Independent integration lifecycle and narrowly scoped rollout.
- Conversational access with traceable source data and explicit completeness.
### Risks / trade-offs
- Additional service and OAuth/delegation complexity.
- Tenant routing, prompt injection and data disclosure risk require dedicated tests.
- API gaps and contract drift may limit workflows.
- ChatGPT account entitlement and current remote MCP requirements remain unverified; no workaround that weakens security.
### Follow-up
- [ ] Architecture owner accepts ADR; implementation GO remains separate.
- [ ] MCP-01: verify current OpenAPI operation IDs, filters, handler side effects, auth and tenant membership; produce tool registry, threat model and test plan.
- [ ] Verify official ChatGPT remote MCP requirements and actual account entitlement; document blocker if unsupported.
- [ ] MCP-02 foundation; MCP-03 read-only pilot; MCP-04 ChatGPT integration on synthetic stage data.
- [ ] MCP-07 security/privacy/operations gate before any production read-only pilot; independent of MCP-05/06 write phases.
- [ ] MCP-05/06 separately approve draft/write capability design; MCP-08 optional developer diagnostics.
- [ ] Acceptance AC01–AC14 from linked plan: tenant isolation, token and scope enforcement, no writes, fidelity, error handling, redaction, injection resistance, contract drift, ChatGPT E2E, audit and authorized references.
- [ ] Reuse local CI infrastructure and independent GitHub CI checkpoints; isolated test DB and --keepdb for applicable Django tests. Record SHA, results and blockers.
- [ ] Prepare runbook, kill switch, monitoring, rotation and rollback before production authorization.

No repository creation, API changes, DNS, deployment, production access or secrets changes are authorized by this Proposed ADR.

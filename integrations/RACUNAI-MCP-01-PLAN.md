# RACUNAI-MCP-01 — racunAI MCP Gateway
Status: PROPOSED — plan only, implementation requires approval
Date: 2026-11-27

## 1. Goal and boundaries
Enable authenticated ChatGPT users to work with their authorized racunAI tenant using curated business tools. Preserve API-first architecture, domain services, immutable accounting, submission lifecycle and audit. MCP is an integration adapter, not a new business domain. No direct database access, arbitrary HTTP/SQL/shell tools, model writes or duplicated tax/accounting logic.

Proposed separate repository racunai-hr/mcp (not created), Python official MCP SDK, HTTP client, Docker, remote Streamable HTTP endpoint https://mcp.racunai.hr/mcp (proposed DNS, not provisioned). Deployment configuration belongs to stack; documentation to docs. Use existing development environment, then isolated stage before production. GitHub connector remains the source for repository operations.

## 2. Evidence and capability inventory
Source: api/openapi.yaml on develop, blob 6bb55c144aa8c1140030e3ad0eb2907bd0870cb8; api/README.md (generated contract, never edit manually); docs/architecture/NORTH_STAR.md and ADR-0010, ADR-0011, ADR-0013, ADR-0014, ADR-0020, ADR-0021, ADR-0022. Inventory below verifies method/path presence, not runtime permissions, completeness or side-effect freedom. Before exposing each tool inspect handler, serializers, query parameters, auth, tenant resolution and tests. All paths below are relative to the trusted tenant API origin.

### V1 allowlist candidates (GET only)
| Tool | Existing GET path | Notes |
|---|---|---|
| whoami | /api/auth/me/ | Verify membership claims; no token exposure |
| search_partners | /api/partners/ | Supported filters only |
| get_partner | /api/partners/{id}/ | Minimize personal fields |
| search_documents | /api/documents/ | Canonical document read model |
| get_document | /api/documents/{direction}/{id}/ | Invoice/expense read model; preserve direction |
| list_accounts | /api/finance/chart-of-accounts/ | Tenant chart |
| list_journal_entries | /api/finance/journal-entries/ | Bounded filters |
| get_journal_entry | /api/finance/journal-entries/{id}/ | Source links and status |
| get_partner_summary | /api/finance/partners/{id}/financial-summary/ | Canonical totals |
| get_partner_statement | /api/finance/partners/{id}/statement/ | Date bounds |
| get_partner_subledger | /api/finance/partners/{id}/subledger/ | Open items |
| list_bank_transactions | /api/banking/transactions/ | Minimize IBAN and narrative |
| get_banking_overview | /api/banking/overview/ | Verify aggregation |
| list_bank_statements | /api/banking/statements/ | Bounded |
| get_bank_statement | /api/banking/statements/{id}/ | Bounded detail |
| get_open_item_candidates | /api/banking/transactions/{id}/open-item-candidates/ | No matching |
| list_vat_periods | /api/tax/pdv/periods/ | Status |
| get_vat_period | /api/tax/pdv/periods/{period}/ | Preserve submission status |
| get_vat_boxes | /api/tax/pdv/periods/{period}/boxes/ | Evidence and mapping |
| get_pdv_s_period | /api/tax/pdv-s/periods/{period}/ | Read only |
| get_zp_period | /api/tax/zp/periods/{period}/ | Read only |
| list_fixed_assets | /api/assets/fixed-assets/ | Bounded |
| get_fixed_asset | /api/assets/fixed-assets/{id}/ | Canonical values |
| get_depreciation_schedule | /api/assets/fixed-assets/{id}/depreciation-schedule/ | Read only |
| get_asset_journal_entries | /api/assets/fixed-assets/{id}/journal-entries/ | Traceability |
| list_vehicles | /api/finance/vehicles/ | No inferred VIN search |
| get_posting_preview | /api/finance/expenses/{id}/posting-preview/ | Verify no persistence or side effects before allowlisting |

Start with whoami, partners, documents, journals, subledger, transactions and PDV period; add remaining candidates after tests. Tool names are proposals, not implemented operations.

### Gaps / not authorized in V1
- list_tenants: no dedicated GET path found; inspect auth/me response and membership service. Add scoped API only if necessary.
- Standalone GET expense detail, transaction detail, vehicle detail, account balance and aging: no dedicated GET operation found in this snapshot. Use existing canonical read models where sufficient; otherwise propose new domain API contract. Never infer missing facts or calculate authoritative balances from incomplete pages.
- PDF/UBL/attachments/export and submission XML: existing paths, excluded initially for privacy, size and prompt injection. Later use authorized document references and bounded extraction.
- HR/payroll/JOPPD, partner contacts/bank accounts, payment orders: excluded initially; separate scopes and privacy review.
- POST accounting-review and PDV ledger paths are not assumed read-only. No POST in V1.
- No generic OpenAPI-to-MCP auto-exposure; contract generation may assist typed clients, not authorization.

## 3. Identity and security
OAuth 2.1 authorization code + PKCE with compatible authorization server and protected-resource metadata; verify current ChatGPT client requirements and account entitlement before implementation. Existing auth/token and refresh endpoints do not establish OAuth compatibility. Do not collect ERP passwords in chat. Validate issuer, audience, expiry, scopes, revocation; separate MCP and upstream API audiences, no blind token passthrough. Prefer delegated user identity; if token exchange is unavailable, design scoped delegation with backend enforcement before rollout. No global service-account access.
Every tool requires explicit tenant, checked against authenticated membership and role on every request. Resolve tenant origin server-side from trusted registry; no user-supplied URLs or default tenant fallback. Backend independently enforces permissions and tenant scope. Separate stage/prod credentials and audiences; fail closed. No cross-tenant bulk aggregation by default. Recheck access on pagination, references and retries.
Scopes proposed: partners:read, documents:read, finance:read, banking:read, tax:read, assets:read; least privilege, no write scopes in V1. Treat tool annotations as hints, not enforcement. Restrict outbound destinations, timeouts, rate limits, bounded pagination, response bytes, redacted logs; TLS, secret manager/env injection, no secrets in Git. Do not expose raw tracebacks. Treat document text as untrusted data, never instructions. Review data processing, retention and user consent before real financial data leaves ERP.

## 4. Tool contract
Typed JSON schemas, explicit tenant and bounded filters, no arbitrary URL or SQL. Output includes tenant, source entity IDs, authorized app links where verified, retrieved_at, pagination and completeness; preserve decimal strings, currency, business dates and statuses. Distinguish zero, missing, forbidden and incomplete. No fabricated source links. Tool errors: unauthenticated, forbidden, invalid input, not found, rate limited, upstream unavailable; avoid existence leakage across tenants. Read tools have readOnlyHint=true only after handler verification. Search/fetch compatibility tools if required by target ChatGPT mode, with tenant-bound opaque IDs and authorized source links.
Audit correlation ID, actor, tenant, tool, outcome, timestamp, latency; redact sensitive parameters and payloads. Audit reads without logging entire documents.

## 5. Execution sequence and gates
- MCP-01 — inventory and ADR: verify current OpenAPI operation IDs, actual filters, auth/membership implementation, handler side effects, missing endpoints, and official ChatGPT remote MCP support. Deliver accepted ADR, tool registry, threat model and test plan. No implementation until approval.
- MCP-02 — gateway foundation: create proposed repo only after approval; pinned SDK, typed API client, OAuth integration, tenant resolver, audit, Docker and health endpoints. Local CI on existing development server; mock upstream and test tenant isolation. No production access.
- MCP-03 — read-only pilot: implement initial allowlist, contract drift checks and integration tests against isolated stage with synthetic tenants. Add remaining tools incrementally.
- MCP-04 — ChatGPT integration: connect remote MCP in supported custom app/developer mode; test authentication, consent, tool discovery, search/fetch if required, revocation and real conversational workflows. If plan does not support custom MCP, record blocker; do not weaken auth. No production rollout.
- MCP-05 — draft/preview design: separate approval for writes, inspect existing purchasing import/confirm, partner create and finance preview operations; distinguish read-only simulation from persistent drafts. No automatic posting.
- MCP-06 — controlled writes: separately approved capability list, explicit user confirmation bound to actor, tenant, exact payload, resource version, expiry and idempotency key; revalidate on execution; use canonical API/domain services. Test replay, stale state, duplicate requests and failures. Posting, reversal, bank reconciliation, payment execution and tax submission excluded until individual risk/approval gates. Never modify posted entries or submitted XML directly.
- MCP-07 — production readiness: security review, privacy review, operational runbook, monitoring, kill switch, credential rotation, incident response and staged rollout. Separate deployment authorization. Read-only production pilot may be approved independently after MCP-04 and this gate; does not depend on write implementation.
- MCP-08 — optional developer diagnostics: separate identity/scopes and endpoint or service, read-only CI status, contract inventory and architecture lookup; reuse GitHub connector where possible. No shell, deployment, secret access or unrestricted repo mutation.

## 6. Acceptance tests
AC01 authorized user reads own tenant; AC02 foreign tenant and foreign entity IDs denied even with colliding IDs; AC03 missing tenant denied; AC04 expired/revoked/wrong audience token denied; AC05 role/scope enforcement on each tool; AC06 V1 rejects all writes and GET handlers verified side-effect-free; AC07 pagination and decimal/date/status fidelity; AC08 timeout/429/5xx handled without fabricated answers; AC09 no secrets or full sensitive payloads in logs; AC10 prompt injection in documents cannot trigger actions or change tenant; AC11 OpenAPI drift fails CI; AC12 ChatGPT end-to-end auth and tool calls on synthetic data; AC13 audit traceability and revocation; AC14 source references resolve only for authorized users. For later writes add confirmation/replay/idempotency/concurrency and rollback tests.
CI: unit tests, mocked HTTP contract tests, integration tests with two synthetic tenants, security checks, tool schema snapshots, contract drift gate. Reuse existing CI infrastructure; Django server tests only against isolated test DB with --keepdb where applicable. Record exact SHA, commands, results and blockers; no claim of deployment from push.

## 7. Example workflows
Find incoming Mercedes GLB invoice by supplier; retrieve its document and linked journal if canonical reference exists. Show partner open items and statement. Find bank transaction by supported filters, then show reconciliation candidates without matching. Explain PDV period and boxes using actual API evidence; distinguish facts from interpretation. Never claim completeness if filters/pages are incomplete.

## 8. Deliverables and governance
Proposed docs: ADR (next free number after checking register), MCP_TOOL_REGISTRY.md, MCP_SECURITY.md, MCP_TEST_PLAN.md, MCP_RUNBOOK.md, phase reports. Proposed code repo mcp; stack integration separately authorized. No changes to existing API unless documented gap and approved contract change. No implementation, DNS, deployment, repo creation, production access or secret changes authorized by this planning document.
Next decision: approve MCP-01 scope and separate repo proposal; then prepare ADR and exact implementation inventory. Current limitations: OpenAPI inspected statically; runtime, auth, ChatGPT entitlement and deployment not verified.

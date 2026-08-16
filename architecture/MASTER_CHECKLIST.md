# Master Checklist — racunAI ERP

Living dokument: funkcionalni moduli, NFR i compliance. Ažurirati pri zatvaranju milestonea.

**Legenda:** `[x]` gotovo · `[~]` djelomično · `[ ]` nije započeto

North Star KPI: [`NORTH_STAR.md`](NORTH_STAR.md) · Roadmap: [`ROADMAP.md`](ROADMAP.md)

---

## Platform

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 1 | Auth (session, JWT API, Turnstile) | [~] | JWT + Turnstile na admin login; nema SSO/OAuth |
| 2 | RBAC (owner / accountant / viewer) | [x] | `tenants/permissions.py`, `TenantAdminMixin` |
| 3 | Celery (async + beat) | [x] | Bank sync, fiskalizacija outbox, scheduled tasks |
| 4 | MFA / 2FA | [ ] | Nema implementacije |
| 5 | Feature flags | [ ] | Planirano u `domains/platform/` |
| 6 | Notifications | [~] | `dashboard.Notification` model; nema push/email servisa |

---

## Core

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 7 | Tenants (multi-tenant, domains, invites) | [x] | `Tenant`, `TenantMembership`, `TenantInvitation` |
| 8 | Users | [~] | Django User + `UserProfile`; nema self-service UI |
| 9 | Company settings | [x] | `CompanySettings`, `TaxRate`, `TaxOffice`, `ResponsiblePerson` |

---

## Finance

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 10 | GL / Chart of accounts (RRIF) | [x] | RRIF template + tenant `ChartOfAccounts` |
| 11 | Journal entries & auto-posting | [x] | `JournalEntry`, `PostingRule`, `post()`/`reverse()` |
| 12 | Balance sheet / RDG / trial balance | [x] | XLSX export, testovi |
| 13 | Saldakonti (otvorene stavke, aging) | [x] | `SubledgerItem`, posting hooks, `domains/finance` facade |
| 14 | Početno stanje import | [ ] | Odgođeno — čeka računovodstvenu dokumentaciju (issue #1) |

---

## Tax

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 15 | PDV obrazac (v11.0 pipeline) | [x] | Frozen arch ADR-0007; CI regresija |
| 16 | PDV boxovi 610+ (EU, reverse charge, OSS) | [x] | 610–615, 207/307, 209/210, 204/214/215/308 — `PDV_MAPPING_VERSION = 7`; vidi [`ADR-0014`](ADR-0014-tax-domain-completion.md) |
| 17 | PDV-S | [~] | Aggregate/render/parse; ePorezna manual only |
| 18 | I-RA / U-RA (knjige PDV) | [~] | `generate_vat_ledger`; vidi [`FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md) |
| 19 | ZP | [~] | L2 Gen/Parse/Manual Sub + admin + `verify_zp_period` — [`ZP_ARCHITECTURE.md`](../tax/ZP_ARCHITECTURE.md); G2B v2 backlog |
| 20 | JOPPD | [ ] | Planned candidate v1.5 — potvrda nakon Sprint 3 gate; [`FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md) |
| 21 | Submission audit (SubmissionEvent) | [x] | ADR-0009 frozen |
| 22 | Tax FORM_REGISTRY (SSOT) | [x] | [`docs/tax/FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md) |
| 23 | ePorezna Submission Matrix (docs SSOT) | [x] | plan v3 — [`EPOREZNA_SUBMISSION_MATRIX.md`](../tax/EPOREZNA_SUBMISSION_MATRIX.md); 9ec6958 + 8d1b010 |

---

## Sales

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 20 | Invoices (računi, PDF, numeracija) | [x] | `Invoice`/`InvoiceItem`, auto-posting |
| 21 | eRačun outbound | [~] | DIRECT default; SUPER rollback via `USE_SUPER_ERACUN_FALLBACK` |
| 22 | Ponude / otpremnice | [ ] | Faza 2 |

---

## Purchasing

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 23 | Expenses (troškovi, F1 import) | [x] | Full lifecycle, PDF attachments |
| 24 | eRačun inbound (AS4 → expense) | [~] | AS4 push + admin timeline; prod cutover migracija 0008 |
| 25 | PO / likvidatura | [ ] | Faza 2 |
| 26 | OCR ulaznih računa | [ ] | Faza 2 (Purchasing AI) |

---

## Integration

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 27 | UBL (HR CIUS 2025) | [x] | Builder, parser, XSD/Schematron, XAdES |
| 28 | AS4 (Domibus) | [~] | Inbound API + prod DIRECT cutover (0008); PTS JIR otvoren |
| 29 | OTP PSD2 (AIS/PIS) | [~] | Sandbox E2E; produkcijski go-live otvoren |
| 30 | Fiskalizacija 2.0 / CIS | [~] | Demo + prod config; S003 cert blokator |
| 31 | SUPER eRačun (privremeno) | [~] | Deprecated; rollback flag; Celery → `integrations.tasks` |
| 32 | ePorezna G2B submission (implementacija) | [ ] | v1 = manual portal; G2B konektor v2 backlog — docs SSOT #23 [x]; matrica: [`EPOREZNA_SUBMISSION_MATRIX.md`](../tax/EPOREZNA_SUBMISSION_MATRIX.md) |

---

## Extended modules

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 33 | Reporting & dashboard | [~] | Bilanca/RDG [x]; dashboard scaffold |
| 34 | Assets (dugotrajna imovina) | [~] | Aktivacija, linearna amortizacija, `domains/assets` facade L3 |
| 35 | Banking & reconciliation | [~] | CAMT import, matching; OTP sandbox |
| 36 | Workflow (odobravanja) | [~] | Expense status; nema generičkog BPM |
| 37 | DMS | [~] | Expense attachments only |
| 38 | CRM | [ ] | Faza 2 |
| 39 | Inventory | [ ] | Faza 2 |
| 40 | HR / plaće | [ ] | Faza 3 |

---

## MDM (Master Data)

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 41 | Partneri | [x] | `Partner` + contacts + bank accounts |
| 42 | Supplier unifikacija | [x] | TD-001 — `Expense.supplier` → `Partner`; `expenses.Supplier` deprecated |
| 43 | Artikli / usluge | [ ] | Faza 2 |
| 44 | Kontni plan (RRIF) | [x] | Template + tenant override |
| 45 | Stope PDV-a | [x] | `TaxRate` u settings |
| 46 | Zaposlenici (MDM) | [ ] | Faza 3 |

---

## Compliance

| # | Modul | Status | Napomena |
|---|-------|--------|----------|
| 47 | PDV zakonska usklađenost | [x] | ADR-0007 pipeline, CI regresija |
| 48 | Fiskalizacija | [~] | CIS demo + prod seed; PTS JIR (S003) |
| 49 | eRačun zakonska usklađenost | [~] | UBL + DIRECT prod cutover; stability window |
| 50 | GDPR (retention, export, brisanje) | [ ] | Nema dedicated implementacije |
| 51 | Audit trail | [~] | `AuditLog`, `SubmissionEvent`, lifecycle audit |
| 52 | Digitalni potpisi (XAdES) | [x] | UBL signing implementiran |

---

## NFR (Non-Functional Requirements)

| # | Zahtjev | Status | Napomena |
|---|---------|--------|----------|
| 53 | Multi-tenant izolacija | [x] | `TenantMixin`, scoped manageri, middleware |
| 54 | API-first (OpenAPI) | [~] | Minimalan REST (auth + fiscal inbound) |
| 55 | Test coverage (unit + integration) | [~] | PDV + banking CI; nema global coverage gate |
| 56 | Observability (logging, health) | [~] | `/api/ready/`; nema Sentry/Prometheus |
| 57 | Security (HSTS, Turnstile, secrets) | [~] | Secure cookies; nema MFA |
| 58 | Performance (Celery, indexes) | [~] | PDV perf baseline; nema load testing |
| 59 | Backup & DR | [~] | Docker volumes; nema formalnog DR plana |
| 60 | CI/CD (PDV, PIS regression) | [x] | `pdv-ci.yml`, `pis-posting-ci.yml` |

---

## Sažetak

| Status | Broj |
|--------|-----:|
| [x] Gotovo | 19 |
| [~] Djelomično | 28 |
| [ ] Nije započeto | 14 |

**North Star KPI (trenutno):** ~25% — tvrtka može legalno voditi knjige, pratiti saldakonti, generirati ZP i prošireni PDV (EU/RC/OSS); opening balance, G2B predaja i operativni UI još nedostaju.

Ažurirano: **2026-07-10** (Sprint 3 zatvoren — ADR-0014)

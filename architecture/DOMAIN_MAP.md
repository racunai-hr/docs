# Domain Map — racunAI ERP

Developer view: kako je kod organiziran po domenama. Product view: [`CAPABILITY_MAP.md`](CAPABILITY_MAP.md). Ovisnosti: [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md).

ADR: [`ADR-0010-domain-architecture.md`](ADR-0010-domain-architecture.md).

---

## Module Maturity Model (L0–L5)

| Razina | Opis | Kriterij ulaska |
|--------|------|-----------------|
| **L0** | Ideja | Samo u ROADMAP/checklistu |
| **L1** | Stub | `domains/*/__init__.py` ili facade re-export |
| **L2** | Djelomično | Produkcijski kod, ali nedovršen DoD faze |
| **L3** | Produkcijski spremno | DoD trenutne faze ispunjen, testovi, docs |
| **L4** | Stabilno | 2+ produkcijska razdoblja bez P1 bugova |
| **L5** | Enterprise | Multi-tenant scale, HA, puni API, AI capability |

Maturity se ažurira pri zatvaranju milestonea ili kvartalnom reviewu.

---

## Domene

| Domena | Maturity | Facade / ciljna putanja | Django appovi (privremeno) | Import rules |
|--------|----------|-------------------------|----------------------------|--------------|
| **Platform** | L2 | `domains/platform/` | `accounts`, `tenants` (auth/RBAC dio) | Ne importira poslovne domene |
| **Core** | L3 | `domains/core/` | `tenants`, `settings` | Importira samo Platform |
| **Finance** | L3 | `domains/finance/` (+ `ai/`) | `accounting`, `payments` | Importira Core, MDM; ne Tax direktno (eventi) |
| **Tax** | L3 | `domains/tax/` (+ `ai/`) | `accounting/services/tax_forms/` | Importira Finance (VATLedger), Core; SSOT: [`docs/tax/FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md) |
| **Sales** | L2 | `domains/sales/` (+ `ai/`) | `invoices` | Importira Core, MDM, Integration |
| **Purchasing** | L2 | `domains/purchasing/` (+ `ai/`) | `expenses` | Importira Core, MDM, Integration |
| **Integration** | L3 | `domains/integration/` | `integrations`, `fiscal_gateway`, `super_integration`, `ubl` | Importira Core; ne Finance/Tax direktno |
| **Banking** | L3 | `domains/banking/` | `banking`, `payments` | Importira Core, Finance (eventi) |
| **Reporting** | L2 | `domains/reporting/` (+ `ai/`) | `accounting/services/reports.py`, `dashboard`, `domains/reporting/documents/` | Read-only: Finance, Tax, Sales, Purchasing, Banking, Integration, DMS |
| **Assets** | L3 | `domains/assets/` | `accounting` (FixedAsset) | Importira Finance, Purchasing |
| **Workflow** | L1 | `domains/workflow/` | `accounting/services/submission/` | Importira Core, Tax |
| **Compliance** | L2 | cross-cutting (ne app) | audit u `accounts`, `SubmissionEvent` | Cross-cutting — ne importira domene |
| **MDM** | L2 | cross-cutting (ne app) | `partners`, `settings` (TaxRate) | Importira samo Core |
| **DMS** | L0 | `domains/dms/` (stub) | — | Faza 2 |
| **CRM** | L0 | `domains/crm/` (stub) | `partners` (djelomično) | Faza 2 |
| **Inventory** | L0 | `domains/inventory/` (stub) | — | Faza 2 |
| **HR** | L0 | `domains/hr/` (stub) | — | Faza 3 |

---

## Početno stanje (2026-07-09)

```
Platform ....... L2    (multi-tenant, auth, Celery — bez MFA)
Core ........... L3    (tenanti, postavke, korisnici)
Finance ........ L3    (GL, temeljnice, Bilanca/RDG, saldakonti)
Tax ............ L3    (PDV, PDV-S; VAT ledger L2; I-RA/U-RA kontrolni pregledi — ADR-0019; vidi docs/tax/FORM_REGISTRY.md)
Sales .......... L2    (računi, PDF, eRačun — bez ponuda/otpremnica)
Purchasing ..... L2    (troškovi, F1, inbound — bez PO/likvidature)
Integration .... L3    (UBL, AS4, MPS, OTP — fiskalizacija u tijeku)
Reporting ...... L2    (Bilanca/RDG export — bez GFI/dashboarda)
Assets ......... L3    (FixedAsset, aktivacija, linearna amortizacija, Celery)
Banking ........ L3    (OTP AIS/PIS, izvodi — frozen v2)
Compliance ..... L2    (PDV/fiskalizacija djelomično — bez punog GDPR)
MDM ............ L2    (partneri, stope — TD-001 unified Partner)
Workflow ....... L1    (submission + outbox — bez generičkog BPM)
DMS ............ L0
CRM ............ L0
Inventory ...... L0
HR ............. L0
```

---

## Ciljna struktura koda

```
erp/erp/app/
├── domains/
│   ├── platform/             # Auth facade, scheduler, feature flags
│   ├── core/                 # Tenant, org, company settings
│   ├── finance/ (+ ai/)
│   ├── tax/ (+ ai/)          # ledger, forms, submission facade, validation, verification
│   ├── sales/ (+ ai/)
│   ├── purchasing/ (+ ai/)
│   ├── assets/
│   ├── banking/
│   ├── reporting/ (+ ai/)
│   ├── integration/
│   ├── workflow/             # Stub Faza 2
│   ├── dms/                  # Stub Faza 2
│   ├── crm/                  # Stub Faza 2
│   ├── inventory/            # Stub Faza 2
│   └── hr/                   # Stub Faza 3
├── shared/                     # Pure helpers
├── events/                     # Cross-cutting event bus
└── [postojeći Django appovi]   # Persistence layer (privremeno)
```

**Pravilo:** Django modeli = persistence. Business logic = `domains/*/services/`. Views/Admin = thin.

---

## Import pravila (sažetak)

1. **Platform** ne importira poslovne domene
2. **Core** importira samo Platform
3. **Poslovne domene** importiraju Core + MDM; međusobno samo preko eventa ili shared helpers
4. **Compliance** je cross-cutting — audit hookovi, ne direktni importi
5. **Reporting** smije **čitati** Finance, Tax, Sales, Purchasing, Banking, Integration, DMS (ADR-0020)
6. **Zabranjeno:** kružne ovisnosti, Tax → Sales, Reporting **write** u Finance (obrnuti write tok)

Detalji: [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md).

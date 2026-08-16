# ADR-0010 — Domain Architecture

```text
Status: Accepted
Date: 2026-07-09
Type: Domain
Supersedes: —
Related: ADR-0011-architecture-freeze-v1.0.md, ADR-0012-sprint-1-retrospective.md, DOMAIN_MAP.md, CAPABILITY_MAP.md, DOMAIN_DEPENDENCY_MAP.md
```

## Status

**Accepted** — domenska arhitektura racunAI ERP-a: 14+ domena, Platform/Core razdvajanje, AI kao capability, Compliance/MDM kao cross-cutting odgovornosti.

## Context

ERP je rastao kroz Django appove (`accounting`, `invoices`, `expenses`, …) bez jasne domenske granice. `accounting` postaje „god-app" (finance + tax + assets + reporting). Potrebna je ciljna struktura koja:

- odvaja tehničku SaaS jezgru od poslovnog core-a,
- sprječava kružne ovisnosti između domena,
- omogućuje inkrementalnu migraciju bez big-bang splita,
- definira gdje AI živi (ne kao zasebna domena).

## Decision

### 1. Platform ≠ Core

| Sloj | Odgovornost | Primjeri |
|------|-------------|----------|
| **Platform** | Tehnička jezgra SaaS-a | Auth, RBAC, Scheduler, Celery, Feature Flags, Notifications, Audit infra, Settings infra, Licensing |
| **Core** | Poslovni core | Multi-tenant model, organizacije, korisnici, postavke tvrtke |
| **Compliance** | Cross-cutting odgovornost (ne app) | GDPR, Zakon o računovodstvu, OPZ, Fiskalizacija, eRačun, PDV, revizija, arhiviranje, digitalni potpisi |
| **MDM** | Cross-cutting (ne app) | Centralni šifrarnici: partneri, artikli, konta, stope, valute, OJ, skladišta, zaposlenici |

### 2. Poslovne domene

| Domena | Odgovornost | Trenutni Django appovi |
|--------|-------------|------------------------|
| **Finance** | GL, temeljnice, knjiženje, saldakonti, izvještaji | `accounting`, `payments` |
| **Tax** | PDV, PDV-S, JOPPD (buduće), porezni obrasci | `accounting/services/tax_forms/` |
| **Sales** | Računi, ponude (buduće), eRačun outbound | `invoices` |
| **Purchasing** | Troškovi, ulazni računi, likvidatura (buduće) | `expenses` |
| **Integration** | Connectori, UBL, AS4, fiskalizacija hub | `integrations`, `fiscal_gateway`, `super_integration`, `ubl` |
| **Banking** | Bankovni izvodi, PSD2, usklađivanje | `banking`, `payments` |
| **Reporting** | Bilanca, RDG, dashboard, BI (buduće) | `accounting/services/reports.py`, `dashboard` |
| **Assets** | Dugotrajna imovina, amortizacija | `accounting` (FixedAsset) |
| **Workflow** | Odobravanja, submission outbox | `accounting/services/submission/` |
| **DMS** | Dokumenti, arhiva (Faza 2) | — (stub) |
| **CRM** | Kupci, prodajni pipeline (Faza 2) | `partners` (djelomično) |
| **Inventory** | Zalihe, skladišta (Faza 2) | — (stub) |
| **HR** | Plaće, JOPPD, zaposlenici (Faza 3) | — (stub) |

### 3. AI nije domena — AI je capability

Umjesto `domains/ai/`, AI se implementira po domenama:

```
domains/tax/ai/          — auto-PDV pravila, klasifikacija
domains/finance/ai/      — auto-kontiranje, provjera knjiženja
domains/purchasing/ai/   — OCR ulaznih računa
domains/sales/ai/        — prepoznavanje partnera
domains/reporting/ai/    — AI asistent za računovođe
```

U dokumentaciji: **Capability Map** ima stupac `AI capability` po modulu.

### 4. Dvostruka mapa: Domain Map + Capability Map

- **Domain Map** — kako je kod organiziran (developer view): [`DOMAIN_MAP.md`](DOMAIN_MAP.md)
- **Capability Map** — što kupac vidi (product view): [`CAPABILITY_MAP.md`](CAPABILITY_MAP.md)

### 5. Ciljna struktura koda

```
erp/erp/app/
├── domains/                    # Business logic (servisi, use-cases, facades)
│   ├── platform/
│   ├── core/
│   ├── finance/ (+ ai/)
│   ├── tax/ (+ ai/)
│   ├── sales/ (+ ai/)
│   ├── purchasing/ (+ ai/)
│   ├── assets/
│   ├── banking/
│   ├── reporting/ (+ ai/)
│   ├── integration/
│   ├── workflow/
│   ├── dms/                    # Stub Faza 2
│   ├── crm/                    # Stub Faza 2
│   ├── inventory/              # Stub Faza 2
│   └── hr/                     # Stub Faza 3
├── shared/                     # Pure helpers
├── events/                     # Cross-cutting event bus
└── [postojeći Django appovi]   # Privremeno u rootu — persistence layer
```

**Pravilo:** Django modeli = persistence layer. Business logic = `domains/*/services/`. Views/Admin = thin.

Migracija je inkrementalna — postojeći appovi ostaju dok se logika ne ekstrahira u `domains/`.

### 6. Module Maturity Model (L0–L5)

Svaka domena ima razinu zrelosti definiranu u [`DOMAIN_MAP.md`](DOMAIN_MAP.md). Zamjenjuje nejasni status „partial".

### 7. Ovisnosti između domena

Dozvoljene ovisnosti definirane u [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md). **Zabranjeno:** kružne ovisnosti, Tax → Sales, Reporting → Finance (obrnuti tok).

## Consequences

### Prednosti

- Jasna granica Platform/Core/Compliance/MDM sprječava miješanje tehničke i poslovne logike
- AI capability po domeni omogućuje neovisni razvoj bez centralnog bottlenecka
- Inkrementalna migracija — nema big-bang splita Django appova
- Module Maturity daje mjerljiv napredak umjesto subjektivnog „partial"

### Rizici / trade-off

- Privremeno dupliciranje: logika u starim appovima i novim `domains/` paketima dok traje migracija
- `accounting` god-app ostaje dok se Finance/Tax/Assets ne ekstrahiraju (TD-002)
- `domains/` scaffolding mora postojati prije nego GitHub issuei referenciraju putanje

### Follow-up

- [x] Sprint 1: `domains/` + `shared/` facade re-exporti (scaffolding) — vidi ADR-0012
- [x] CI import lint (warning mode) — vidi ADR-0012
- [ ] Sprint 2–5: ekstrakcija logike iz god-appova u domene
- [ ] Ažurirati maturity u DOMAIN_MAP.md pri zatvaranju milestonea

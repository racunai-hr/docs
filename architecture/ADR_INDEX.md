# ADR Index — racunAI ERP

Katalog arhitekturnih odluka. Governance: [`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md).

---

## Aktivni ADR-ovi

| ID | Naslov | Tip | Status | Datum | Supersedes |
|----|--------|-----|--------|-------|------------|
| [ADR-0007](ADR-0007-pdv-module.md) | PDV modul (Obrazac PDV v11.0) | Business | Accepted | 2026-07-05 | — |
| [ADR-0008](ADR-0008-submission-events.md) | Generic Submission Event Architecture | Compliance | Accepted (amended) | 2026-07-07 | — |
| [ADR-0009](ADR-0009-submission-module-v1.md) | Submission modul v1 | Domain | Accepted | 2026-07-07 | — |
| [ADR-0010](ADR-0010-domain-architecture.md) | Domain Architecture | Domain | Accepted | 2026-07-09 | — |
| [ADR-0011](ADR-0011-architecture-freeze-v1.0.md) | Architecture Freeze v1.0 | Governance | Accepted | 2026-07-09 | — |
| [ADR-0012](ADR-0012-sprint-1-retrospective.md) | Sprint 1 Retrospective | Governance | Accepted | 2026-07-09 | — |
| [ADR-0013](ADR-0013-finance-domain-stabilization.md) | Finance Domain Stabilization (Sprint 2) | Governance | Accepted | 2026-07-09 | — |
| [ADR-0014](ADR-0014-tax-domain-completion.md) | Tax Domain Completion (Sprint 3) | Governance | Accepted | 2026-07-10 | — |
| [ADR-0017](ADR-0017-fiscal-gateway-model-a.md) | Fiscal Gateway (Model A) | Integration | Accepted | 2026-08-16 | — |
| [ADR-0018](ADR-0018-django-eracun-traffic-migration.md) | Django eRačun Traffic Migration to Fiscal Gateway | Integration | Accepted | 2026-08-17 | — |
| [ADR-0019](ADR-0019-tax-classification-engine.md) | Jedinstveni porezni filter i PDV evidencijski sloj | Business | Proposed | 2026-08-17 | — |
| [ADR-0020](ADR-0020-document-read-model.md) | Jedinstveni read model ulaznih i izlaznih računa | Domain | Proposed | 2026-08-19 | — |
| [ADR-0021](ADR-0021-banking-operational-ui-api.md) | Banking Operational UI and API Contract | Domain | Proposed | 2026-08-19 | — |
| [ADR-0022](ADR-0022-partner-management-mdm-api.md) | Partner Management & MDM API v1 | Domain | Accepted | 2026-08-19 | — |
| [ADR-0023](ADR-0023-partner-tax-jurisdiction.md) | Partner Geographic Jurisdiction (HR / EU / NON_EU) | Domain | Accepted | 2026-08-19 | — |
| [ADR-0024](ADR-0024-deposit-kaucija.md) | Kaucija/Depozit (given deposit v1) | Domain | Accepted | 2026-08-20 | — |
| [ADR-0025](ADR-0025-bank-reconcile-open-item.md) | Explicit bank reconcile to open SubledgerItem | Domain | Accepted (Amended) | 2026-08-20 | — |
| [ADR-0026](ADR-0026-private-funds-claim.md) | Private funds / paid on behalf of company | Domain | Accepted | 2026-08-20 | — |

---

## Rezervirani ADR-ovi (Sprint retrospectives)

| ID | Naslov | Sprint | Status |
|----|--------|--------|--------|
| ADR-0015 | Assets & Fiscal Platform Production | Sprint 4 | Planned |
| ADR-0016+ | Sprint retrospectives + governance backlog | Sprint 5+ | Planned |

---

## Povezani arhitekturni dokumenti (nije ADR)

| Dokument | Opis |
|----------|------|
| [banking-v2.md](banking-v2.md) | Banking v2 arhitektura (frozen) |
| [submission-module-backlog.md](submission-module-backlog.md) | Submission backlog |
| [FISCAL_GATEWAY_CANONICAL_API.md](FISCAL_GATEWAY_CANONICAL_API.md) | Kanonski API v1 `racunai-api` ↔ `intermediary` (ADR-0017; cutover Django prometa: ADR-0018) |
| [FAZA3_SALDAKONTI_SPEC.md](FAZA3_SALDAKONTI_SPEC.md) | Faza 3 operativni Finance workflow — Documents-centric; Faza 3a Implemented (app `9d088df`); bez modula `/saldakonti` |

---

## Kvartalni review log

| Datum | Bilješka |
|-------|----------|
| 2026-07-09 | Architecture Freeze v1.0 — inicijalni P0 dokumenti kreirani |
| 2026-07-09 | Sprint 1 zatvoren — ADR-0012; scaffolding + import lint + PR template |
| 2026-07-09 | Sprint 2 zatvoren — ADR-0013; saldakonti, TD-001, North Star ~15% |
| 2026-07-10 | Sprint 3 zatvoren — ADR-0014; ZP lifecycle, PDV 610+/RC/OSS, North Star ~25% |
| 2026-08-16 | ADR-0017 Accepted — Fiscal Gateway (Model A); 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-16 | Kanonski API v1 — [`FISCAL_GATEWAY_CANONICAL_API.md`](FISCAL_GATEWAY_CANONICAL_API.md) |
| 2026-08-17 | ADR-0018 Proposed — Django eRačun traffic migration; ADR-0017 editorial follow-up (`SuperAdapter` odvojen od migracije prometa) |
| 2026-08-17 | ADR-0018 Accepted — cutover/vlasništvo/rollback; prvi Django outbound slice i dalje čeka zaseban impl. plan |
| 2026-08-17 | Kanonski API — outbound-provider odvojen od inbound-bindinga (ADR-0018 §2); Slice 0 intermediary čeka ovaj ugovor |
| 2026-08-17 | Kanonski API — verzionirani outbound-provider, document stamp, DISABLED, 409 ne rezervira send ključ |
| 2026-08-17 | ADR-0019 Proposed — jedinstveni porezni filter; I-RA/U-RA interni kontrolni pregledi; 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-19 | ADR-0020 Proposed — jedinstveni read model ulaznih/izlaznih računa; 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-19 | ADR-0021 Proposed — Banking Operational UI and API Contract; 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-19 | ADR-0022 Proposed — Partner Management & MDM API v1 (composite UI; MDM-only partners API); 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-19 | ADR-0022 Accepted — Finance HTTP putanje, default lista active, UI picker enforcement, `partner_iban_conflict`; implementacija zasebno |
| 2026-08-19 | ADR-0023 Proposed — Partner Geographic Jurisdiction (HR/EU/NON_EU); jurisdikcija ≠ PDV registracija; 0015/0016 ostaju rezervirani sprint retroi |
| 2026-08-19 | ADR-0023 Accepted — country_code SSOT, HR/EU/NON_EU, VAT ≠ jurisdikcija; implementacija uz GO |
| 2026-08-20 | ADR-0024 Accepted — Kaucija/Depozit (given v1); 1152/1020/1000; full return + reverse; 0015/0016 rezervirani |
| 2026-08-20 | ADR-0025 Amended — SaM 9.9k nije budući BankTx; ADR-0026 Accepted — PrivateFundsClaim; CoA 2309 + Partner; OIB Ante 11528564544 |
| 2026-08-22 | Living docs reconcile (Korak A) — North Star ~35%; FAZA3 spec rev. 2 PASS (Documents + Partner + Banking workflow) |
| 2026-08-22 | Faza 3a Implemented / Accepted — app `develop` @ `9d088df`; legacy URL, operativni subnav, banking CTA, reconcile deep-link; 62/62 testova; Faza 3b (aging) ostaje opcionalni backlog |
| 2026-10-09 | Sljedeći planirani review (Q4 2026) |

---

## ADR Template

Kopiraj za novi ADR. Numeracija: sljedeći slobodni broj u katalogu.

```markdown
# ADR-XXXX — [Kratki naslov]

```text
Status: Proposed | Accepted | Deprecated | Superseded
Date: YYYY-MM-DD
Type: Business | Domain | Integration | Infrastructure | Security | Compliance | Performance | Governance
Supersedes: ADR-XXXX (ili —)
Related: [linkovi]
```

## Status

**Proposed** — [jedna rečenica o trenutnom stanju odluke]

## Context

[Problem, ograničenja, zašto sada]

## Decision

[Ključne odluke — tablice i bulleti po potrebi]

## Consequences

### Prednosti
- …

### Rizici / trade-off
- …

### Follow-up
- [ ] …
```

### Pravila

1. **Status** mora biti ažuran pri mergeu
2. **Type** mora odgovarati Decision Matrix u [`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md)
3. Zamjena starog ADR-a: novi ADR s `Supersedes:` linkom; stari → `Superseded by ADR-XXXX`
4. Amendment na isti ADR: sekcija `> **Amendment (ADR-XXXX):**` na vrhu povezanog dokumenta
5. Sprint retrospective ADR-ovi: max 2 stranice, fokus na odluke i lekcije

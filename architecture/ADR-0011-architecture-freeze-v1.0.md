# ADR-0011 — Architecture Freeze v1.0

```text
Status: Accepted
Date: 2026-07-09
Type: Governance
Supersedes: —
Related: ADR-0010-domain-architecture.md, ADR-0012-sprint-1-retrospective.md, ADR_INDEX.md, ARCHITECTURE_GOVERNANCE.md
```

## Status

**Accepted** — dokumentacijska struktura od 34 dokumenta i domenska arhitektura iz ADR-0010 su zamrznute. Sljedeći korak je isporuka, ne dizajn.

## Context

Living architecture plan definira hijerarhiju od 34 dokumenata, 14+ domena, Module Maturity Model i governance pravila. Bez formalnog freeze-a postoji rizik od:

- neprekidnog proširivanja dokumentacije umjesto implementacije,
- ad-hoc dodavanja novih domena bez ADR-a,
- improvisational promjena strukture koja zbunjuje developere.

Planiranje je završeno 2026-07-09. Potrebno je formalizirati granice.

## Decision

### Zamrznuta dokumentacijska struktura (34 dokumenta)

```
racunAI ERP Architecture
│
├── NORTH_STAR.md                  ← P0, kompas projekta
├── ARCHITECTURE_PRINCIPLES.md     ← P0, tehnička pravila
├── ARCHITECTURE_GOVERNANCE.md     ← P0, tko/kada/kako odlučuje
├── ADR_INDEX.md
├── ADR-0007 … ADR-0011 …
│
├── Reference & Structure
│   ├── REFERENCE_ARCHITECTURE.md
│   ├── DOMAIN_MAP.md
│   ├── CAPABILITY_MAP.md
│   ├── DOMAIN_DEPENDENCY_MAP.md
│   └── DATA_ARCHITECTURE.md
│
├── Development Process (P1)
│   ├── FEATURE_LIFECYCLE.md
│   ├── CODING_RULES.md
│   ├── API_GUIDELINES.md
│   ├── QUALITY_GATES.md
│   ├── ARCHITECTURE_REVIEW_CHECKLIST.md
│   └── RELEASE_POLICY.md
│
├── Operations (P1)
│   ├── DEPLOYMENT.md
│   ├── SECURITY.md
│   ├── PERFORMANCE.md
│   ├── OBSERVABILITY.md
│   └── TEST_STRATEGY.md
│
├── Integrations & Data (P1/P2)
│   ├── CONNECTORS.md
│   ├── SHARED_LIBRARIES.md
│   ├── EVENTS.md
│   └── MIGRATION_STRATEGY.md
│
├── Product Management (P1/P2)
│   ├── MASTER_CHECKLIST.md
│   ├── ROADMAP.md
│   ├── TECHNICAL_DEBT_REGISTRY.md
│   ├── OWNERSHIP.md
│   ├── KPI.md
│   └── GLOSSARY.md
│
└── GitHub
    ├── Milestones (po domeni/capability)
    └── Issues (konkretan posao)
```

Svi dokumenti u `erp/docs/architecture/` osim GLOSSARY-a koji može biti u `erp/docs/`.

**P0 dokumenti** (Sprint 1, tjedan 1): kreirani odmah.
**P1/P2 dokumenti**: nastaju kad razvoj otkrije potrebu — ne unaprijed.

### Dopuštene izmjene bez ADR-a

- Ažuriranje living docs: checklist, maturity, events, connectors, tech debt, KPI
- Dodavanje novih eventa u [`EVENTS.md`](EVENTS.md) (uz Issue)
- Dodavanje novih servisa u postojećoj domeni
- Dodavanje helpera u `shared/`
- Sprint retrospective ADR-ovi (0012–0016+)

### Zahtijeva ADR

- Nova domena
- Novi Django app
- Nova vrsta dokumenta (amendment na ovaj freeze)
- Promjena governance pravila
- Promjena freeze verzije
- Promjena PDV/banking/submission jezgre (posebni ADR-ovi)

### Sljedeći freeze

**v2.0** samo ako:
- tim naraste (>5 developera), ili
- sustav se dijeli u mikroservise

→ novi ADR s obrazloženjem.

### Operativno pravilo

> **Vrijeme ulagati u implementaciju. Novi dokumenti nastaju samo kada razvoj otkrije stvarnu potrebu za novom arhitekturnom odlukom.**

## Consequences

### Prednosti

- Tim zna da je planiranje gotovo i fokus je na Fazu 1 isporuci
- Jasna granica između living doc ažuriranja i arhitekturnih odluka
- 34 dokumenta pokriva sve aspekte ozbiljnog ERP-a bez monolitnog doc-a od 20+ stranica

### Rizici / trade-off

- P1/P2 dokumenti ne postoje odmah — developeri moraju koristiti postojeći kod + P0 docs
- Freeze ne sprječava promjenu koda — samo strukturu dokumentacije i governance

### Follow-up

- [x] P0 dokumenti (Sprint 1)
- [x] `domains/` + `shared/` scaffolding — vidi ADR-0012
- [x] GitHub milestones + Sprint 2 issuei — vidi ADR-0012 follow-up
- [ ] Implementacija Faze 1 (Sprint 2–5)

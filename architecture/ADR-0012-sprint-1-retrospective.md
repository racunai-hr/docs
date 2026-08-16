# ADR-0012 — Sprint 1 Retrospective

```text
Status: Accepted
Date: 2026-07-09
Type: Governance
Supersedes: —
Related: ADR-0010-domain-architecture.md, ADR-0011-architecture-freeze-v1.0.md, ROADMAP.md, DOMAIN_MAP.md
```

## Status

**Accepted** — Sprint 1 (Arhitekturni temelj) zatvoren. Nova domenska struktura postoji u kodu i dokumentaciji; poslovna funkcionalnost nije promijenjena.

## Context

Sprint 1 je imao jedan cilj: **postaviti arhitekturni temelj** prije Faze 1 isporuke (Sprint 2–5). Plan je bio:

1. P0 living dokumenti + Architecture Freeze v1.0
2. `domains/` + `shared/` scaffolding s facade patternom
3. CI import lint (warning mode) + PR template s quality gates
4. Sprint retrospective ADR (ovaj dokument)

North Star KPI ostaje **~10%** — Sprint 1 ne povećava samostalnost korisnika, ali uklanja blockere za Sprint 2+ (jasne putanje, governance, lint).

## Sprint outcome

| Stavka | Deliverable | Status |
|--------|-------------|--------|
| P0 dokumenti | NORTH_STAR, GOVERNANCE, PRINCIPLES, REFERENCE/DATA/DOMAIN/CAPABILITY maps, EVENTS, MASTER_CHECKLIST, ROADMAP | ✅ |
| Arhitekturne odluke | ADR-0010 (Domain Architecture), ADR-0011 (Freeze v1.0) | ✅ |
| `domains/` scaffolding | 15 domena s `facade.py`, `MATURITY`, re-exporti ili stubovi | ✅ |
| `shared/` scaffolding | 10 paketa (`money`, `vat`, `oib`, `iban`, …) | ✅ |
| Import lint | `import_lint` scanner + CI workflow (warning mode) | ✅ |
| PR template | Quality gates (tests, lint, living docs, ADR, PDV gate) | ✅ |
| GitHub milestones + issues | Sprint 2 issuei s putanjama `domains/finance/`, `accounting/`, `partners/` — [#19–#25](https://github.com/avrcanio/racunai.hr/issues?q=is%3Aissue+Sprint+2) | ✅ |

**Rezultat:** Nova arhitektura postoji; korisnik ne vidi promjenu u produkciji.

## Key decisions

### 1. Freeze prije scaffoldinga

ADR-0011 (Freeze v1.0) prije masovnog kreiranja koda u `domains/`. Sprječava proširivanje dokumentacije tijekom implementacije scaffoldinga.

### 2. Facade = re-export, ne migracija

Produkcijske domene (Finance L3, Tax L3, …) delegiraju na postojeće servise u `accounting/`, `invoices/`, … Stub domene (DMS, CRM, Inventory, HR — L0) imaju samo `MATURITY` i prazan `__all__`. **Nema big-bang splita Django appova u Sprint 1.**

Primjer:

```python
# domains/finance/facade.py — re-export dok migracija ne završi
from accounting.services.posting import post_document, ...
MATURITY = 'L3'
```

### 3. Import lint — warning mode prvi kvartal

CI workflow (`domain-import-lint.yml`) pokreće `python -m import_lint --warn`. Legacy kod u `accounting/` i drugim appovima još krši [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md); lint educira, ne blokira. **Plan:** error mode nakon Sprint 3 kad Tax/Finance stabilizacija smanji broj postojećih kršenja.

### 4. Scaffolding prije GitHub issuea

Putanje `domains/finance/`, `domains/tax/`, `shared/money/` postoje prije otvaranja Sprint 2 issuea. Issuei referenciraju stvarne direktorije, ne „buduću strukturu".

### 5. Shared = pure helpers + selektivni re-export

- **Pure:** `shared/money/` (`quantize_money`) — nema ovisnosti o domenama
- **Re-export:** `shared/vat/` — delegira na `accounting.services.tax_forms.pdv.mapping` dok se ne ekstrahira

Pravilo: `shared/` ne smije uvoditi poslovna pravila; PDV pipeline ostaje u Tax domeni (ADR-0007 gate).

## Lessons learned

| Lekcija | Detalj |
|---------|--------|
| **Planiranje je gotovo** | 34-dokumentna hijerarhija i 14+ domena definirani u Freeze v1.0. Sljedeći sprintovi su isporuka, ne dizajn. |
| **Architecture follows the product** | Sprint 1 nije dodao feature jer nijedan korisnički blocker nije zahtijevao novu domenu — samo temelj za Sprint 2 (saldakonti). |
| **`accounting` god-app je očekivan** | Finance/Tax/Assets/Reporting još dijele `accounting/`. Inkrementalna ekstrakcija u `domains/*/services/` — ne hitna, ali dokumentirana u DOMAIN_MAP maturity. |
| **Warning lint je pragmatičan** | Stotine legacy importa; blocking CI bi zaustavio sve PR-ove. Warning + PR checklist daje postupnu disciplinu. |
| **Stub L0 domene su OK** | DMS/CRM/Inventory/HR ne troše implementacijski kapacitet, ali omogućuju konzistentne putanje u roadmapu i issueima. |
| **P1/P2 docs čekaju potrebu** | CODING_RULES, TEST_STRATEGY, CONNECTORS — nastaju kad Sprint 2+ otkrije praznine. P0 docs + postojeći kod su dovoljni za start. |

## Blockers removed / remaining

### Uklonjeno (Sprint 1)

- Nema jasne domenske strukture → DOMAIN_MAP + `domains/` postoje
- Nema governance pravila → ARCHITECTURE_GOVERNANCE + Freeze v1.0
- Nema import discipline → import_lint + CI
- Nema PR quality gates → pull_request_template.md

### Preostalo (Sprint 3+)

| Blocker | Sprint | Domena |
|---------|--------|--------|
| ~~Saldakonti (otvorene stavke, aging)~~ | Sprint 2 | Finance — ✅ ADR-0013 |
| Početno stanje import | TBD (issue #1) | Finance |
| ~~Dupli Supplier model (TD-001)~~ | Sprint 2 | MDM/Purchasing — ✅ |

## North Star KPI

| Metrika | Prije Sprint 1 | Nakon Sprint 1 |
|---------|:--------------:|:--------------:|
| Samostalno vođenje bez Excela/drugog ERP-a | ~10% | ~10% (nepromijenjeno) |

Sprint 1 je infrastrukturni — KPI raste od Sprint 2 (+~5% procjena: saldakonti + migracija + partneri).

## Consequences

### Prednosti

- Novi developer: NORTH_STAR → PRINCIPLES → DOMAIN_MAP → `domains/<domena>/facade.py`
- Sprint 2 issuei mogu odmah referencirati `domains/finance/` bez refaktora strukture
- Import lint + PR template ugrađuju arhitekturu u svakodnevni workflow

### Rizici / trade-off

- Privremeno dupliciranje API-ja (facade + legacy servisi)
- Warning lint ne sprječava nova kršenja u legacy appovima — discipline kroz review
- GitHub milestones kreirani (2026-07-09) — milestonei `Meta: Architecture`, `Finance`, `MDM`, `Core/Purchasing`; Sprint 2 issuei #19–#25

### Follow-up

- [x] GitHub milestones po domenama + Sprint 2 issuei (Sprint 1 završetak) — milestonei #2–#5; issuei [#19](https://github.com/avrcanio/racunai.hr/issues/19)–[#25](https://github.com/avrcanio/racunai.hr/issues/25)
- [x] Sprint 2: saldakonti, Partner unifikacija → [ADR-0013](ADR-0013-finance-domain-stabilization.md) (početno stanje odgođeno)
- [x] Ažurirati DOMAIN_MAP maturity pri zatvaranju Sprint 2 milestonea
- [ ] Razmotriti import lint error mode nakon Sprint 3 (Tax stabilizacija)

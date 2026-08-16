# Roadmap — racunAI ERP

Poslovni plan po fazama. North Star KPI: [`NORTH_STAR.md`](NORTH_STAR.md). Checklist: [`MASTER_CHECKLIST.md`](MASTER_CHECKLIST.md).

---

## North Star KPI

> **Koliko novih klijenata može prijeći na racunAI bez Excela ili drugog ERP-a?**


| Verzija       | Samostalno vođenje poslovanja | Cilj                                                 |
| ------------- | ----------------------------- | ---------------------------------------------------- |
| v1.0 (Faza 1) | 10%                           | Zakonska jezgra — knjige, PDV, eRačun                |
| v1.5          | 40%                           | + saldakonti, amortizacija, fiskalizacija produkcija |
| v2.0 (Faza 2) | 70%                           | Svakodnevno poslovanje bez admin hackova             |
| v3.0 (Faza 3) | 90%                           | Enterprise — plaće, proizvodnja, veći timovi         |


**Trenutno:** ~25% (Sprint 3 zatvoren) — vidi [`ADR-0014`](ADR-0014-tax-domain-completion.md).

Svaki sprint povećava ovaj postotak. Sprint retrospectives (ADR) bilježe uklonjene blockere.

---



## Faza 1 — Zakonska jezgra [Q3–Q4 2026]


|                  |                                                                                                          |
| ---------------- | -------------------------------------------------------------------------------------------------------- |
| **Goal**         | Tvrtka legalno vodi poslovne knjige u Hrvatskoj                                                          |
| **Scope**        | Platform, Core, Finance, Tax, Sales, Purchasing, Integration (min), Reporting (min), Compliance (min)    |
| **Out of scope** | Inventory, CRM, HR, plaće, proizvodnja, puni operativni UI                                               |
| **DoD**          | PDV predaja, I-RA/U-RA, temeljnice, Bilanca/RDG, eRačun produkcija, banke, saldakonti, fiskalizacija PTS |
| **Risks**        | Fiskalizacija cert, ePorezna G2B ovisnost, operativni UI                                                 |
| **Dependencies** | M1.6–M1.8, PDV stabilization gate                                                                        |
| **North Star**   | 10% → 40% (v1.5)                                                                                         |

Tax registry: [`docs/tax/FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md). **JOPPD:** planned candidate v1.5 — gate otvoren nakon Sprint 3 ([`ADR-0014`](ADR-0014-tax-domain-completion.md)).

**Sprint 3 zatvoren (2026-07-10):** ZP lifecycle, PDV 610+/reverse charge/OSS — [`ADR-0014`](ADR-0014-tax-domain-completion.md), [`SPRINT_3_ZP_XSD_GATE.md`](SPRINT_3_ZP_XSD_GATE.md).


### Sprintovi (Faza 1)


| Sprint   | Fokus                                               | North Star Δ | ADR                                           |
| -------- | --------------------------------------------------- | ------------ | --------------------------------------------- |
| Sprint 1 | Arhitekturni temelj (P0 docs, domains scaffolding)  | —            | ADR-0010, ADR-0011, ADR-0012                  |
| Sprint 2 | Saldakonti, početno stanje, Partner unifikacija     | +~5%         | ADR-0013 · [plan](SPRINT_2_IMPLEMENTATION.md) |
| Sprint 3 | ZP, PDV 610+, reverse charge, OSS                   | +~10%        | ADR-0014 ✅ · [gate](SPRINT_3_ZP_XSD_GATE.md)   |
| Sprint 4 | Amortizacija, M1.6–M1.8 fiskalizacija produkcija    | +~15%        | ADR-0015                                      |
| Sprint 5 | Workflow, DMS light, OCR, operativni frontend shell | —            | ADR-0016                                      |


---



## Faza 2 — Poslovni ERP [2027 H1]


|                  |                                                                               |
| ---------------- | ----------------------------------------------------------------------------- |
| **Goal**         | Svakodnevno poslovanje bez admin hackova                                      |
| **Scope**        | Inventory, CRM, Workflow, DMS, napredni Sales/Purchasing, Purchasing AI (OCR) |
| **Out of scope** | Plaće, JOPPD, proizvodnja                                                     |
| **DoD**          | Ponude, narudžbe, odobravanja, OCR ulaznih računa, operativni frontend        |
| **Risks**        | Scope creep, Partner/MDM bez rješenja                                         |
| **Dependencies** | Faza 1 DoD, MDM partnera/artikala                                             |
| **North Star**   | 70%                                                                           |


---



## Faza 3 — Enterprise [2027 H2+]


|                  |                                                                  |
| ---------------- | ---------------------------------------------------------------- |
| **Goal**         | Kompleksni poslovni modeli (proizvodnja, usluge, veći timovi)    |
| **Scope**        | HR, plaće, JOPPD, putni nalozi, proizvodnja, projekti, BI        |
| **Out of scope** | Multi-country, franšize                                          |
| **DoD**          | JOPPD predaja, platne liste, putni nalozi, proizvodnja normativi |
| **Risks**        | Regulatorna kompleksnost HR modula                               |
| **Dependencies** | Faza 2, MDM zaposlenici                                          |
| **North Star**   | 90%                                                              |


---



## Faza 4 — AI ERP [paralelno od Faze 1]


|                  |                                                                                          |
| ---------------- | ---------------------------------------------------------------------------------------- |
| **Goal**         | AI diferencijacija od konkurencije                                                       |
| **Scope**        | AI capabilities po domenama (ne zasebna domena)                                          |
| **Out of scope** | Generički chatbot bez poslovne vrijednosti                                               |
| **DoD**          | OCR, auto-kontiranje, auto-PDV, auto-zatvaranje izvoda, AI provjera knjiženja, asistenti |
| **Risks**        | AI halucinacije u financijama, regulatorni prihvat                                       |
| **Dependencies** | Stabilna jezgra (Faza 1), [`EVENTS.md`](EVENTS.md) katalog                               |


AI capabilities po domeni (ADR-0010):


| Capability                | Domena     | Faza |
| ------------------------- | ---------- | ---- |
| Auto-PDV pravila          | Tax        | 4    |
| Auto-kontiranje           | Finance    | 4    |
| OCR ulaznih računa        | Purchasing | 2–4  |
| Prepoznavanje partnera    | Sales      | 4    |
| AI asistent za računovođe | Reporting  | 4    |


---



## GitHub Milestones


| Milestone            | Faza | Tip                         |
| -------------------- | ---- | --------------------------- |
| `Meta: Architecture` | —    | Dokumentacija + scaffolding |
| `Platform`           | 1    | Tehnička jezgra             |
| `Core`               | 1    | Poslovni core               |
| `Compliance`         | 1    | Cross-cutting               |
| `MDM`                | 1–2  | Cross-cutting               |
| `Finance`            | 1    | Domena                      |
| `Tax`                | 1    | Domena                      |
| `Sales`              | 1    | Domena                      |
| `Purchasing`         | 1    | Domena                      |
| `Integration`        | 1    | Domena                      |
| `Reporting`          | 1    | Domena                      |
| `Assets`             | 1    | Domena                      |
| `Inventory`          | 2    | Domena                      |
| `CRM`                | 2    | Domena                      |
| `Workflow`           | 2    | Domena                      |
| `DMS`                | 2    | Domena                      |
| `HR`                 | 3    | Domena                      |
| `AI: Tax`            | 4    | Capability                  |
| `AI: Finance`        | 4    | Capability                  |
| `AI: Purchasing`     | 4    | Capability                  |
| `AI: Reporting`      | 4    | Capability                  |


---



## Status planiranja

**ZAVRŠENO — Architecture Freeze v1.0 (2026-07-09)**

Struktura se više ne mijenja. Sljedeći korak je isporuka, ne dizajn.

### Finalni redoslijed izvršavanja

```
1. P0 dokumenti          (Sprint 1) ✅
2. domains/ + shared/    (Sprint 1) ✅
3. GitHub milestones     (Sprint 1 završetak) ✅
4. Implementacija        (Sprint 2–5) ← Sprint 2 ✅ · Sprint 3 ✅ · Sprint 4 ← aktivno
```



### To-do

**Aktivno:** Sprint 4 — amortizacija, M1.6–M1.8 fiskalizacija produkcija → ADR-0015. Cursor plan: [`.cursor/plans/sprint_4_assets_fiskalizacija.plan.md`](../../.cursor/plans/sprint_4_assets_fiskalizacija.plan.md)

**Sprint 3 (završeno):** [`SPRINT_3_ZP_XSD_GATE.md`](SPRINT_3_ZP_XSD_GATE.md) · [`ADR-0014`](ADR-0014-tax-domain-completion.md) · Cursor plan (completed): [`.cursor/plans/sprint_3_zp_next.plan.md`](../../.cursor/plans/sprint_3_zp_next.plan.md)

**Sprint 2 (završeno):** [`.cursor/plans/sprint_2_implementation.plan.md`](../../.cursor/plans/sprint_2_implementation.plan.md) · [`SPRINT_2_IMPLEMENTATION.md`](SPRINT_2_IMPLEMENTATION.md)

#### Sprint 1 završetak

- [x] **Faza A** — GitHub milestones + Sprint 2 issuei ([#19–#25](https://github.com/avrcanio/racunai.hr/issues?q=is%3Aissue+Sprint+2))

#### Sprint 2 — Finance + MDM ✅

- [x] **B1** — TD-001 Partner/Supplier unifikacija
- [x] **B2–B3** — `SubledgerItem` model + posting hooks
- [x] **B4** — Partner aging izvještaj
- [x] **B5** — Admin UI
- [x] **B6** — `domains/finance/services/` facade + testovi
- [x] **Faza D** — ADR-0013 retrospective

**North Star nakon Sprint 2:** ~15% ([`ADR-0013`](ADR-0013-finance-domain-stabilization.md))

**Odgođeno (Sprint 2):** početno stanje import — čeka računovodstvenu dokumentaciju (GitHub issue #1).

#### Sprint 3 — Tax / ZP ✅

**Status:** **zatvoren** (2026-07-10). Retrospektiva: [`ADR-0014`](ADR-0014-tax-domain-completion.md).

| Faza | Zadatak | Status |
|------|---------|--------|
| **Gate A–E** | ZP XSD import + manifest + smoke test | ✅ |
| 1–5 | ZP pipeline + CI (12 `test_zp_*.py`) | ✅ |
| **Ops** | Admin UI, `verify_zp_period`, I-RA → 101/103 | ✅ |
| 6 | PDV 610+ → reverse charge → OSS (`PDV_MAPPING_VERSION = 7`) | ✅ |
| D | ADR-0014 retrospective | ✅ |

**Isporučeno (sažetak):**

| Domena | Deliverable |
|--------|-------------|
| ZP | `tax_forms/zp/` lifecycle, `ZPReturn`, XSD `v1-0/`, cross-check 101/103 |
| ZP ops | Django admin (draft, XML export, mark submitted), `verify_zp_period` |
| PDV | Boxovi 610–615 (VIII.1), RC (207/307, 209/210, 309), OSS/IOSS (204, 214, 215, 308) |
| CI | `pdv-ci.yml` — ZP + PDV EU/RC/OSS testovi |

**North Star nakon Sprint 3:** ~25% ([`ADR-0014`](ADR-0014-tax-domain-completion.md))

**Git:** ZP core commitan (`26784fa`); Sprint 3 completion (ops + PDV proširenja + ADR-0014) — **commit/PR pending** u working tree.

**Preostalo (backlog, izvan Sprint 3 scopea):**

- ePorezna G2B konektor — manual portal v1; strojna predaja v2
- Produkcijska regresija: `verify_zp_period` + `verify_pdv_period` na Fine Star EU razdoblju
- JOPPD scope procjena (v1.5 candidate)
- VIES cross-check hook (`verify.py` spreman, servis nije integriran)

#### Sprint 4 — Assets + Fiskalizacija (aktivno)

Cursor todos: [`.cursor/plans/sprint_4_assets_fiskalizacija.plan.md`](../../.cursor/plans/sprint_4_assets_fiskalizacija.plan.md)

| Faza | Zadatak | Status |
|------|---------|--------|
| **A1–A2** | Aktivacija OS (`can_activate`, `activate_fixed_asset`, admin) | [ ] |
| **A3–A5** | Plan amortizacije + JE posting + Celery beat | [ ] |
| **A6** | `domains/assets` facade + testovi + docs | [ ] |
| **B1–B2** | M1.6 UI cutover (outbound + inbound produkcija) | [x] |
| **B3** | M1.7 uklanjanje `super_integration` | [~] |
| **B4–B6** | M1.8 Track 0 + PTS runbook + produkcijski cutover | [~] |
| D | ADR-0015 retrospective | [ ] |

**North Star cilj nakon Sprint 4:** ~40% (v1.5 prag)

#### Sprint 5 (backlog)

- [ ] **Sprint 5** — Workflow, DMS light, OCR, operativni frontend shell → ADR-0016

Ažurirano: **2026-07-10**

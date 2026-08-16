# ADR-0014 — Tax Domain Completion (Sprint 3 Retrospective)

```text
Status: Accepted
Date: 2026-07-10
Type: Governance
Supersedes: —
Related: ADR-0013-finance-domain-stabilization.md, ADR-0007-pdv-module.md, ADR-0009-submission-module-v1.md,
          ROADMAP.md, SPRINT_3_ZP_XSD_GATE.md, ZP_ARCHITECTURE.md, pdv-extensions-roadmap.md
```

## Status

**Accepted** — Sprint 3 (Tax / ZP + PDV proširenja) zatvoren. ZP lifecycle kompletan (backend + ops); PDV boxovi 610+, reverse charge i OSS mapiranje isporučeni. ePorezna G2B predaja ostaje manual-only backlog.

## Context

Sprint 3 je imao pet ciljeva iz [`ROADMAP.md`](ROADMAP.md):

1. **ZP (Zbirna prijava)** — prvi novi obrazac po TaxFormEngine konvenciji, bez refaktora frozen PDV pipelinea
2. **ZP ops** — produkcijski ledger 101/103, `verify_zp_period`, Django admin za računovođe
3. **PDV boxovi 610+** — VIII.1 ispravak pretporeza (611–615) + agregat 610
4. **Reverse charge** — EU stjecanje, građevinski prijenos, B2B usluge (boxovi 207/307, 209/210, 309)
5. **OSS/IOSS** — e-trgovina EU (boxovi 204, 214, 215, 308)

**Infrastructure freeze** ([`docs/tax/README.md`](../tax/README.md)) održan — nema refaktora `SubmissionService`, PDV `build_pdv_payload` arhitekture ni `domains/tax/` strukture.

North Star KPI: **~15% → ~25%** (+~10% za EU izvoz, ZP, PDV proširenja).

## Sprint outcome

| Stavka | Deliverable | Status |
|--------|-------------|--------|
| Gate A–E | ZP XSD byte-identično u `accounting/schemas/zp/v1-0/` + smoke test | ✅ |
| ZP pipeline | aggregate → render/parse/validation → `ZPReturn` → `SubmissionService` | ✅ |
| ZP CI | 12 `test_zp_*.py` modula u `pdv-ci.yml` | ✅ |
| ZP ledger 101/103 | `generate_vat_ledger()` — EU outbound Invoice → I-RA | ✅ |
| ZP ops | `verify_zp_period`, Django admin (draft, XML export, mark submitted) | ✅ |
| PDV 610+ | Boxovi 610–615 u registry + `_MAPPING_RULES`; `_sync_viii1_box_610` | ✅ |
| Reverse charge | Boxovi 207/307, 209/210, 309/306 — JE + Expense izvori | ✅ |
| OSS/IOSS | Boxovi 204, 214, 215, 308 — `vat_procedure` na Invoice/Expense | ✅ |
| PDV mapping | `PDV_MAPPING_VERSION = 7`; `pdv-mapping.md` + snapshot testovi | ✅ |
| ePorezna G2B | Strojna predaja PDV/ZP/PDV-S | ⏸ manual portal (v2 backlog) |

**Rezultat:** Tvrtka s EU izlaznim isporukama može generirati ZP XML, uskladiti s PDV boxovima 101/103 i proširenim PDV mapiranjem; predaja ostaje ručna preko ePorezna portala.

## ZP lifecycle — ključne odluke

### 1. TaxFormEngine konvencija potvrđena na ZP-u

ZP je referentni obrazac za sve buduće forme (PDV-K, JOPPD). Layout: `fixtures/tax/zp/` → `tax_forms/zp/{payload,aggregate,build,render,parse,validation,verify,submit}.py`. PDV pipeline **nije diran** — ZP dijeli `VATLedgerEntry` kao izvor, ali ima zaseban `ZpPayload` i `ZPReturn`.

### 2. Agregacija iz VATLedger, ne iz Invoice ORM-a

`build_zp_payload(period)` i `build_pdv_payload(period)` koriste isti ledger gate — contract test `test_zp_pdv_cross_check` osigurava ZP Σ dobra/usluge = PDV box 101/103. Produkcijski put: `Invoice (EU)` → `generate_vat_ledger()` → box 101/103 → `aggregate_zp_rows()`.

### 3. SubmissionService bez dupliciranog audita

`mark_zp_submitted()` delegira na frozen `SubmissionService` (ADR-0009). `ZPReturn` implementira `SubmissionDocument` protocol — isti pattern kao `VATReturn` / `PDVSReturn`.

### 4. Ops parity s PDV-om

`verify_zp_period` po uzoru na `verify_pdv_period`: cross-check ZP ↔ PDV + opcionalni XML diff. Django admin: draft akcija, XML download, označi predano — uzorak `VATReturnAdmin`.

## PDV proširenja — ključne odluke

### 5. Boxovi dodani u registry + ledger, ne u build sloj

610+, reverse charge i OSS slijede frozen gate iz ADR-0007: novi boxovi u `VAT_BOX_REGISTRY` + `_MAPPING_RULES`, proširenje `generate_vat_ledger()` u `vat.py`. `build_pdv_payload()` automatski uključuje sve `active` boxove — nema promjene arhitekture.

### 6. `PDV_MAPPING_VERSION` inkrementiran na 7

Svaka promjena koja utječe na generirani payload zahtijeva inkrement — snapshot testovi i `pdv-mapping.md` ažurirani.

### 7. OSS preko `vat_procedure`, ne zasebnog modela

Invoice/Expense polje `vat_procedure` (`eu_distance`, `eu_electronic`, `oss`, `ioss`) određuje mapiranje u boxove 204/214/215/308 bez novog domain modela.

## Gapovi — G2B i admin

| Gap | Status | Sprint | Napomena |
|-----|--------|--------|----------|
| ePorezna G2B konektor | ❌ backlog | v2 | Manual portal za PDV, ZP, PDV-S; matrica u [`EPOREZNA_SUBMISSION_MATRIX.md`](../tax/EPOREZNA_SUBMISSION_MATRIX.md) |
| ZP G2B predaja | ❌ backlog | v2 | `mark_zp_submitted()` + attachment potvrde — isti manual flow kao PDV |
| Export boxovi 102/104 | ❌ L1 | TBD | Treće zemlje — izvan Sprint 3 scopea |
| VIES cross-check | ❌ hook | TBD | `verify.py` spreman; vanjski servis nije integriran |
| Produkcijska ZP regresija | ⏳ | ops | Fine Star EU razdoblje — `verify_zp_period --xml` |
| JOPPD | L0 | v1.5 candidate | Gate otvoren nakon Sprint 3 — procjena opsega |

**Admin UI:** ZPReturn admin **isporučen** (Sprint 3 ops). G2B gap nije admin problem — transport sloj nedostaje.

## Lessons learned

| Lekcija | Detalj |
|---------|--------|
| **XSD gate prije koda** | Gate A–E spriječio rework namespacea/verzija; byte-identičan import obavezan. |
| **Fixtures prije implementacije** | 6 ZP scenarija + contract test ubrzali render/parse iteraciju. |
| **Cross-form testovi rano** | `test_zp_pdv_cross_check` hvata ledger drift prije produkcije. |
| **PDV proširenja bez refaktora** | 610+/RC/OSS dodani samo u registry + ledger — arhitekturni gate održan. |
| **Manual submission dovoljan za v1** | G2B konektor je integracijski projekt, ne blocker za generiranje i audit u ERP-u. |

## Blockers removed / remaining

### Uklonjeno (Sprint 3)

- Nema ZP obrasca → puni lifecycle L2 (Gen + Parse + Manual Sub)
- PDV boxovi 610+ neimplementirani → VIII.1 + EU/RC/OSS mapiranje u ledgeru
- EU izlazni računi ne mapiraju 101/103 → `generate_vat_ledger()` proširen
- Nema ZP admina / regresije → admin + `verify_zp_period`
- TaxFormEngine nepotvrđen → ZP kao referentna implementacija

### Preostalo (Sprint 4+)

| Blocker | Sprint | Domena |
|---------|--------|--------|
| Početno stanje import | TBD (issue #1) | Finance |
| Amortizacija, fiskalizacija produkcija | Sprint 4 | Assets / Integration |
| ePorezna G2B konektor | v2 | Integration |
| JOPPD | v1.5 candidate | Tax + HR |
| Operativni frontend | Sprint 5 | Workflow |

## North Star KPI

| Metrika | Prije Sprint 3 | Nakon Sprint 3 |
|---------|:--------------:|:--------------:|
| Samostalno vođenje bez Excela/drugog ERP-a | ~15% | **~25%** |

Dodano: EU izlazne isporuke (ZP + PDV 101/103), VIII.1 ispravak pretporeza, reverse charge, OSS/IOSS mapiranje, ZP admin workflow. Još nedostaje: opening balance, G2B predaja, amortizacija, operativni UI.

## Consequences

### Prednosti

- Tax L2+ maturity za ZP; PDV mapiranje prošireno bez arhitekturnog duga
- TaxFormEngine konvencija spremna za PDV-K i JOPPD
- JOPPD v1.5 gate otvoren — Sprint 3 kriteriji ispunjeni
- CI regresija pokriva ZP + PDV proširenja

### Rizici / trade-off

- Manual ePorezna predaja — korisnik mora uploadati XML ručno
- Produkcijska verifikacija ZP/PDV proširenja ovisi o Fine Star EU podacima
- `PDV_MAPPING_VERSION` 7 — postojeći arhivirani payloadi starijih verzija trebaju migration awareness
- OSS boxovi implementirani na mapiranju; puni OSS obrazac (P1) još L0 u FORM_REGISTRY

### Follow-up

- [ ] Produkcijska regresija: `verify_zp_period` + `verify_pdv_period` na Fine Star EU razdoblju
- [ ] JOPPD scope procjena (v1.5 candidate)
- [ ] Sprint 4: amortizacija, fiskalizacija produkcija → ADR-0015
- [ ] ePorezna G2B konektor — integracijski backlog (v2)

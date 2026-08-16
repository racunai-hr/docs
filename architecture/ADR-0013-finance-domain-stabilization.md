# ADR-0013 — Finance Domain Stabilization (Sprint 2 Retrospective)

```text
Status: Accepted
Date: 2026-07-09
Type: Governance
Supersedes: —
Related: ADR-0012-sprint-1-retrospective.md, ROADMAP.md, SPRINT_2_IMPLEMENTATION.md, DATA_ARCHITECTURE.md
```

## Status

**Accepted** — Sprint 2 (Finance + MDM) zatvoren. Saldakonti i TD-001 partner unifikacija isporučeni; početno stanje import odgođeno.

## Context

Sprint 2 je imao tri cilja iz [`ROADMAP.md`](ROADMAP.md):

1. **TD-001** — ukloniti dupli `expenses.Supplier` model; kanonski MDM je `partners.Partner`
2. **Saldakonti** — otvorene stavke potraživanja/obveza, aging, admin pregled
3. **Početno stanje import** — planirano, ali **out of scope** (nema računovodstvene dokumentacije, GitHub issue #1)

North Star KPI: **~10% → ~15%** (+~5% za saldakonti i unified counterparty).

## Sprint outcome

| Stavka | Deliverable | Status |
|--------|-------------|--------|
| B1 TD-001 | `Expense.supplier` → `Partner`; `AnalyticAccount` unified; `partner_resolver`; `expenses.Supplier` deprecated | ✅ |
| B2–B3 Saldakonti | `SubledgerItem`, `SubledgerAllocation`; posting hooks u `posting.py`; storno handler | ✅ |
| B4 Aging | `partner_aging`, bucketi (current, 1–30, 31–60, 61–90, 90+); XLSX export; `report_subledger_aging` | ✅ |
| B5 Admin UI | `admin_subledger.py` — list/filter, inline na Invoice/Expense, aging view | ✅ |
| B6 Facade + testovi | `domains/finance/services/{subledger,aging}.py`; facade re-export; lifecycle testovi | ✅ |
| Početno stanje | CSV/XLSX import, seed open items | ⏸ odgođeno |

**Rezultat:** Tvrtka može pratiti otvorena potraživanja i obveze po partneru; MDM nema dupli supplier model.

## Key decisions

### 1. `SubledgerItem` kao SSOT za open AR/AP

Otvoreni iznos i status (`open` / `partial` / `closed` / `cancelled`) žive u `SubledgerItem`. `Payment.related_invoice` ostaje legacy veza; alokacija ažurira saldakont preko `SubledgerAllocation`. Bank matching i postojeća analitička konta (1201/2201) ostaju; saldakont dodaje operativni sloj za aging.

### 2. Posting hooks u postojećem pipelineu

Hooks su u [`posting.py`](../../erp/app/accounting/services/posting.py), ne u Django signalima:

- `post_document('invoice_issued'|'expense_approved')` → `sync_subledger_for_document_posting`
- `post_invoice_payment` → `sync_subledger_for_invoice_payment`
- Storno temeljnice → `handle_journal_entry_reversal` (cancel/restore open item)

Poslovna logika ekstrahirana u `domains/finance/services/subledger.py`; model ostaje u `accounting/models.py` (nema split Django appa u Sprint 2).

### 3. TD-001 — Partner kanonski, Supplier deprecated

`Partner.partner_type` (customer/supplier/both) određuje analitičko konto (1201 vs 2201). Tablica `expenses_supplier` zadržana jedan sprint za rollback; novi kod koristi isključivo `partners.Partner`. Detalji: [`DATA_ARCHITECTURE.md`](DATA_ARCHITECTURE.md) § Partner, Expense.

### 4. Alokacije v1 = 1:1

Jedna uplata/isplata → jedna otvorena stavka (`SubledgerAllocation` unique per `journal_entry`). N:M alokacije i operativni frontend ostaju Sprint 5 (workflow).

### 5. Početno stanje eksplicitno odgođeno

Import opening balance ovisi o računovodstvenoj dokumentaciji. Preduvjeti (TD-001 + saldakonti) ispunjeni; implementacija ide u zaseban val nakon issue #1.

## Lessons learned

| Lekcija | Detalj |
|---------|--------|
| **TD-001 prije saldakonta** | Aging i open items per-counterparty zahtijevaju jedinstveni Partner model — redoslijed B1→B2 bio ispravan. |
| **Facade bez big-bang migracije** | `domains/finance/` re-exporta servise; `accounting/` ostaje persistence layer — konzistentno s ADR-0012 odlukom #2. |
| **Admin-first UI** | Django admin dovoljan za računovođe do Sprint 5 operativnog shella; izbjegnut scope creep (N:M, self-service). |
| **GFK izvor dokumenta** | `source_content_type` + `source_object_id` omogućuje Invoice, Expense i buduće Manual JE bez schema promjena. |

## Blockers removed / remaining

### Uklonjeno (Sprint 2)

- Dupli Supplier model (TD-001) → unified `Partner`
- Nema saldakonta / aginga → `SubledgerItem` + izvještaji
- Sprint 2 Finance issuei bez implementacije → [#19–#23](https://github.com/avrcanio/racunai.hr/issues?q=is%3Aissue+Sprint+2) isporučeni (osim odgođenog opening balance)

### Preostalo (Sprint 3+)

| Blocker | Sprint | Domena |
|---------|--------|--------|
| Početno stanje import | TBD (issue #1) | Finance |
| PDV boxovi 610+ (EU, reverse charge, OSS, ZP) | Sprint 3 | Tax |
| Amortizacija, fiskalizacija produkcija | Sprint 4 | Assets / Integration |
| Operativni frontend, N:M alokacije | Sprint 5 | Workflow / Finance |

## North Star KPI

| Metrika | Prije Sprint 2 | Nakon Sprint 2 |
|---------|:--------------:|:--------------:|
| Samostalno vođenje bez Excela/drugog ERP-a | ~10% | **~15%** |

Dodano: praćenje dospjelih potraživanja/obveza po partneru i unified counterparty za Sales/Purchasing/PDV-S. Još nedostaje: opening balance, PDV 610+, amortizacija, operativni UI.

## Consequences

### Prednosti

- Finance L3 maturity s saldakontom ([`DOMAIN_MAP.md`](DOMAIN_MAP.md))
- Sprint 3 (PDV 610+) može koristiti iste counterparty izvore bez TD-001 duga
- Testovi pokrivaju lifecycle: create → partial payment → closed → storno

### Rizici / trade-off

- `expenses.Supplier` tablica još postoji — ukloniti nakon stabilizacije u produkciji
- 1:1 alokacije ne pokrivaju složene uplate na više računa
- Opening balance bez importa — novi tenanti nemaju historijske open items

### Follow-up

- [ ] Početno stanje import kad stigne dokumentacija (issue #1)
- [ ] Hard drop `expenses.Supplier` nakon jednog produkcijskog ciklusa
- [x] Sprint 3: ZP + PDV 610+ → [`ADR-0014`](ADR-0014-tax-domain-completion.md)
- [ ] Razmotriti import lint error mode nakon Sprint 3 (Tax stabilizacija) — iz ADR-0012

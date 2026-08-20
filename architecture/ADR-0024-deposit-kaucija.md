# ADR-0024 — Kaucija/Depozit (given deposit v1)

```text
Status: Accepted
Date: 2026-08-20
Type: Domain
Supersedes: —
Related: ADR-0013-finance-domain-stabilization.md, ADR-0019-tax-classification-engine.md, ADR-0020-document-read-model.md, ADR-0021-banking-operational-ui-api.md, DATA_ARCHITECTURE.md, DOMAIN_MAP.md
```

## Status

**Accepted** — Finance dokument tipa **Kaucija/Depozit** (v1: **samo dana kaucija**), kontiranje RRIF **1152 / 1020 / 1000**, `SubledgerItem` SSOT, full return, reverse iz `open`, strogi idempotency/concurrency, bez FX/OCR/`related_expense`. Implementacija slijedi eksplicitni GO.

Ne superseda Invoice/Expense. Ne mijenja Banking v2 lifecycle. Ne uvodи N:M alokacije.

## 1. Context

Kaucije ne utječu na prihod/rashod; vode se kao potraživanje (dana) ili obveza (primljena). Saldakonti danas znaju samo Invoice (AR) i Expense (AP). Plaćanje kaucije gotovinom i povrat na žiro mora biti vidljivo u partner saldakontu bez lažnog Expensea.

## 2. Decision

Uvesti model `accounting.Deposit` (Finance ownership) s workflowom:

```text
draft → cancelled
draft → post → open
open → full return → returned
open → reverse → reversed
```

### Knjiženje (RRIF)

| Događaj | JE | Subledger |
|---------|----|-----------|
| given | Dr **1152** / Cr **1020** | open receivable |
| returned | Dr **1000** / Cr **1152** | allocate → closed |
| reversed (iz open) | Dr **1020** / Cr **1152** (storno given JE) | cancel open item |

v1: 1152 sintetički; partner SSOT na `SubledgerItem`.

### Invarijante v1

- Full return only: `return_amount == deposit.amount`
- Ista valuta; bez FX
- `return_bank_account` required na returnu; tenant match
- Tax: NOT_TAX_RELEVANT — 0 VAT ledger, 0 P&L
- Idempotency-Key + `select_for_update` + 1 JE po `(deposit, posting_kind)`
- Race return↔reverse: lock Deposit pa state check
- Cross-tenant: 404
- `unique(tenant, number)` concurrency-safe
- Nema `related_expense`

### Read model

`workflow_status` na Depositu; `operational_status` / `open_amount` iz `SubledgerItem`. Document read model uključuje `kind=deposit` (ili ekvivalent) u listi / partner projekciji.

## 3. Out of v1

Primljene kaucije (2147), reverse nakon returned, partial return, FX, 0652, OCR, Expense link, **auto** BankTransaction→return (CAMT/suggest/background), N:M, klasa 9, 1152-P analitika.

**Dopušteno (ADR-0025):** eksplicitna korisnička akcija `reconcile-open-item` koja poziva Finance `return_deposit` pa Banking match na return JE.

## 4. Acceptance

Vidi implementation plan acceptance matrix (parallel post/return, guards, VAT/P&L, SaM smoke).

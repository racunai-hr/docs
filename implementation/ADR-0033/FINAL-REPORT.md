# ADR-0033: završni izvještaj

Datum: 2026-11-27. ADR ostaje **Proposed**. Ništa nije deployano i produkcijski podaci nisu mijenjani.

## 1. Sažetak

Ulazni račun u stranoj valuti više se ne može proknjižiti, uvesti u saldakonto ili PDV knjigu, zatvoriti bankom ni prikazati kao EUR bez potvrđenog preračuna. Izgrađeni su centralni servis tečajeva, potvrda tečaja i valute obveze, odvojen porezni preračun, plaćanja s tečajnim razlikama, vrednovanje i EUR-primarni prikaz.

Dva dijela namjerno su zaključana do računovodstvene odluke: knjiženje tečajnih razlika (traži odobrena konta) i uporaba ranijeg tečaja za vikende i praznike (0 dana).

## 2. Isporuka

| Repozitorij | Grana | Commit | Pull request |
|-------------|-------|--------|--------------|
| `api` | `feat/adr-0033-incoming-fx` | `8c982e8ab3ed4f85ffb7de23c11e073a88136dee` | [api#18](https://github.com/racunai-hr/api/pull/18) (draft, u `develop`) |
| `app` | `feat/adr-0033-incoming-fx` | `ed56bb27b6b08209a5332e27d8f2997438fafbb0` | [app#8](https://github.com/racunai-hr/app/pull/8) (draft, u `develop`) |
| `docs` | `docs/adr-0033-implementation` | vidi pull request | draft, u `develop`; nosi i dva ADR-0033 commita koja su dosad bila samo na `main` |
| `stack` | | nije mijenjan | |

Redoslijed mergea: `api`, zatim `app`.

Testovi: [TEST-REPORT](TEST-REPORT.md). Odluke: [ACCOUNTING-REVIEW](ACCOUNTING-REVIEW.md). Dizajn i uvođenje: [IMPLEMENTATION-PLAN](IMPLEMENTATION-PLAN.md).

## 3. Promjene ponašanja

- Strani račun (osim HRK) ne može postati `approved` ni `paid` bez potvrde.
- Generiranje PDV knjige staje za cijelo razdoblje ako strani trošak koji traži redak nema porezni tečaj.
- Devizna obveza zatvara se samo evidentiranjem deviznog plaćanja. Opće bankovno usklađenje i „Platio partner” je odbijaju.
- Strane kaucije, potraživanja privatnih sredstava i predujmovi u knjigovodstvenoj valuti različitoj od EUR odbijaju se.
- HRK: neto je sada bruto minus PDV u EUR, pa obveza odgovara temeljnici do centa.
- KPI ne zbraja nepoznate EUR iznose i prikazuje koliko ih nedostaje.
- Stariji strani zapis prikazuje EUR samo kad saldakonto odgovara zabilježenom tečaju; inače „potreban pregled”.

## 4. Ograničenja

- Sučelje: plaćanje samo u EUR i bez vezanja bankovne stavke; storno plaćanja i vrednovanje samo kroz API; konta samo u Django adminu.
- Devizni predujmovi i tečajne razlike na deviznom računu nisu obuhvaćeni.
- EUR dokument sa stranom obvezom nije podržan.
- Strani službeni dokumenti više se ne mogu proknjižiti (nema potvrde tečaja za njih); takvih zapisa sada nema.
- Nije napravljen vizualni pregled u pregledniku.
- Dio postojećih testova ne izvršava se ni prije ni poslije (vidi TEST-REPORT §2).

## 5. Otvorena pitanja

Računovodstvena i porezna: Q1–Q9 u [ACCOUNTING-REVIEW](ACCOUNTING-REVIEW.md). Najhitnija za račun 547: valuta obveze (13.800 EUR ili 103.500 DKK), PDV tretman rabljenog vozila i porezni tečaj.

Governance:

- Architecture Review ADR-0033; nakon prihvaćanja dopuna ADR-0020 §3.
- Novi vanjski izvor (ECB, HNB) traži RFC i Issue; `CONNECTORS.md` na koji governance upućuje ne postoji.
- Struktura `implementation/ADR-0033/` je prema zadatku; dosadašnji jedini plan bio je `architecture/ADR-0029-implementation-plan.md`.

Operativno:

- Uvođenje uključuje migracije i korake iz IMPLEMENTATION-PLAN §5; nije izvršeno.

## 6. Račun 547

Pregledan samo za čitanje; nije mijenjan. Dokumentacija dobavljača navodi i 103.500 DKK i 13.800 EUR, pa valuta obveze nije utvrđena. Test s potvrđenim 7,50 daje 13.800,00 EUR. Detalji: ACCOUNTING-REVIEW §4.

## 7. Izmijenjene datoteke

### `api` (68)

Novo (26):

- `app/accounting/admin_fx.py`
- `app/accounting/fx_models.py`
- `app/accounting/management/commands/audit_fx_documents.py`
- `app/accounting/management/commands/ensure_fx_beat_schedule.py`
- `app/accounting/management/commands/fetch_fx_rates.py`
- `app/accounting/migrations/0045_adr0033_fx.py`
- `app/accounting/services/fx_conversion.py`
- `app/accounting/services/fx_rates/__init__.py`
- `app/accounting/services/fx_rates/ingest.py`
- `app/accounting/services/fx_rates/lookup.py`
- `app/accounting/services/fx_rates/providers.py`
- `app/accounting/tests/test_fx_migrations.py`
- `app/accounting/tests/test_fx_rates.py`
- `app/domains/finance/api/fx_schema.py`
- `app/domains/finance/api/fx_views.py`
- `app/domains/finance/services/fx_common.py`
- `app/domains/finance/services/fx_confirmation.py`
- `app/domains/finance/services/fx_settlement.py`
- `app/domains/finance/services/fx_state.py`
- `app/domains/finance/tests/fx_base.py`
- `app/domains/finance/tests/test_fx_acceptance.py`
- `app/domains/finance/tests/test_fx_confirmation_api.py`
- `app/domains/finance/tests/test_fx_settlement.py`
- `app/domains/finance/tests/test_fx_tax_and_legacy.py`
- `app/domains/reporting/documents/fx_view.py`
- `app/expenses/migrations/0028_adr0033_fx_confirmation.py`

Izmijenjeno (42):

- `app/accounting/admin.py`
- `app/accounting/management/commands/generate_vat_ledger.py`
- `app/accounting/models.py`
- `app/accounting/services/foreign_vat.py`
- `app/accounting/services/fx.py`
- `app/accounting/services/posting.py`
- `app/accounting/services/tax_shadow/adapters.py`
- `app/accounting/services/vat.py`
- `app/accounting/signals.py`
- `app/accounting/tasks.py`
- `app/config/openapi_contract.py`
- `app/config/settings.py`
- `app/domains/banking/write/expense_close.py`
- `app/domains/banking/write/reconcile.py`
- `app/domains/finance/api/accounting_review_schema.py`
- `app/domains/finance/api/schema.py`
- `app/domains/finance/api/urls.py`
- `app/domains/finance/services/accounting_review.py`
- `app/domains/finance/services/bank_settlement.py`
- `app/domains/finance/services/deposits.py`
- `app/domains/finance/services/expenses.py`
- `app/domains/finance/services/payable_documents.py`
- `app/domains/finance/services/private_funds.py`
- `app/domains/finance/services/subledger.py`
- `app/domains/finance/services/supplier_prepayments.py`
- `app/domains/finance/services/vehicle_dependent_cost_capitalization.py`
- `app/domains/finance/tests/test_accounting_review.py`
- `app/domains/finance/tests/test_vehicle_inventory_cost.py`
- `app/domains/reporting/api/schema.py`
- `app/domains/reporting/documents/assemble.py`
- `app/domains/reporting/documents/export.py`
- `app/domains/reporting/documents/filters.py`
- `app/domains/reporting/documents/incoming_detail.py`
- `app/domains/reporting/documents/projection.py`
- `app/domains/reporting/documents/relations.py`
- `app/domains/reporting/documents/service.py`
- `app/domains/reporting/tests/test_document_read_model.py`
- `app/domains/tax/classification/engine.py`
- `app/expenses/admin.py`
- `app/expenses/models.py`
- `app/fiscal_gateway/services/inbound_as4_actions.py`
- `openapi.yaml`

### `app` (21)

Novo (5):

- `src/components/documents/DualAmount.test.tsx`
- `src/components/documents/DualAmount.tsx`
- `src/components/documents/ExpenseFxPanel.test.tsx`
- `src/components/documents/ExpenseFxPanel.tsx`
- `src/lib/fx.ts`

Izmijenjeno (16):

- `openapi/openapi.yaml`
- `src/app/globals.css`
- `src/components/documents/AccountingReviewPanel.test.tsx`
- `src/components/documents/AccountingReviewPanel.tsx`
- `src/components/documents/CloseDraftDialog.tsx`
- `src/components/documents/DocumentDetailPanel.tsx`
- `src/components/documents/DocumentKpi.tsx`
- `src/components/documents/DocumentTable.tsx`
- `src/components/documents/IncomingExpenseDetail.test.tsx`
- `src/components/documents/IncomingExpenseDetail.tsx`
- `src/components/partners/PartnerDocumentsPanel.tsx`
- `src/components/purchasing/InvoiceReview.tsx`
- `src/lib/accountingReview.ts`
- `src/lib/documentLabels.ts`
- `src/lib/documents.ts`
- `src/lib/openapi/generated.ts`

### `docs`

`accounting/pdv-mapping.md`, `implementation/ADR-0033/IMPLEMENTATION-PLAN.md`, `ACCOUNTING-REVIEW.md`, `TEST-REPORT.md`, `FINAL-REPORT.md`.

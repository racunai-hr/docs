# ADR-0025 — Explicit bank reconcile to open SubledgerItem

```text
Status: Accepted (Amended)
Date: 2026-08-20
Amended: 2026-08-20 — partial Invoice/Expense; Deposit full-only; JE marker by BankTx; Expense paid on AP close
Type: Domain
Supersedes: —
Related: ADR-0013-finance-domain-stabilization.md, ADR-0021-banking-operational-ui-api.md, ADR-0024-deposit-kaucija.md, ADR-0026-private-funds-claim.md, DATA_ARCHITECTURE.md, DOMAIN_MAP.md
```

## Status

**Accepted (Amended)** — eksplicitna korisnička akcija `reconcile-open-item` povezuje `BankTransaction` s otvorenom `SubledgerItem` (Invoice AR / Expense AP / Deposit). Banking identificira cilj; Finance facade izvršava settlement; zatim Banking match na settlement JE. CAMT / suggest / background **ne** smiju zatvarati stavku niti raditi Deposit return.

Ne superseda PaymentOrder / PaymentExecution freeze. Ne uvodi N:M Reconciliation.

## 1. Context

Usklađivanje SPA je bilo read-only; match API veže samo Payment/JE bez zatvaranja saldakonta (ADR-0021). Deposit return je zaseban Finance write (ADR-0024) bez bank matcha. Operativno treba: bankovna uplata/isplata → odabir open stavke → jedna eksplicitna akcija koja knjiži, alocira i označi BankTx matched. Realni slučajevi (npr. SaM) zahtijevaju **djelomičnu** alokaciju na Invoice/Expense uz zadržani full Deposit return.

## 2. Decision

### Invariant

> Bank match identificira cilj; Finance use-case izvršava poslovnu radnju. Sam `BankTransaction` nikada automatski ne stvara Deposit return niti zatvara `SubledgerItem`.

### API

- `GET /api/banking/transactions/{id}/open-item-candidates/`
- `POST /api/banking/transactions/{id}/reconcile-open-item/` + `Idempotency-Key`
- Body: **samo** `{ "subledger_item_id": N }`
- `return_bank_account` / settlement bank account = deterministički iz `BankTransaction.statement.bank_account`

### Amount / direction

- Settlement amount = `abs(bank_tx.amount)` (pozitivan)
- **Nakon** `select_for_update` na `SubledgerItem`:
  - Deposit: `amount == open_amount == deposit.amount` (full-only)
  - Invoice / Expense: `0 < amount ≤ open_amount` (partial dopušten)
- credit → receivable; debit → payable
- Candidates: Deposit točan iznos; Invoice/Expense `open_amount >= bank_amount` (heuristika; autoritativna provjera je nakon locka)

### Settlement JE marker

Primarni dedupe ključ je BankTransaction:

```text
[bank_reconcile:btx-{bank_transaction_id}]
```

Source ID smije biti u ostatku opisa radi debuga. Replay istog BankTx ne stvara drugi JE; drugi BankTx na isti dokument stvara novi partial JE.

### Expense paid invariant

Ako settlement zatvori AP (`SubledgerItem.status = closed`), u **istoj** `transaction.atomic`:

> `Expense.status → paid`

Partial settlement ostavlja `Expense.status = approved`.

### Lock order (jedan `transaction.atomic`)

1. `BankTransaction`
2. `SubledgerItem`
3. source (`Deposit` / Invoice / Expense)

Finance ne re-locka obrnutim redoslijedom. Posting + allocation + JE match (+ Expense paid sync) commitaju ili rollbackaju zajedno.

### Domain boundary

Banking orchestration zove **Finance facade** (`settle_open_item_from_bank`). Ne importira `accounting.services.*` niti privatne `domains.finance.services.*`.

### Deposit

Eksplicitna akcija → `return_deposit` (Dr bank ledger / Cr 1152) → match return JE. Nije auto `BankTransaction→return`. Full amount only.

### Idempotency

| Case | Result |
|------|--------|
| same key + same tx + same item | replay, no new JE |
| same key + different body/item | 409 idempotency conflict |
| new key, tx already matched | 409 already reconciled |

### Unmatch

Briše samo bank FK. Ne stornira JE / allocation; ne otvara saldakonto.

## 3. Out of v1

Partial Deposit return, N:M, FX, auto-return, reverse-through-unmatch, received deposits.

## 4. Acceptance

- SaM kaucija full return (v1)
- SaM T-2026-0011: 33 000 → approve → AP open → BankTx #39 23 100 → AP partial 9 900, Expense approved
- **Generički:** kasniji BankTx za cijeli preostali `open_amount` → AP closed + Expense `paid` (ista txn)
- **SaM stvarno stanje:** nakon BankTx #39 ostaje `partial` 9 900; preostalih 9 900 **nije** Fine Star BankTx i **ne** zatvara se kroz ADR-0025 — vidi ADR-0026 (privatna sredstva)
- Race: AP 1000, dva BankTx po 600 paralelno → jedna alokacija 600, open 400; drugi 422/409
- Idempotency, rollback-before-match

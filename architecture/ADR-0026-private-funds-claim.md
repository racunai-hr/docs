# ADR-0026 — Private funds / paid on behalf of company

```text
Status: Accepted
Date: 2026-08-20
Type: Domain
Supersedes: —
Related: ADR-0024-deposit-kaucija.md, ADR-0025-bank-reconcile-open-item.md, ADR-0013-finance-domain-stabilization.md, DATA_ARCHITECTURE.md, DOMAIN_MAP.md
```

## Status

**Accepted** — Finance dokument `PrivateFundsClaim` bilježi da je fizička osoba (Partner) vlastitim sredstvima platila ili financirala obvezu društva. Nastaje obveza društva prema toj osobi na GL **2309** uz Partner analitiku. Ne širi ADR-0025 (bank reconcile).

## 1. Context

Fine Star / SaM: BankTx #39 zatvorio 23 100 od 33 000 AP; ostatak **9 900** platio je Ante privatno (nema Fine Star BankTx). Posebno: Ante je financirao **5 000** kauciju; SaM je vratio 5 000 na žiro Fine Stara, ali dug Fine Star→Ante ostaje. Ukupno Ante open **14 900** dok nema refundacije.

ADR-0025 pokriva samo `BankTransaction → open SubledgerItem`. Privatna sredstva su drugi poslovni događaj: osoba plati za društvo → zatvori vanjskog vjerovnika (ili korigira funding) → nastane obveza prema toj osobi.

## 2. Decision

### Identity

| Sloj | Uloga |
|------|--------|
| `Partner` | SSOT — tko je vjerovnik (`partner_type=other` za fizičku osobu koja nije klasični dobavljač) |
| `PrivateFundsClaim` | Zašto firma duguje (tip + iznos + veza na izvor) |
| `SubledgerItem(partner=…)` | Pojedinačna otvorena obveza |
| `ExpensePayer` | Legacy private settlement na Expense; **nije** SSOT. Isti OIB → link na Partner bez drugog identiteta |

Ante Vrcan: OIB **`11528564544`** (potvrđen s osobne; isti kao postojeći ExpensePayer seed).

### CoA

> **PrivateFundsClaim payable = sintetika `2309`**, uz obaveznu **Partner** analitiku (`2309-P{partner.pk}`).

`2309` = GL obveza po privatno financiranim izdacima. `SubledgerItem` = pojedinačna otvorena obveza. Ne defaultati na `2201` (dobavljački AP).

Legacy `ExpensePayer` → `2309-P{payer.pk}` ostaje za `expense_paid` private_*; claimovi koriste Partner analitiku.

### Claim tipovi (jedan model)

| `claim_type` | Semantika |
|--------------|-----------|
| `supplier_payment` | Privatno plaćena obveza dobavljaču (creditor swap) |
| `deposit_funding` | Privatno financirana kaucija |

### Creditor-swap invariant (`supplier_payment`)

> Jedan `PrivateFundsClaim` → **jedan posted JE** → allocation na vanjski AP **+** novi payable `SubledgerItem` prema Partneru.

Zabranjeno: samo mutirati saldakonte bez JE.

Primjer SaM 9 900:

```text
Dr  2201-P{SaM}     9.900
Cr  2309-P{Ante}    9.900
```

+ `allocate_payment` na Expense (SaM AP)  
+ `create_subledger_item` payable Ante na claim  
+ ako SaM AP `closed` → `Expense.status=paid` u istoj txn

### Deposit funding (`deposit_funding`)

Nije creditor-swap SaM Expense AP. Nakon given JE (Dr 1152 / Cr 1020) i eventualnog returna, ekonomski Ante je financirao društvo:

```text
Dr  1020            5.000   # poništi fiktivni odljev blagajne društva
Cr  2309-P{Ante}    5.000
```

+ novi Ante payable SubledgerItem; **ne** dira Expense T-2026-0011; Deposit može ostati `returned`.

### Bank refund (ADR-0025)

Povrat Anti s žira: debit BankTx → reconcile na Ante open claim stavku:

```text
Dr  2309-P{Ante} / Cr banka  + allocate na tu stavku
```

Ne dira SaM Expense ni Deposit.

### API (v1)

- `POST /api/finance/private-funds-claims/` + `Idempotency-Key` (create draft)
- `POST /api/finance/private-funds-claims/{id}/post/` + `Idempotency-Key` (post JE + subledger)
- `GET /api/finance/private-funds-claims/{id}/`

### Idempotency

Marker JE: `[private_funds:{claim_id}]`. Replay ne stvara drugi JE.

## 3. Out of v1

Novi sintetski konto; FX; N:M; auto CAMT; reverse-through-unmatch; UI knjiženje na partner kartici; blind migracija svih ExpensePayer zapisa.

## 4. Acceptance

| Stavka | Stanje |
|--------|--------|
| SaM T-2026-0011 AP | **0** |
| Expense T-2026-0011 | **paid** |
| SaM Deposit | **0 / returned** |
| Ante `supplier_payment` | **9 900** open |
| Ante `deposit_funding` | **5 000** open |
| Ante ukupno | **14 900** |

Refund test: BankTx 5 000 Anti zatvara **samo** `deposit_funding` stavku.

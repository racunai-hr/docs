# ADR-0030 — Cost Centers / Mjesta troška

```text
Status: Accepted
Date: 2026-08-30
Type: Domain
Supersedes: —
Related: ADR-0013-finance-domain-stabilization.md, ADR-0022-partner-management-mdm-api.md, ADR-0028-official-document.md, ADR-0029-official-document-posting.md, DATA_ARCHITECTURE.md, docs/accounting/mjesta-troska.md
```

## Status

**Accepted** — Faza 0 preflight je dokazao da `AnalyticAccount` nije generička dimenzija. Uveden je zaseban `CostCenter` i kanonski trag na `JournalEntryLine.cost_center`.

## 1. Context

Ugostiteljski tenant treba pratiti isplativost po lokaciji (restoran, kuhinja, vinoteka, sobe) i po objektu (pojedino vozilo). `AnalyticAccount` to ne može nositi: svaki red materijalizira `ChartOfAccounts` list pod 1201/2201/2309 i svi writeri zahtijevaju partnera ili platitelja.

## 2. Decision

1. Novi tenant šifarnik `CostCenter` **bez** `ChartOfAccounts` veze.
2. `JournalEntryLine.cost_center` je **kanonski računovodstveni trag** nakon knjiženja.
3. `Expense.cost_center`, `Invoice.cost_center`, `OfficialDocument.cost_center`, `Vehicle.cost_center`, `FixedAsset.cost_center` i `ExpenseCategory.default_cost_center` su **input za resolver** u trenutku knjiženja. `JournalEntryLine.cost_center` je **kanonski trag** — writeri ga ne čitaju ponovo s dokumenta. Naknadna promjena MT-a na dokumentu ne smije retroaktivno mijenjati već proknjižene stavke; `save()` na `Expense` / `OfficialDocument` / `Invoice` to odbija. Ispravak ide kroz storno/reposting.
4. MT smije završiti samo na RDG kontima (klase 4, 5, 7). Invariant je u `validate_cost_center_for_account` + `persist_journal_entry_line`; `clean()` je dodatna zaštita.
5. `AnalyticAccount` ostaje isključivo saldakonti.

## 3. Consequences

- `JournalEntry.reverse()` kopira `cost_center`.
- Bank reconcile / depoziti / private funds prolaze kroz persist s `cost_center=None`.
- `Expense` zaključava MT već nakon izlaska iz `draft`; `OfficialDocument` i `Invoice` nakon postojeće proknjižene JE.
- Prevaljivanje (npr. 70/30 s vozila na kuhinju) nije u v1.

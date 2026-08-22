# Manual Journal Entry + Bank Matching

ERP implementira **generičku infrastrukturu** za ručne temeljnice i njihovo usklađivanje s bankovnim transakcijama. Ovo nije modul za pojedinačne poslovne slučajeve (pozajmice, OTP korekcije, početna stanja) — to su primjeri uporabe istog frameworka.

Transakcije iz **2026.** knjiže se normalno odmah — banka se ne smije ostaviti neproknjižena. Ako nedostaje povijesni kontekst (npr. otvoreni račun iz 2025.), koristi se **privremeno knjiženje** na prijelazni konto (vidi dolje).

Import početne bilance iz prethodnog računovodstva planiran je kao zaseban korak — vidi GitHub issue #1 (*Opening Balance Import — pending accounting documentation*).

Vidi i: [reporting-storno.md](reporting-storno.md) za NET/AUDIT pogled u izvještajima.

Za **uvoz vozila** (T-Cross, Golf): [uvoz-vozila-knjizenje.md](uvoz-vozila-knjizenje.md).

Za **osnovna sredstva** (issue #6): [fixed-assets-architecture.md](fixed-assets-architecture.md).

---

## Kada koristiti

| Tok | Alat |
|-----|------|
| Izdani račun, trošak, naplata računa | Automatsko knjiženje (`post_document`, `post_invoice_payment`) |
| Početna stanja, ispravke, split knjiženja, pozajmice | **Ručna temeljnica** + bank match |
| Bankovne naknade (OTP i slično) | Vidi [Bankovne naknade](#bankovne-naknade-hybrid-vs-statement-only) |
| Naplata računa s Payment zapisom | Payment usklađenje (postojeći flow) |

---

## Bankovne naknade (hybrid vs statement-only)

Kad banka tereći račun za vođenje / platni promet, kanonski put ovisi o tome postoji li ulazni račun (npr. SUPER):

| Situacija | Kanonski put |
|-----------|--------------|
| Postoji ulazni / SUPER račun banke | Expense → kategorija bankovne usluge / konto troška platnog prometa (npr. RRiF **4650**) → `approve` → AP → bank `reconcile-open-item` |
| Postoji samo bankovni izvod | Ručna `bank_fee` JE → Dr trošak platnog prometa / Cr banka → `match` na bankovnu transakciju |

**Ne** knjižiti isti terećenje i kao direktni `bank_fee` JE **i** kao odobreni Expense (duplo). Ako je prvo korišten statement-only put, a kasnije stigne SUPER račun: unmatch → storno `bank_fee` JE → zatim hybrid put.

Konto troška je tenant konfiguracija (`ExpenseCategory.default_account` / `ChartOfAccounts`); RRiF **4650** je primjer, ne hardkod u poslovnoj logici — vidi [chart-of-accounts-rules.md](chart-of-accounts-rules.md).

### VAT napomena (domestic bank fee, PDV 0)

Za **domestic** bankovnu uslugu / naknadu s `tax_amount=0`, bez knjiženja pretporeza (npr. 1400) i bez posebnog `vat_procedure` mappinga:

- **ne** generira se umjetni U-RA / box **303** red
- classifier rezultat `EXPENSE_GENERIC_303_REMOVED` je **očekivan** (namjerna zaštita: 303 je pretporez 25%, ne oslobođena bankovna usluga)
- nema deductible VAT-a; classifier / PDV registry se zbog toga **ne** širi ad-hoc

Ostali `REVIEW_REQUIRED` kodovi nisu automatski „očekivani” — tretiraju se kao blocker dok se ne razjasne.

---

## Komponente

```
erp/app/accounting/services/manual_journal.py   # create_manual_journal_entry, JournalLineInput
erp/app/banking/reconciliation.py             # match / unmatch / suggest
erp/app/banking/ledger.py                       # resolve_bank_ledger_account
```

### `JournalLineInput`

Servis prihvaća proširena polja za buduću analitiku. U v1 se perzistiraju samo polja koja `JournalEntryLine` podržava (`account`, `analytic_account`, `description`, iznosi).

### `BankAccount.ledger_account`

Svaki bankovni račun (OTP, Revolut, devizni…) mapira se na konto knjige (npr. 1000, 1001). Ako nije postavljeno, koristi se fallback `1000`.

Usklađivanje validira stavku na **tom** kontu, ne hardkodirano 1000.

Vidi: [chart-of-accounts-rules.md](chart-of-accounts-rules.md) — konta su konfiguracija tenant-a; poslovna logika ne smije ovisiti o šifri konta.

---

## Workflow

1. **Kreiraj ručnu temeljnicu** (Django admin ili `create_manual_journal_entry`)
2. **Knjiži** temeljnicu (admin akcija ili `post=True`)
3. **Uskladi bankovnu transakciju** — postavi `matched_journal_entry` u adminu ili pozovi `match_transaction_to_journal_entry`
4. Za poništavanje: admin akcija „Poništi usklađenje” ili `unmatch_transaction`

### Storno temeljnice

Ako je temeljnica usklađena s bankovnom transakcijom, **storno je zabranjen** (`ValidationError`).

Redoslijed za računovođu:

1. Poništi usklađenje bankovne transakcije
2. Storniraj temeljnicu

---

## Split knjiženja (samo ilustracija)

Poslovni događaj može imati bruto iznos različit od iznosa na izvodu (npr. dogovor da dio ostane društvu). ERP ne implementira tu logiku — korisnik je unosi ručno:

| Duguje | Potražuje | Iznos |
|--------|-----------|------:|
| 2145 | | 2.000 |
| | 1000 (banka) | 1.980 |
| | 7792 (prihod) | 20 |

Bankovna transakcija (1.980) usklađuje se jer odgovara stavci na `ledger_account` bankovnog računa.

Ovo je **primjer u dokumentaciji**, ne hardkodirana poslovna pravila u kodu.

---

## Model usklađenja (v1)

Trenutno: **1 bankovna transakcija ↔ 1 temeljnica** (`BankTransaction.matched_journal_entry`).

Budući smjer (nije implementirano): poseban `Reconciliation` model za:

- više transakcija → jedna temeljnica
- jedna transakcija → više temeljnica
- djelomična usklađenja

---

## API reference

```python
from accounting.services.manual_journal import JournalLineInput, create_manual_journal_entry
from banking.reconciliation import (
    match_transaction_to_journal_entry,
    unmatch_transaction,
    suggest_journal_matches,
)

# Idempotentno: isti par → no-op; druga temeljnica → ValidationError
match_transaction_to_journal_entry(bank_tx, journal_entry, user)
unmatch_transaction(bank_tx, user)
```

---

## Privremena knjiženja (prijelazna / suspense)

Koristi se kad bankovni promet **mora biti evidentiran odmah**, ali ciljni konto još nije poznat (npr. uplata od kupca čiji računi iz 2025. još nisu u sustavu).

**Prijelazni konto nije hardkodiran u ERP-u.** Računovođa odabire odgovarajući konto iz kontnog plana tenant-a (RRiF). ERP ne pretpostavlja nikakvu fiksnu šifru.

### Korak 1 — odmah, uz bank match

| Duguje | Potražuje | Iznos |
|--------|-----------|------:|
| 1000 (banka) | *prijelazni konto* | iznos s izvoda |

Opis temeljnice: npr. `Privremeno knjiženje — uplata Dalmacija`.

### Korak 2 — razrješenje (nakon opening balance ili identifikacije)

| Duguje | Potražuje | Iznos |
|--------|-----------|------:|
| *prijelazni konto* | 1201 (analitika partnera) | iznos |

Referentni slučaj: uplata 8.000 EUR od Dalmacija Eko Projekt (21.05.2026.) — vidi GitHub issue za privremena knjiženja.

### `resolution_status` (planirano)

Odvojeno od `JournalEntry.status` (draft / posted / reversed):

| Vrijednost | Značenje |
|----------|----------|
| `OPEN` | Proknjiženo, ali još nije razriješeno na konačni konto |
| `RESOLVED` | Razriješeno reklasifikacijom; popuniti `resolved_at`, `resolved_by` |

Temeljnica može biti **posted** i istovremeno **OPEN**. Omogućuje izvještaj: *Privremena knjiženja koja još čekaju razrješenje*.

### UI terminologija

| Kontekst | Termin |
|----------|--------|
| Kod (interni enum) | `suspense` |
| ERP sučelje | **Privremeno knjiženje** ili **Privremena stavka** |
| Ne koristiti u UI | „suspense“ |

---

## Početna stanja

Import početne bilance iz prethodnog računovodstva **ne počinje** dok ne stigne dokumentacija (završni račun, bruto bilanca, glavna knjiga, salda partnera). Format importa definirat će se prema stvarno dostupnim podacima.

Tek nakon importa provjeravaju se salda konta (2145, 2200, 1200, …) i razrješavaju privremena knjiženja.

Revizijski trag (ugovor, e-mail) za specifične dogovore vodi se izvan ERP-a.

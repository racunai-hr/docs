# ADR-0029 — OfficialDocument posting profiles + posting lifecycle

```text
Status: Proposed
Date: 2026-08-24
Type: Domain
Supersedes: —
Related: ADR-0028-official-document.md, ADR-0013-finance-domain-stabilization.md, ADR-0020-document-read-model.md, ADR-0024-deposit-kaucija.md, DATA_ARCHITECTURE.md, docs/accounting/uvoz-vozila-knjizenje.md, DOCUMENT_POSTING_REMEDIATION_MATRIX.md
```

## Status

**Proposed** — čeka Architecture Review. Ovaj ADR **ne** mijenja Accepted ADR-0028. ADR-0028 ostaje v1 evidencija (dokument + PDF + opcionalni `JournalEntry.source` link). Cross-link iz 0028 na ovaj ADR ide **tek nakon** prihvaćanja.

Ne superseda Invoice, Expense ni Deposit. Ne širi `OfficialDocument.kind`. Ne širi `ExpenseCategory`. Ne uvodi drugi posting engine. Ne uvodi DMS.

Implementacija je **zaseban gate** i čeka eksplicitni GO nakon Accepta. **Knjiži u v1 ovog ADR-a radi A i B u istom koraku** — JE bez otvorene obveze nije uspješan posting.

## 1. Context

ADR-0028 uveo je `OfficialDocument` kao četvrti kanonski tip. Referentni slučaj je PPMV rješenje Carinske uprave **UP/I-410-22/26-09/49557** (Audi A8, 10.347,20 €). Dokument i PDF postoje; korisnik i dalje mora ručno napraviti temeljnicu.

To nije `Expense`. PPMV ovdje ulazi u nabavnu vrijednost vozila (`0373`), ne u rashod. `ExpenseCategory` to ne smije rješavati: kategorija troška ne smije zaključivati vrstu službene isprave.

Na `Expense` je ista granica već zaključana: `Expense.posting_profile` određuje način knjiženja; konto je posljedica profila. Isti zakon treba za službeni dokument, na vlastitom katalogu.

Korisnički model: dokument je radna površina; temeljnica je pozadinski GL dokaz. Akcija je **Knjiži**, ne „otvori temeljnicu”.

```text
Službeni dokument
  kind: tax_decision
  profil: PPMV – nabava vozila
           ↓
        [Knjiži]
           ↓
A  JournalEntry     D 0373 / K 2201
B  SubledgerItem    payable, open, partner = issuer
```

## 2. Decision

Uvesti **katalog posting profila** za `OfficialDocument` i **lifecycle event** knjiženja kroz postojeći `post_document` / `PostingRule` motor.

### 2.1 `kind` ≠ profil

Glavna odluka ovog ADR-a:

| Polje | Opisuje | Ne opisuje |
|-------|---------|------------|
| `OfficialDocument.kind` | što je **isprava** (`tax_decision` \| `other`) | ekonomski učinak, konta |
| `OfficialDocument.posting_profile` | što dokument **računovodstveno znači** | vrstu papira |

Profil se **ne** izvodi iz `kind`. `tax_decision` nije posting plan. `kind=other` nije default profil.

### 2.2 Katalog `OfficialDocumentPostingProfile`

Novi model u `accounting` (Finance ownership). Tenant-scoped. Unique `(tenant, code)`.

| Polje | Pravilo |
|-------|---------|
| `code` | stabilni strojni identitet |
| `name` | HR oznaka |
| `economic_effect` | `capitalize` \| `expense` \| `liability_only` — **jedan** učinak po profilu |
| `allowed_kinds` | podskup `OfficialDocument.kind` |
| `requires_fixed_asset` | uvjet za **Knjiži**, ne za registraciju |
| `is_active` | soft-disable |

Profil **ne nosi konta**. Dr/Cr ostaju isključivo na `PostingRule`. Drugi posting engine se ne uvodi.

Na `OfficialDocument`: nullable FK `posting_profile`. Registracija bez profila ostaje valjana (ADR-0028). **Knjiži zahtijeva profil.**

### 2.3 Seed v1 kataloga

Samo profili čiji je `economic_effect` zaključan:

| `code` | `allowed_kinds` | `economic_effect` | `requires_fixed_asset` |
|--------|-----------------|-------------------|------------------------|
| `ppmv_vehicle_acquisition` | `tax_decision` | `capitalize` | da |
| `administrative_fee` | `other` | `expense` | ne |

`tax_obligation` **nije** u v1 katalogu. `liability_only` i `expense` nisu dvije varijante istog profila. Drugi porezni dokument čeka vlastiti profil kad postoji stvarni slučaj.

Upravljačka pristojba, carinska davanja, komunalna naknada, sudska pristojba — **nisu** seed. Dolaze kao novi `code` kad postoji dokument.

### 2.4 Lifecycle event, ne katalog u `PostingRule`

`PostingRule` ostaje **event engine**. Novi `document_type` je događaj, usklađen s `invoice_issued` / `expense_approved`:

```text
official_document_posted
```

Ime prati trenutak knjiženja: dokument je već `registered`; akcija je Knjiži; status dokumenta i dalje **nema** `posted` (ADR-0028 — `posted` živi na `JournalEntry`).

`PostingRule.condition.posting_profile` bira plan unutar tog eventa. Primjer namjere (konto ostaje tenant konfiguracija, ne hardkod u ADR-u kao jedina istina; referentni RRIF za PPMV je **0373 / 2201**):

| Event | `condition.posting_profile` | Namjera GL |
|-------|-----------------------------|------------|
| `official_document_posted` | `ppmv_vehicle_acquisition` | D CIP (0373) / K AP (2201) |
| `official_document_posted` | `administrative_fee` | D rashod / K AP (2201) |

Nescope-ana `expense_approved` pravila **ne** smiju pucati na `OfficialDocument`.

Novi payment event (`official_document_paid`) **nije** potreban za B. Zatvaranje obveze ide postojećim Banking putom (`reconcile-open-item`, ADR-0025), kao kod Expense.

### 2.5 Capability A i B — oba obavezna, isti Knjiži

```text
A. Knjiži OfficialDocument  →  JournalEntry
B. Kreiraj obvezu           →  SubledgerItem (payable) + payment status iz saldakonta
```

Jedan `official_document_posted` radi **oba**. Nema slicea „samo A”. JE bez `SubledgerItem` je neuspješan posting (isti hook obrazac kao `expense_approved` → `sync_subledger_for_document_posting`, ADR-0013).

| Capability | Daje |
|------------|------|
| **A** | JE po `PostingRule`; `JournalEntry.source` → dokument; FixedAsset veza kad profil traži imovinu |
| **B** | `SubledgerItem` payable (`partner` = `issuer`, iznos = dokument, due = `due_date`); status `open` / `partial` / `closed` je SSOT za „nije plaćeno / djelomično / plaćeno” |

`related_fixed_asset` i dalje **nije** identitet dokumenta. Za `ppmv_vehicle_acquisition` jest **uvjet Knjiži**.

Read-model badge nakon Knjiži smije afirmirati „nije plaćeno” samo iz `SubledgerItem`. Bez stavke: `not_recorded`, ne `0`.

### 2.6 Pravila prioriteta (izbor profila)

Prvo koje pogodi, stane. Niže ne prepisuje više.

1. **Eksplicitni odabir korisnika** — jedini write na `OfficialDocument.posting_profile`.
2. **Već postoji `JournalEntry.source` na ovaj dokument** — ne nagađati; ponovni Knjiži vidi §2.7.
3. **Prijedlog UI-a (nije write):** `kind=tax_decision` **i** `related_fixed_asset` popunjen → `ppmv_vehicle_acquisition`.
4. **Prijedlog UI-a:** `kind=other` → `administrative_fee` smije biti ponuđen; nije implicitni write.
5. Inače prazno — Knjiži pada dok korisnik ne odabere profil.

Zabranjeno za izbor profila: iznos, PDF/OCR, `ExpenseCategory`, konto, hardkodirani OIB izdavatelja.

### 2.7 Idempotency (invariant)

```text
Jedan uspješan posting po OfficialDocument + posting eventu.
To znači jedna JE i jedan SubledgerItem.
Ponovni Knjiži ne smije proizvesti novu JE niti drugi open item.
```

`select_for_update`, unique guard i točan `posting_kind` ostaju implementacijska odluka uz postojeći `post_document`. ADR ne zaključava tehniku.

### 2.8 Read model i UI

`OfficialDocument.status` se **ne** širi s `posted` / `paid`. Knjiženje iz `JournalEntry`. Plaćanje iz `SubledgerItem` (+ bank match kao dokaz, postojeće pravilo).

UI na detalju (nakon Knjiži):

```text
PPMV – nabava vozila
10.347,20 EUR
✓ Dokument registriran
✓ Knjiženo
○ Nije plaćeno
[Otvori PDF]
```

## 3. Out of v1 ovog ADR-a

```text
širi OfficialDocument.kind
tax_obligation / customs / communal / court seed
konta na profilu
drugi posting engine
izvod profila iz kind
ExpenseCategory / Expense.posting_profile reuse
official_document_paid / novi payment event (zatvaranje = postojeći bank reconcile)
storno workflow
OfficialDocumentLine / komponente iznosa
partner-specific posting rules
OCR
DMS / multi-attachment
```

## 4. Consequences

### Prednosti

- Korisnik knjiži s dokumenta; GL zapis ostaje dokaz, ne radna površina
- PPMV ne postaje lažni Expense ni lažna `ExpenseCategory`
- `kind` ostaje taksonomija isprave; profil ostaje ekonomski učinak
- Jedan `PostingRule` motor; nema paralelnog plana na profilu
- Payment badge je dokaziv od prvog Knjiži, bez novog payment eventa

### Rizici / trade-off

- Knjiži je veći gate: mora proći i GL i AP, ili ništa
- `administrative_fee` je uski ventil za `kind=other`; ne smije postati kanta za sve
- Tenant mora imati `PostingRule` za `official_document_posted` + profil prije Knjiži
- Issuer mora biti valjan `Partner` jer je on AP counterparty (TD-001)

### Follow-up

- [ ] Architecture Review ovog ADR-a
- [ ] Nakon Accept: jedan cross-link u ADR-0028 Related / Follow-up — bez mijenjanja v1 granica
- [ ] Implementacijski GO: A+B u istom Knjiži
- [ ] Amendment `PostingRule.DOCUMENT_TYPES` + seed pravila po tenantu
- [ ] Living-doc ER tek uz implementaciju

## 5. Acceptance

Referent: PPMV **UP/I-410-22/26-09/49557**, Carinska, Audi A8, **10.347,20 €**, `related_fixed_asset` popunjen.

| Slučaj | Očekivanje |
|--------|------------|
| Registracija bez profila | valjana (ADR-0028) |
| Knjiži bez profila | odbijeno |
| Knjiži `ppmv_vehicle_acquisition` bez imovine | odbijeno |
| Knjiži PPMV s profilom + imovinom | jedna JE (D 0373 / K 2201) **i** jedan `SubledgerItem` payable, partner = Carinska, open 10.347,20 € |
| Ponovni Knjiži | ista JE, isti open item; nema druge JE ni drugog itema |
| `kind=tax_decision` sam | **ne** postavlja profil |
| Seed `tax_obligation` | ne postoji |
| Profil nosi Dr/Cr | polja ne postoje |
| `expense_approved` pravilo | ne knjiži OfficialDocument |
| Detalj nakon Knjiži, bez bank matcha | afirmativno „nije plaćeno” iz `SubledgerItem.open` |
| Bank reconcile na tu stavku (ADR-0025) | „plaćeno” / `closed` — bez `official_document_paid` |
| ADR-0028 v1 | netaknut do Accepta ovog ADR-a |

# ADR-0028 — Službeni ulazni dokument (`OfficialDocument`)

```text
Status: Accepted
Date: 2026-08-24
Type: Domain
Supersedes: —
Related: ADR-0010-domain-architecture.md, ADR-0011-architecture-freeze-v1.0.md, ADR-0013-finance-domain-stabilization.md, ADR-0020-document-read-model.md, ADR-0024-deposit-kaucija.md, DATA_ARCHITECTURE.md, DOMAIN_MAP.md, docs/accounting/uvoz-vozila-knjizenje.md
```

## Status

**Accepted** — Architecture Review 2026-08-24. Nova vrsta kanonskog poslovnog dokumenta i njezino uključivanje u document read model (ADR-0020) su zaključani. Ovaj ADR **ne** uvodi DMS infrastrukturu, generički attachment sloj niti vlastiti posting/payment workflow. Implementacija je **zaseban gate** i čeka eksplicitni GO.

Ne superseda Invoice, Expense ni Deposit. Ne mijenja Banking v2. Ne mijenja DMS maturitet (ostaje L0 / stub).

## 1. Context

Ulazni dokument u racunAI nije nužno račun dobavljača. Referentni slučaj je **porezno rješenje Carinske uprave** (PPMV) kojim nastaje obveza prema državnom proračunu za konkretno vozilo. PDF postoji; ručna temeljnica postoji; kartica `FixedAsset` postoji. Poslovni dokument koji ih povezuje **ne postoji** — broj rješenja i VIN žive u `JournalEntry.description` / `reference`.

`Expense` to ne smije biti: to je ulazni račun dobavljača s vlastitim posting profilom i PDV lifecycleom. `ExpenseCategory` to ne rješava. DMS je L0 i roadmap ga drži za Sprint 5 / Fazu 2.

Document read model (ADR-0020) već nije vezan samo za Invoice/Expense: ADR-0024 dodao je `Deposit` kao treći kanonski tip s vlastitim `direction`. Isti obrazac vrijedi ovdje — četvrti write model, četvrti UNION branch — bez nove poslovne domene i bez novog Django appa.

Governance: nova vrsta dokumenta zahtijeva ADR + Architecture Review ([`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md)). Novi Django app bi zahtijevao zaseban gate; zato v1 namjerno ostaje u `accounting`.

## 2. Decision

Uvesti `accounting.OfficialDocument` (Finance ownership) kao **četvrti kanonski write model** i uključiti ga u `/api/documents/` kao `direction=official`.

```text
Ulazni dokumenti
├── Račun dobavljača     → Expense          direction=incoming
├── Porezno rješenje     → OfficialDocument direction=official
└── (kasnije: carinsko, naknada, pristojba — kroz kind, ne kroz novi model)
```

PDF ostaje primarni poslovni dokument. Temeljnica je njegov računovodstveni rezultat, ne zamjena.

### 2.1 Model

| Polje | Pravilo |
|-------|---------|
| `tenant` | `TenantMixin`; svi upiti scoped |
| `kind` | `tax_decision` \| `other` |
| `issuer` | FK `partners.Partner`, `PROTECT` — kanonski MDM (TD-001) |
| `document_number` | broj izdavatelja (npr. `UP/I-410-22/26-09/49557`) |
| `reference` | slobodna referenca (VIN, poziv na broj) |
| `issue_date`, `due_date` | `DateField` — poslovni datumi |
| `amount` | `Decimal(15, 2)` |
| `currency` | `CharField(3)`, default `EUR` |
| `status` | `draft` \| `registered` \| `cancelled` |
| `original_file` | jedan `FileField` na dokumentu |
| `original_filename` | originalni naziv |
| `file_sha256`, `content_type`, `file_size` | obrazac `IncomingInvoiceImport` |
| `related_fixed_asset` | nullable FK `FixedAsset` — **nije dio identiteta** |
| `notes` | slobodan tekst (PPMV komponente u v1 idu ovdje) |
| unique | `(tenant, issuer, document_number)` |

Identitet dokumenta je **izdavatelj + broj izdavatelja**, ne interni serijski broj. v1 ne uvodi zasebnu tenant seriju.

`related_fixed_asset` je opcionalna domen-specifična poveznica (PPMV ↔ kartica vozila). Službeni dokument bez imovine je valjan.

Komponente iznosa (vrijednosna / ekološka PPMV) **nisu** strukturirani model u v1. Nema `OfficialDocumentLine` ni `JSONField components`.

### 2.2 Lifecycle

```text
draft → registered
draft → cancelled
registered → cancelled
```

Nema `posted`, `paid` ni `superseded`.

`posted` i `paid` bi duplicirali stanje koje već pripada `JournalEntry` i (u budućnosti) `SubledgerItem`. Zamjensko/ispravljeno rješenje se ne modelira unaprijed: kad dođe stvarni slučaj, odlučuje se treba li audit lanac `supersedes` ili je dovoljno povezivanje dokumenata.

### 2.3 Datoteka

Jedan `FileField` direktno na `OfficialDocument`. Validatori kao za expense/OCR privitke (PDF/JPG/PNG, 10 MB). Upload path:

```text
official_documents/{tenant_id}/{doc_id}/{safe_filename}
```

Nema kolekcije privitaka. Nema `dms.Attachment`. Nema novog Django appa. DMS ostaje L0.

### 2.4 Veza na knjiženje

v1 **povezuje** postojeću ručnu temeljnicu: `JournalEntry.source` GenericFK → `OfficialDocument` (ADR-0013, bez schema promjene).

v1 **ne** kreira temeljnicu, `SubledgerItem`, VAT ledger niti bank match. Dokument je `NOT_TAX_RELEVANT` u smislu PDV evidencije — 0 `VATLedgerEntry`.

### 2.5 Read model

`direction` ostaje **isključivo identitet**. Četvrta vrijednost:

```text
outgoing | incoming | deposit | official
```

`(direction, id)` je složeni ključ (ADR-0020 §10). Službeni dokument **ne** dijeli `incoming` s `Expense`.

Read-model `kind` za ovaj tip: `official`. To nije isto polje kao `OfficialDocument.kind` (`tax_decision` \| `other`).

Grupiranje za UI tab **Ulazni** ide kroz **set-valued** filter:

```text
direction=incoming,official
```

Jedna vrijednost i dalje znači točno jedan identitet. `direction=incoming` nikad ne znači grupu. Novi `group=` parametar se ne uvodi.

Lista: četvrti `UNION ALL` branch (`_official_values`), isti redoslijed (`document_date DESC, direction ASC, id DESC`). `document_date` = `issue_date`. `source_number` = `document_number`.

v1 **nema** vlastiti subnav tab „Službeni”. Dokument se pojavljuje pod **Svi** i **Ulazni**.

### 2.6 Derivirano operativno stanje

| Prikaz | Izvor | Ako zapisa nema |
|--------|-------|-----------------|
| Status dokumenta | `OfficialDocument.status` | 404 |
| Knjiženje | `JournalEntry` GFK na dokument | `not_posted` |
| Plaćeno | postojeće pravilo `bank_matched` + `posting.status == posted` (isti precedent kao uvoz vozila u `operational_incoming`) | `not_provable` / `not_recorded` |
| Otvorena obveza / aging | — | `not_recorded` — v1 ne kreira `SubledgerItem` |
| PDV evidencija | — | `not_tax_active` |
| Privitak | `original_file` | prazna lista / `missing_pdf_xml` |

Frontend **ne** smije `value: null` prikazati kao potvrdan status (ADR-0020 §4).

Svjesna posljedica: v1 badge pokriva *za knjiženje → knjiženo → plaćeno (uz bankovni dokaz)*, ali ne daje aging ni otvoreni saldo.

### 2.7 Kontrole (v1, restriktivno)

Assembler emitira **samo** kontrole koje proizlaze iz vlastitih podataka dokumenta:

- nedostaje PDF (`original_file` prazan ili blob nedostupan)
- nedostaje izdavatelj
- nedostaje `document_number` ili `issue_date`
- nedostaje ili nevaljan `amount`

v1 **ne** emitira accounting/aging kontrole: `issued_unposted`, `overdue_unpaid`, `vat_*`, `settlement_evidence_missing`, `paid_status_subledger_*`. Nema vlastiti AP lifecycle iz kojeg bi te kontrole bile dokazive.

### 2.8 API i UI (nakon implementacijskog GO)

Write: registracija (draft → registered uz PDF), cancel, povezivanje postojeće temeljnice na `JournalEntry.source`.

Read: postojeći `GET /api/documents/` + `GET /api/documents/official/{id}/` + JWT download PDF-a (bez javnog URL-a, ADR-0020 §12).

UI: tab Ulazni šalje `direction=incoming,official`; detalj `/t/{slug}/dokumenti/sluzbeni/{id}`; PDF preview; prikaz povezane temeljnice. PDF preview smije se reusedati i za postojeće Expense privitke — to je UI popravak, ne nova DMS odluka.

## 3. Out of v1

```text
generički dms.Attachment
novi DMS Django app
multi-attachment
auto-posting / post_document profil
SubledgerItem creation
payment lifecycle
storno workflow
superseded / supersedes lanac
OfficialDocumentLine / JSONField components
širi kind enum (customs_decision, levy, fee)
partner-specific posting rules
OCR
vlastiti subnav tab „Službeni”
tenant-wide interni brojčanik
```

Buduće auto-knjiženje **može** ići preko `JournalEntry.source` GFK bez schema promjene (ADR-0013). Konta, `posting_kind`, idempotency i concurrency **nisu** predmet ovog ADR-a.

## 4. Consequences

### Prednosti

- PPMV rješenje (i slične isprave) postoji kao pravi ulazni dokument, ne kao tekst u temeljnici
- Četvrti tip prati već izvedeni Deposit obrazac; nema duplog modela ni lažnog Expensea
- `direction` ostaje čist identitet; grupiranje je filter, ne semantika
- Status dokumenta ne duplicira Finance SSOT
- ADR ne vuče DMS ni posting odluku

### Rizici / trade-off

- Jedan `FileField` nije DMS; drugi službeni privitak (npr. uplatnica) čeka Sprint 5 ili zasebni ADR
- Bez `SubledgerItem` nema aginga ni otvorenog salda na dokumentu
- Set-valued `direction` mijenja filter ugovor liste; klijenti koji šalju samo `incoming` neće vidjeti službene dokumente dok UI ne pošalje skup
- `kind=other` je uski ventil — namjerno, da se kategorije ne izmišljaju unaprijed

### Follow-up

- [x] Architecture Review ovog ADR-a (APPROVE, 2026-08-24)
- [ ] Nakon GO: model, read-model UNION, API, UI (zaseban implementacijski plan)
- [ ] Upload path: dokument mora imati PK (ili stabilni UUID) **prije** spremanja `original_file`; `{doc_id}` ne smije ovisiti o prvom saveu s praznim id-em
- [ ] `registered` server-side zahtijeva valjani PDF; `missing_pdf` na registriranom dokumentu je integrity/storage greška, ne normalno stanje
- [ ] Amendment ADR-0020 §3 (izvor istine) i §10 (UNION + `direction`) tek uz implementaciju
- [ ] Amendment `DATA_ARCHITECTURE.md` ER uz implementaciju
- [ ] Ažuriranje `uvoz-vozila-knjizenje.md`: PPMV rješenje kao `OfficialDocument`, ne samo opis JE
- [ ] DMS / generički `Attachment` ostaje Sprint 5 gate, zaseban ADR

## 5. Acceptance

Referentni slučaj: PPMV rješenje **UP/I-410-22/26-09/49557**, Carinska uprava, 03.08.2026., Audi A8, **10.347,20 €**.

| Slučaj | Očekivanje |
|--------|------------|
| Dokument registriran s PDF-om | vidljiv u `/dokumenti` (Svi) i tabu Ulazni (`direction=incoming,official`) |
| Redak u listi | `direction=official`, `kind=official`, izdavatelj, broj, datum, iznos |
| `GET /api/documents/?direction=incoming` | **ne** vraća OfficialDocument |
| `GET /api/documents/?direction=official` | samo OfficialDocument |
| Detalj | PDF se otvara iz UI-a; povezana temeljnica vidljiva ako je `JournalEntry.source` postavljen |
| `related_fixed_asset` popunjen | veza na karticu vozila; dokument bez imovine i dalje valjan |
| Nema JE | knjiženje `not_posted`; nema kontrole `issued_unposted` |
| JE + bank match | operativno „plaćeno” iz postojećeg bank/posting pravila |
| Nema `SubledgerItem` | open amount / aging `not_recorded`, ne `0` i ne zeleni status |
| Drugi tenant + poznati ID | 404 |
| Viewer preuzme tuđi file | 404 |
| `kind` izvan `tax_decision` \| `other` | odbijeno |
| `status` `posted` / `paid` / `superseded` | polje ne postoji |

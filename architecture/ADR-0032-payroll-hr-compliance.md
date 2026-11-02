# ADR-0032 — Payroll & HR Compliance Architecture

```text
Status: Accepted
Date: 2026-11-02
Type: Domain
Supersedes: —
Related: ADR-0008-submission-events.md, ADR-0009-submission-module-v1.md, ADR-0010-domain-architecture.md, ADR-0011-architecture-freeze-v1.0.md, ADR-0031-tz2-return.md, ARCHITECTURE_PRINCIPLES.md, ARCHITECTURE_GOVERNANCE.md, EVENTS.md, banking-v2.md
```

## Status

**Accepted** — retroaktivno dokumentira implementirani Payroll/HR modul. Amendment na freeze (ADR-0011): novi Django app `payroll` i nove vrste dokumenta (`JOPPDReturn`, `PayrollDocument` IP1/NP1/IO1/NO1). Bez izmjene frozen `SubmissionService`, bez izmjene Banking v2 lifecyclea, bez izmjene PDV jezgre.

Arhitektura je prihvaćena iako produkcijski compliance gate **nije PASS** (vidi [Known blockers](#known-blockers)). Blokeri su podatkovni i regulatorni, ne arhitekturni.

## Context

ADR-0010 vodi **HR** kao L0 stub (`domains/hr/`, „Plaće, JOPPD, zaposlenici — Faza 3”). ADR-0014 vodi JOPPD kao v1.5 kandidata (Tax + HR). [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md) propisuje da Tax ne importira HR modele, nego prima DTO.

Implementacija je uvela:

- novi Django app `payroll` (zaposlenici, ugovori, obračunski periodi, obračunske liste, stavke, zakonski parametri, evidencija radnog vremena, ovrhe, isplate, isprave),
- HTTP sloj u `domains/hr/api/` i facade `domains/hr/facade.py`,
- obračun plaće (bruto → MIO → dohodak → porez → neto → za isplatu),
- versionirane zakonske parametre i snimku parametara po listi,
- knjiženje u glavnu knjigu i storno,
- JOPPD kao poreznu ispravu u Tax domeni (`domains/tax/forms/joppd/`, `accounting.JOPPDReturn`).

Governance ([`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md)) traži ADR za novi Django app i za novu vrstu dokumenta. Ovaj ADR to pokriva jednim zapisom.

## Decision

### 1. `payroll` je persistence i operativni sloj

Django app `payroll` (`api/app/payroll/`) drži modele i operativne servise: `Employee`, `EmploymentContract`, `PayrollPeriod`, `Payslip` (+ earning / contribution / adjustment lines), `PayrollTaxYear`, `PayrollLegalParameter`, `MunicipalityIncomeTaxRate`, `WorkTimeType` / `WorkTimeEntry` / `WorkTimeMonth` / `WorkTimeRateOverride`, `EmployeeEnforcement`, `PayrollPayment`, `AnnualLeaveEntitlement`, `PayrollDocument`, `JoppdCode`.

### 2. Business logika u servisima i facadeu, bez reorganizacije

- Obračun i tokovi: `payroll/services/` (`calculate`, `generate`, `legal_params`, `worktime`, `posting`, `records`, `lifecycle`, `reports`, `pdf`, `api`).
- HTTP: `domains/hr/api/` (tanki viewovi nad `payroll.services.api`).
- Granica prema Tax: `domains/hr/facade.py` vraća DTO. `domains/tax` **ne importira** `payroll.models`.
- Nema big-bang premještanja `payroll` u `domains/hr/`. Isti obrazac kao postojeći `accounting` / `domains/*` (ADR-0010).

### 3. Zakonski parametri nisu hardkodirani

`PayrollLegalParameter`: `code`, `value`, `valid_from` / `valid_to`, `effective_basis` (`PAYMENT_DATE` | `PAYROLL_MONTH` | `ENTITLEMENT_PERIOD` | `EVENT_DATE`), `source_reference`, `verified_by` / `verified_at`. `MunicipalityIncomeTaxRate` ima iste validity i basis atribute. Resolver je `payroll/services/legal_params.py` (`resolve_rules`). `PayrollTaxYear` ostaje fallback izvor dok tablica parametara nije potpuna.

### 4. Snimka parametara po obračunu

Svaki obračun upisuje `Payslip.parameters_snapshot` (vrijednost, datum osnovice, basis, id retka parametra, JLS stopa). Povijesni obračun ostaje reproducibilan i nakon promjene parametara.

### 5. Evidencija radnog vremena prije obračuna

`WorkTimeType` → `WorkTimeEntry` (dan × vrsta × sati) → `WorkTimeMonth` (`open` / `reviewed` / `locked` / `reopened`). Zaključavanje upisuje `content_hash` (SHA-256 nad redcima). Generiranje liste iz **zaključanog** mjeseca stvara earning stavke po vrsti rada (`source=worktime`). Bez zaključanog mjeseca vrijedi fallback: ugovorni bruto, proporcionalan danima u mjesecu i postotku radnog vremena (`source=contract`). Ponovno otvaranje mjeseca nije dopušteno ako postoji knjižena lista; obračunate liste vraća u nacrt.

### 6. Nepromjenjivost i ispravci

- **Posted** lista i njezine stavke su nepromjenjive na razini modela (`Payslip.save`, `_reject_posted_payslip_edit`, admin read-only).
- Do knjiženja je lista radna verzija: smije se ponovno obračunati, a svaki obračun piše novu snimku parametara. Ručni unos stavki dopušten je samo prije knjiženja.
- Ispravak knjižene liste ide novim zapisom (`Payslip.supersedes`) ili stornom, nikad prepisivanjem.
- Prijelazi statusa (`PayrollPeriod`, `Payslip`, `WorkTimeMonth`) idu isključivo kroz `payroll/services/lifecycle.py`, registrirano u `lifecycle_guard`.

### 7. Knjiženje kroz postojeći Finance

Knjiženje (`payroll/services/posting.py`) stvara `accounting.JournalEntry` (`document_type=payroll_posted`) prema `PostingRule` ključevima. Konta dolaze iz pravila, ne iz koda. MIO II se knjiži odvojeno samo ako tenant ima pravilo `payroll_mio1`. Obustave imaju vlastitu obvezu. Knjižena temeljnica se ispravlja `JournalEntry.reverse()`, a storno se veže na `PayrollPeriod.reversal_entry`.

### 8. Bez novog banking lifecyclea

`PayrollPayment` (neto / porez / MIO I / MIO II / ZO / obustava) se opcionalno veže na postojeći `banking.PaymentOrder` preko `banking.services.pis_orders.create_draft_payment_order`. Payroll ne mijenja status naloga niti uvodi vlastiti platni tok (Banking v2 ostaje frozen).

### 9. Payroll isprave

IP1, NP1, IO1, NO1 i rekapitulacije (knjiženje, isplate, obračun) su isprave izvedene iz kanonskog stanja liste i perioda. Gdje se ispravu sprema, sprema se kao `PayrollDocument`: append-only, sa `content_hash`. NP1 nastaje samo za neplaćenu ili djelomično plaćenu plaću nakon roka. IO1 / NO1 nastaju iz liste vrste `termination`.

### 10. JOPPD je integracija Tax + HR

Payroll/HR daje kanonske podatke kroz `domains/hr/facade.py`. Tax gradi `JoppdPayload` i `accounting.JOPPDReturn` (`domains/tax/forms/joppd/`). Predaju i audit bilježe postojeći `SubmissionService` / `SubmissionEvent` (ADR-0008, ADR-0009). Oznaka izvješća je YYDDD prema datumu isplate. Agregacija odbija period koji nije obračunat ni knjižen.

### 11. Frozen Submission jezgra se ne mijenja

`accounting/services/submission/` nije diran. JOPPD koristi `SubmissionService.create_event` / `current_submission` kao i TZ2 (ADR-0031).

### 12. JOPPD radi fail-closed

Predaja je blokirana dok `JOPPD_SUBMIT_ENABLED` nije uključen (zadano `False`). Spremljena shema `accounting/services/schemas/joppd-v1-0-draft.xsd` je zamjenska skica, ne službeni ePorezna XSD. Uključivanje predaje traži službeni XSD i potvrđene šifrarnike (`JoppdCode`).

### 13. Audit i čuvanje zapisa

Payroll radnje se bilježe u postojeći `accounts.AuditLog` (`payroll.services.records.audit`): zaključavanje i otvaranje evidencije, knjiženje, storno, isplata, obustava ovrhe, NP1, prestanak. Poslovni zapisi su zaštićeni `PROTECT` vezama (zaposlenik, period, evidencija, isplate, isprave) i nepromjenjivošću knjiženih lista i isprava. Isti princip kao ostatak racunAI (immutable posted records, ispravak stornom).

### Eventi

Payroll ne uvodi nove domenske evente. Knjiženje je sinkroni poziv Finance servisa iz Payroll servisa. [`EVENTS.md`](EVENTS.md) se ne mijenja.

## Consequences

### Prednosti

- Reproducibilan obračun: snimka parametara po listi.
- Auditabilnost: AuditLog po radnji, hash evidencije i isprava, storno umjesto brisanja.
- Versionirani zakonski parametri s validity, osnovicom datuma i metapodacima o izvoru.
- Integracija bez dupliciranja: Finance (`JournalEntry`, `PostingRule`), Banking (`PaymentOrder`) i Submission (`SubmissionEvent`) koriste se kakvi jesu.

### Rizici / trade-off

- Veća regulatorna složenost: obračun ovisi o točnosti parametara, JLS stopa, pravila ovrhe i minimuma vrsta rada.
- Trajno održavanje: parametri po godini i JOPPD sheme/šifrarnici moraju se ažurirati iz službenih izvora.
- Payroll persistence ostaje u top-level appu, a HTTP u `domains/hr/` (svjesno, bez reorganizacije).

### Poznata odstupanja implementacije od odluka (follow-up, ne nova arhitektura)

- Resolver bira datum po osnovici zapisanoj u kodu; `effective_basis` iz baze se upisuje u snimku, ali još ne određuje izbor datuma.
- `Payslip` nema status `superseded`; lanac ispravaka postoji samo kao `supersedes` FK, bez servisa koji stvara ispravak-listu. `PayrollPeriod.approved` / `closed` su definirani, bez prijelaza.
- IP1 i rekapitulacije renderiraju se kao PDF na zahtjev; `PayrollDocument` se trenutno sprema za NP1, IO1 i NO1.
- Rok čuvanja (npr. evidencija radnog vremena) nije provedeno pravilo; otvoreni mjesec evidencije smije se mijenjati.
- Zaštićeni iznos kod ovrhe ne koristi `PROTECTED_REFERENCE_NET`, a ponovni obračun briše podjelu zaštićeno/redovno.

### Follow-up

- [ ] Living docs: HR maturity u [`DOMAIN_MAP.md`](DOMAIN_MAP.md) / [`CAPABILITY_MAP.md`](CAPABILITY_MAP.md), JOPPD u [`docs/tax/FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md)
- [ ] Odstupanja iz gornje liste
- [ ] Službeni JOPPD XSD + `JoppdCode` seed, zatim uključivanje predaje

## Known blockers

Produkcijski **PAYROLL CALCULATION GATE** i **JOPPD SUBMISSION GATE** trenutno su **BLOCKED** (Payroll Legal Data Gate, 2026-11-02). Blokeri:

- Nepotvrđeni zakonski parametri: nijedan `PayrollLegalParameter` nema službeni `source_reference` ni `verified_by`.
- JLS stope: 556 stopa za 2026. bez službenog izvora (sekundarni izvori).
- Pravila ovrhe i zaštićenog dohotka: kvote i redoslijed su neprovjerene konstante.
- WorkTime minimumi: uvećanja i `legal_min_pct` nisu potvrđeni; naknada za odsutnosti nema zakonsku osnovicu.
- Službeni JOPPD XSD i `JoppdCode` šifrarnici nisu dostupni.

Ovo su blokeri podatkovne i regulatorne spremnosti. Rješavaju se unosom potvrđenih podataka i odstupanjima iz follow-upa, ne promjenom ove arhitekture.

## References

- [ADR-0010 — Domain Architecture](ADR-0010-domain-architecture.md)
- [ADR-0011 — Architecture Freeze v1.0](ADR-0011-architecture-freeze-v1.0.md)
- [ADR-0009 — Submission modul v1](ADR-0009-submission-module-v1.md)
- [ADR-0008 — Generic Submission Event Architecture](ADR-0008-submission-events.md)
- [ADR-0031 — Obrazac TZ 2](ADR-0031-tz2-return.md) (precedent: nova porezna isprava bez izmjene `SubmissionService`)
- [ARCHITECTURE_PRINCIPLES.md](ARCHITECTURE_PRINCIPLES.md)
- [ARCHITECTURE_GOVERNANCE.md](ARCHITECTURE_GOVERNANCE.md)
- [EVENTS.md](EVENTS.md) (bez novih evenata)
- [DOMAIN_DEPENDENCY_MAP.md](DOMAIN_DEPENDENCY_MAP.md) (HR → Tax preko DTO)
- [banking-v2.md](banking-v2.md)
- Payroll compliance remediation plan (`payroll_compliance_remediation_e8bcb94e.plan.md`, Cursor plan, izvan repozitorija)

# ADR-0008 — Generic Submission Event Architecture

```text
Status: Accepted (amended by ADR-0009)
Date: 2026-07-07
Supersedes: —
Related: docs/architecture/ADR-0007-pdv-module.md, docs/architecture/ADR-0009-submission-module-v1.md, docs/accounting/pdv-obrazac-architecture.md
```

> **Amendment (ADR-0009):** Javni API je `SubmissionService` (`service.py`). Model koristi `event_uuid`, `payload_hash`, `source`; `document_type` i `submission_method` uklonjeni (derivirano / zamijenjeno). Detalji: [`ADR-0009-submission-module-v1.md`](ADR-0009-submission-module-v1.md).

## Status

**Accepted** — `SubmissionEvent` zamjenjuje fragmentiranu evidenciju predaje (`VATReturn` submission polja + `PdvSSubmission`) jedinstvenim append-only audit modelom za sve porezne obrasce.

## Context

Evidencija predaje poreznih obrazaca bila je fragmentirana:

- **Obrazac PDV** — submission polja direktno na `VATReturn` (`submitted_at`, `eporezna_identifier`, `submission_method`, …)
- **Obrazac PDV-S** — zasebni model `PdvSSubmission` (1:1 po razdoblju, bez ispravaka, bez verzioniranja)

Oba pristupa dijele isti poslovni koncept (operator predaje obrazac vanjskom sustavu, ERP bilježi audit), ali imaju različite modele i servise. Budući obrasci (JOPPD, ZP, PDV-ZK, INO-DOH) zahtijevaju isti audit bez novog modela po obrascu.

Tri odgovornosti moraju ostati odvojene:

| Odgovornost | Gdje živi |
|-------------|-----------|
| Generiranje obrasca | `build_pdv_payload`, `aggregate_pdv_s_rows`, budući generatori |
| Verzioniranje obrasca | `VATReturn.version`, `PDVSReturn.version` |
| **Evidencija predaje** | **`SubmissionEvent`** (append-only audit) |

## Decision

### SubmissionEvent — jedinstveni audit entitet

`SubmissionEvent` referencira bilo koji obrazac preko `GenericForeignKey` (`content_type` + `object_id`):

```text
SubmissionEvent
  tenant
  document_type          # pdv | pdv_s | joppd | zp | ...
  document               # GenericFK → VATReturn | PDVSReturn | ...
  submission_no          # 1, 2, 3… — po dokumentu, dodjeljuje servis
  state                  # pending | submitted | rejected | cancelled
  destination            # eporezna | mojeporezna | fiskalizacija | hzzo | test | ...
  external_identifier    # UUID ili drugi ID iz vanjskog sustava
  submitted_at
  submitted_by
  submission_method      # manual | api | import
  confirmation_attachment  # opcionalno
  supersedes_submission  # FK self, PROTECT — audit lanac
  created_at
```

### Zašto se napušta PdvSSubmission

- Duplicirani model s istim poljima kao PDV submission
- Nema lanac ispravaka (`submission_no`, `supersedes`)
- Nije proširiv na JOPPD, ZP i druge obrasce
- Jedna migracija i backfill jeftiniji od refaktora `PdvSSubmission` pa kasnije generičkog modela

### Zašto se submission odvaja od generatora

ERP **generira** obrazac (`VATReturn`, `PDVSReturn`). Operator **predaje** na ePoreznu. Audit **bilježi** predaju (`SubmissionEvent`). Predaja nije svojstvo obrasca — to je događaj koji referencira obrazac.

`VATReturn.status=submitted` ostaje workflow flag; audit podaci žive na `SubmissionEvent`.

### Append-only filozofija

- Event se **nikad ne briše**
- Ispravak = novi event s većim `submission_no` i `supersedes_submission` → prethodni
- Core audit polja **immutable** nakon kreacije
- **`state` je jedino polje koje se može mijenjati** — isključivo kroz `transition_submission_state()`

### GenericFK

Jedan model za PDV, PDV-S, JOPPD, ZP, … Novi obrazac = novi Return model + `TaxDocumentType` enum entry — **bez** novog audit modela.

### supersedes_submission — audit lanac ispravaka

Pri kreaciji `#N` (`N > 1`): `supersedes = zadnji event za isti document`.

`get_active_submission(document)` = event s max `submission_no` **gdje je `state=submitted`**.

### Izvedeni submission_type

```python
@property
def submission_type(self) -> str:
    return 'initial' if self.submission_no == 1 else 'correction'
```

Nije u bazi — derivira se iz `submission_no`.

### destination + external_identifier

Model **nije** vezan uz ePoreznu:

| destination | external_identifier | Kontekst |
|-------------|---------------------|----------|
| `eporezna` | UUID | Ručna predaja / API ePorezna |
| `mojeporezna` | … | Budući kanal |
| `fiskalizacija` | … | Fiskalizacija 2.0 |
| `hzzo` | … | Zdravstveni obrasci |
| `test` | … | Sandbox / dev |

Unique constraint: `(tenant, destination, external_identifier)`.

### state lifecycle

```text
[*] → pending: API submit
[*] → submitted: manual_evidence
pending → submitted: confirmation
pending → rejected: rejection
pending → cancelled: cancel
```

Danas: ručna evidencija → `state=submitted` odmah pri kreaciji. Budućnost (API): `create_submission_event(state=pending)` → `transition_submission_state(event, submitted)` nakon potvrde.

### Servis — jedinstveni entry point

Modul: `accounting/services/submission/` (vidi **ADR-0009**)

- `SubmissionService.create_event()` — jedini način kreacije eventa
- `SubmissionService.attach_confirmation()` — prilog potvrde (jednom)
- `SubmissionService.supersede()` — ispravak predaje
- `SubmissionService.validate()` — validacija priloga bez side effects
- `SubmissionService.current_submission()` — aktivna predaja po dokumentu
- `transition_submission_state()` — jedini način promjene `state` (interni helper)

Refaktor:

- `mark_vat_return_submitted()` → `SubmissionService.create_event(..., destination=eporezna)`
- `mark_pdv_s_submitted()` → isto (dokument = `PDVSReturn`)
- `import_signed_vat_return()` → arhiva XML; **ne** koristi XML Identifikator kao `external_identifier`

### PDVSReturn — lagani anchor za PDV-S

```text
PDVSReturn
  tenant
  vat_period          # FK VATPeriod, UNIQUE per period
  version             # default 1; rezervirano za buduće XML verzioniranje
  created_at
```

## Consequences

### Prednosti

- Jedan audit model za cijeli porezni modul
- Lanac ispravaka (PDV-S korekcije) bez brisanja povijesti
- Proširivo na sve vanjske sustave (`destination`) bez migracije
- `state=pending` priprema API integraciju bez nove sheme
- Jasna separacija generator / verzija / predaja

### Svjesni kompromisi

- `GenericForeignKey` — nema DB-level FK integriteta na dokument
- Migracija postojećih podataka iz `VATReturn` i `PdvSSubmission`
- Admin inline za GenericFK zahtijeva `GenericTabularInline`
- `VATReturnSubmissionMethod` (`manual_ep`) zamijenjen s `SubmissionMethod` (`manual`) na eventu

## Alternatives considered

| Odbačeno | Razlog |
|----------|--------|
| Refaktor `PdvSSubmission` | Duplicirani kod; druga migracija kad dođu JOPPD/ZP |
| Submission polja ostaju na `VATReturn` | Ne skalira na PDV-S ispravke ni druge obrasce |
| `submission_type` u bazi | Derivira se iz `submission_no`; suvišna denormalizacija |
| Brisanje eventa za ispravak | Krši audit zahtjeve Porezne uprave |

## References

- [`ADR-0007-pdv-module.md`](ADR-0007-pdv-module.md) — PDV jezgreni pipeline (amendment: predaja u SubmissionEvent)
- [`pdv-obrazac-architecture.md`](../accounting/pdv-obrazac-architecture.md) — modeli i workflow
- [`pdv-accountant-workflow.md`](../accounting/pdv-accountant-workflow.md) — operativni koraci za računovođe

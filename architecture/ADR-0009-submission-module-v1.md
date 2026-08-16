# ADR-0009 — Submission modul v1

```text
Status: Accepted
Date: 2026-07-07
Supersedes: —
Related: docs/architecture/ADR-0008-submission-events.md (amended), docs/accounting/pdv-obrazac-architecture.md
```

## Status

**Accepted** — Submission modul v1 zamrzava arhitekturu evidencije predaje: `SubmissionService` kao jedini ulaz, `event_uuid` + `payload_hash` + `source`, uklanjanje `document_type` iz baze, validacija priloga.

Amendment na **ADR-0008**: servisni modul, polja modela i pozivatelji opisani u ovom ADR-u nadopunjuju ADR-0008.

> **Architecture freeze:** Submission modul v1 smatra se referentnom arhitekturom za evidenciju predaje poreznih obrazaca. `SubmissionService`, `SubmissionEvent`, operativni workflow i lifecycle **ne mijenjaju se** bez novog ADR-a koji obrazlaže odstupanje. Nova ideja mora prvo dokazati da postojeća arhitektura ne može podržati traženo ponašanje. NICE TO HAVE stavke ostaju u backlogu — vidi [`submission-module-backlog.md`](submission-module-backlog.md).

## Context

ADR-0008 uveo je `SubmissionEvent` i `create_submission_event()`. Operativni rad (Fine Star 05/2026) pokazao je potrebu za:

- audit dokazom sadržaja predaje (`payload_hash`), ne samo prilogom,
- javnim identifikatorom eventa (`event_uuid`) odvojenim od DB `id`,
- jasnom razlikom **kako** je zapis nastao (`source`) vs **kamo** je poslano (`destination`),
- centraliziranom validacijom priloga (XML vs ostalo),
- jedinstvenim servisom bez duplicirane poslovne logike u adminu/CLI/importu.

## Decision

### 1. SubmissionService — jedini ulaz

Modul: `accounting/services/submission/service.py`

```python
class SubmissionService:
    @staticmethod
    def create_event(document, *, destination, external_identifier, ...) -> SubmissionEvent: ...

    @staticmethod
    def attach_confirmation(event, file, *, uploaded_by) -> SubmissionEvent: ...

    @staticmethod
    def supersede(document, *, previous_event, ...) -> SubmissionEvent: ...

    @staticmethod
    def validate(file, *, document, event) -> ValidationResult: ...

    @staticmethod
    def current_submission(document) -> SubmissionEvent | None: ...
```

**Zabranjeno:** admin, CLI, import, migracije implementiraju poslovna pravila sami — samo delegacija na `SubmissionService`.

`events.py` postaje interni helper (`transition_submission_state`, `next_submission_no`, …).

### 2. SubmissionEvent append-only

- Nikad edit core polja
- Nikad delete
- Nikad overwrite
- Greška → `supersede()` → novi event (`submission_no` + 1)

Jedina iznimka: `confirmation_attachment` postavlja se **jednom** ako je prazan.

`state` mijenja se **samo** kroz `transition_submission_state()`.

### 3. submission_no

Jedini izvor istine za redoslijed predaja po dokumentu. Dodjeljuje **isključivo servis** (`next_submission_no` + `select_for_update`).

### 4. submission_type — derived, ne u bazi

```python
@property
def submission_type(self) -> str:
    return 'initial' if self.submission_no == 1 else 'correction'
```

### 5. supersedes_submission

FK self, PROTECT — audit chain ispravaka. `supersede()` postavlja link na prethodni event.

### 6. payload_hash — obavezno na eventu

Nova kolona `SubmissionEvent.payload_hash` (CharField, max 64).

Pri `create_event()`:
- Spremiti hash sadržaja koji je predan
- Za PDV: `document.get_payload_hash()` (ERP canonical hash drafta)
- Za import: hash iz parsiranog potpisanog XML-a
- Pri `attach_confirmation` s XML-om: `validate()` uspoređuje hash priloga s `event.payload_hash`

**Važnije od spremanja XML-a** — XML je prilog; hash je audit dokaz.

### 7. confirmation_attachment

Generički naziv — XML, PDF, ZIP, screenshot. **Ne** `signed_xml`.

### 8. Attachment validation

`SubmissionService.validate()` poziva se iz `attach_confirmation()`:

| Tip | Provjera |
|-----|----------|
| XML | OIB, razdoblje, tip obrasca, XSD, digitalni potpis |
| PDF/ZIP/screenshot/ostalo | Spremi; MIME/extension check; bez XML validacije |

### 9. Portal UUID — ADR pravilo

`external_identifier` = **portal zaprimanje** (ePorezna ekran/PDF).

**Nikada** prepisivati UUID-om iz `Metapodaci/Identifikator` u XML-u.

Document UUID ide u log / cross-check, ne u `external_identifier`.

### 10. event_uuid

Nova kolona `SubmissionEvent.event_uuid` (UUIDField, unique, default=uuid4).

- Javni identifikator eventa (API, logovi, podrška)
- **Ne** koristiti DB `id` u vanjskim sučeljima
- Immutable nakon kreacije

### 11. destination

Enum proširen:

```text
eporezna | hzzo | fina | mojeporezna | fiskalizacija | test
```

### 12. external_identifier

Generički naziv — ADR: ne vraćati se na `eporezna_identifier`.

### 13. source — kako je event nastao

Nova kolona `SubmissionEvent.source` (zamjenjuje `submission_method`):

```text
manual | api | migration | import | system
```

Razlika od `destination`: `source` = **kako je ERP zapis nastao**; `destination` = **kamo je poslano**.

### 14. state

```text
pending | submitted | rejected | cancelled
```

### 15. current_submission helper

```python
SubmissionService.current_submission(document) -> SubmissionEvent | None
```

Event s max `submission_no` gdje `state=submitted`. **Zabranjeno:** `order_by('-submission_no')` razbacano po kodu.

### 16. SubmissionDocument protocol

`accounting/services/submission/protocol.py`:

```python
class SubmissionDocument(Protocol):
    def get_period(self) -> VATPeriod: ...
    def get_display_name(self) -> str: ...
    def get_version(self) -> int: ...
    def get_payload_hash(self) -> str | None: ...
```

`SubmissionEvent` **ne zna** što je PDV. Ukloniti `document_type` s modela — tip iz `content_type`.

### GenericFK (v1)

v1 koristi GenericFK (`content_type` + `object_id`) za referencu na dokument. Ako se pokaže potreba (performanse, integritet, složeniji upiti), dopuštena je migracija na jače tipiziran model uz novi ADR — bez promjene audit filozofije (`SubmissionEvent` ostaje generički).

## Konačni model SubmissionEvent (v1)

```text
SubmissionEvent
  event_uuid              UUID, unique, immutable, javni ID
  tenant
  content_type            GFK
  object_id               GFK
  document                GenericFK → SubmissionDocument
  submission_no           servis dodjeljuje
  state                   pending | submitted | rejected | cancelled
  destination             eporezna | hzzo | fina | ...
  external_identifier     portal UUID (immutable)
  payload_hash            SHA-256 sadržaja predaje (immutable)
  submitted_at
  submitted_by
  source                  manual | api | migration | import | system
  confirmation_attachment FileField, opcionalno, postavi jednom
  supersedes_submission   FK self, audit chain
  created_at

  @property submission_type → initial | correction (derived)
  @property document_type → pdv | pdv_s | … (derived from content_type)
```

**Uklonjeno:** `document_type`, `submission_method`

## Operativni workflow

```text
Generiraj XML
    ↓
Predaj u ePoreznu
    ↓
Preuzmi potvrdu
    ↓
Označi predano        → create_event (portal UUID + payload_hash)
    ↓
Priloži potvrdu       → attach_confirmation (XML/PDF/…)
```

## NICE TO HAVE (samo dokumentirano, ne u v1)

| # | Stavka | Napomena |
|---|--------|----------|
| 17 | `SubmissionAttachment` kolekcija | FileField može evoluirati |
| 18 | Timeline UI | Audit prikaz po dokumentu |
| 19 | `reason` na supersede | Accounting adjustment, PU request, … |
| 20 | `event_notes` | Interna napomena, ne ide Poreznoj |

## Consequences

### Prednosti

- Jedan servis, jedan audit trag, testabilan API
- `payload_hash` + prilog = potpuna revizijska slika
- `event_uuid` spreman za vanjski API
- `source` vs `destination` jasno razdvojeni

### Svjesni kompromisi

- GenericFK bez DB integriteta (escape hatch u ADR-u)
- Migracija `submission_method` → `source`, uklanjanje `document_type`
- Backfill postojećih eventa za `payload_hash` i priloge

## References

- [`submission-module-backlog.md`](submission-module-backlog.md) — post-v1 backlog (GitHub issues)
- [`ADR-0008-submission-events.md`](ADR-0008-submission-events.md)
- [`pdv-accountant-workflow.md`](../accounting/pdv-accountant-workflow.md)
- [`pdv-obrazac-architecture.md`](../accounting/pdv-obrazac-architecture.md)

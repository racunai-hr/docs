# Submission modul — backlog (post v1)

```text
Status: Backlog (nakon architecture freeze)
Prerequisite: ADR-0009 Accepted — Submission modul v1 implementiran
Architecture: Ne mijenjati SubmissionService / SubmissionEvent bez novog ADR-a
```

Operativna arhitektura zamrznuta u [`ADR-0009-submission-module-v1.md`](ADR-0009-submission-module-v1.md). Ove stavke **nisu** u v1 scopeu; implementirati tek kad postoji konkretan poslovni zahtjev.

| # | Stavka | GitHub | Prioritet | Napomena |
|---|--------|--------|-----------|----------|
| 1 | `SubmissionAttachment` kolekcija | [#14](https://github.com/avrcanio/racunai.hr/issues/14) | Niska | Zamjena jednog `FileField`-a višestrukim prilozima |
| 2 | Timeline UI | [#15](https://github.com/avrcanio/racunai.hr/issues/15) | Srednja | Audit prikaz predaja po dokumentu u adminu |
| 3 | `reason` na supersede | [#16](https://github.com/avrcanio/racunai.hr/issues/16) | Srednja | Accounting adjustment, PU zahtjev, … |
| 4 | API submission (ePorezna) | [#17](https://github.com/avrcanio/racunai.hr/issues/17) | Niska | `state=pending` → potvrda; ovisi o PU API-ju |
| 5 | Notification hooks | [#18](https://github.com/avrcanio/racunai.hr/issues/18) | Srednja | Email / Slack / audit nakon `SubmissionEvent` |

---

## 1. SubmissionAttachment

**Problem:** v1 dopušta jedan `confirmation_attachment` po eventu.

**Cilj:** Kolekcija priloga (XML, PDF, screenshot, ZIP) bez gubitka append-only audit filozofije.

**Pristup (skica):**
- Novi model `SubmissionAttachment` (FK na `SubmissionEvent`, immutable nakon kreacije)
- `SubmissionService.attach_confirmation()` postaje wrapper ili delegira na `add_attachment()`
- Migracija postojećih `FileField` vrijednosti u prvi attachment

**Ne dirati:** `payload_hash`, `external_identifier`, `event_uuid` semantiku.

---

## 2. Timeline UI

**Problem:** Povijest predaja rasuta po inline tablicama (`VATReturn`, `PDV-S`).

**Cilj:** Jedinstveni audit timeline po dokumentu — submission_no, state, source, payload_hash, prilozi, supersede chain.

**Pristup (skica):**
- Admin view ili custom template na `VATReturn` / `VATPeriod`
- Read-only; podaci iz `SubmissionService.current_submission()` + `get_submission_events()`

---

## 3. Reason na supersede

**Problem:** Ispravak (`submission_no > 1`) nema objašnjenje zašto je predan.

**Cilj:** Opcionalno polje `reason` (enum ili tekst) pri `SubmissionService.supersede()`.

**Primjeri:** `accounting_adjustment`, `pu_request`, `operator_error`.

**Pristup:** Nova kolona + ADR amend; ne mijenja postojeće evente retroaktivno.

---

## 4. API submission (ePorezna)

**Problem:** Danas je sve `source=manual` s `state=submitted` odmah pri kreaciji.

**Cilj:** `create_event(state=pending)` → vanjski API → `transition_state(submitted|rejected)`.

**Gate:** Dostupnost službenog ePorezna API-ja i certifikati / OAuth.

**Pristup:** Transport sloj poziva postojeći `SubmissionService`; bez promjene jezgrenog modela.

---

## 5. Notification hooks

**Problem:** Nema automatskih obavijesti nakon evidencije predaje.

**Cilj:** Domain event nakon `create_event` / `attach_confirmation`:

```text
SubmissionEventCreated / SubmissionConfirmationAttached
    → email računovođi
    → Slack webhook (ops)
    → audit log (vanjski SIEM)
```

**Pristup:** Django signal ili eksplicitni hook u servisu **iza** transakcije; PDV generator ostaje netaknut.

---

## Pravilo pri implementaciji

1. Dokazati da v1 arhitektura ne pokriva zahtjev (ili novi ADR za odstupanje).
2. Jedan PR po stavci backloga.
3. Testovi na `SubmissionService` — ne duplicirati poslovnu logiku u adminu/CLI.

## Reference

- [`ADR-0009-submission-module-v1.md`](ADR-0009-submission-module-v1.md) — architecture freeze
- [`ADR-0008-submission-events.md`](ADR-0008-submission-events.md)
- [`pdv-accountant-workflow.md`](../accounting/pdv-accountant-workflow.md)

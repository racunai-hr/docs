# ADR-0027 — PDV correction lifecycle + Section I derived fields

```text
Status: Accepted
Date: 2026-08-23
Type: Business
Supersedes: —
Related: ADR-0007-pdv-module.md, ADR-0008-submission-events.md, ADR-0009-submission-module-v1.md
```

## Status

**Accepted** — Obrazac PDV ispravak za već predano razdoblje ide kroz novu `VATReturn` verziju i novi `SubmissionEvent`. Section I boxovi 101/103 ulaze u XML iz knjige; 111 i 000 su derived. Ne superseda ADR-0007/0008/0009.

## Context

ePorezna ne dopušta brisanje ni izmjenu već predanog Obrazca PDV unutar iste datoteke. Ispravak je nova verzija za isto razdoblje (Napomena: razlog ponovnog slanja).

ADR-0007 zamrzava jezgru pipelinea (`VAT_BOX_REGISTRY`, `build_pdv_payload`, `VATReturn` verzioniranje). 101/103 su `implemented` u knjizi, ali XML ih ignorira (`value_source=ZERO`); 111 je ZERO pa 000 = samo II. Rebuild `VATLedgerEntry` je zabranjen kad `VATPeriod.status != open`. `current_return` vraća zadnji submitted, pa se ispravak ne vidi na Preuzmi XML.

ADR-0008: `SubmissionEvent` je append-only audit **po dokumentu**. `supersedes_submission` veže se na isti GFK dokument. PDV-S ispravak (`supersede()` na jednom `PDVSReturn`) nije model za Obrazac PDV.

## Decision

### Section I derived fields

| Box | Izvor |
|-----|--------|
| 101, 103 | `AGGREGATE_BASE` iz `VATLedgerEntry` |
| 111 | `TOTAL_I` = zbroj skalara 100–110 |
| 000 | ostaje `111 + 200.vrijednost` (ADR-0007) |

`PDV_MAPPING_VERSION` se povećava. 303 ostaje PAIR iz knjige.

### Correction lifecycle

- `VATPeriod.status` ostaje `submitted` tijekom pripreme i predaje ispravka. Razdoblje se **ne** otvara.
- `current_return` = službeno predana verzija (zadnji `submitted`).
- `latest_return` = radna verzija (`order_by('-version').first()`).
- Običan ledger rebuild i običan draft na `submitted` ostaju 409.
- Ispravak samo kroz `prepare_pdv_correction()`:

```text
transaction.atomic:
  correction-only rebuild VATLedgerEntry
  → build payload
  → XSD validation
  → create VATReturn vN (generated)
```

Fail → rollback knjige i nema vN. Predani v1 payload/XML ostaju immutable.

- Ako `latest_return.status == generated` i postoji stariji submitted → **409 `CORRECTION_ALREADY_IN_PROGRESS`**.

### Audit predaje

Povijest PDV predaja = verzije `VATReturn` + `SubmissionEvent` na svakoj verziji. Nije PDV-S events-only lanac.

```text
VATReturn v1
  SubmissionEvent A  (immutable)

VATReturn v2
  SubmissionEvent B  (novi portal UUID, submitted_at, potvrda)

v1.superseded_by = v2
v1.status = superseded
v2.status = submitted
```

`B.supersedes_submission` se **ne** veže na A. `external_identifier` nije XML `Metapodaci/Identifikator`.

## Consequences

### Prednosti

- XML ispravka sadrži I. (101/103/111) pa ePorezna ne auto-ispravlja 000.
- Službena predaja i radni ispravak vidljivi istodobno (Predano + ispravak u pripremi).
- Audit v1 ostaje; v2 ima vlastiti event.

### Rizici / trade-off

- Correction-only rebuild je iznimka od `VATPeriod` writable gate; smije se zvati samo iz `prepare_pdv_correction`.
- `submission_type` na Event B derivira `initial` (prvi event na v2 dokumentu) — ispravak se čita iz `VATReturn.version` / `superseded_by`, ne iz PDV-S `submission_no`.

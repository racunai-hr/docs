# ADR-0031 — Obrazac TZ 2 (`TZ2Return`)

```text
Status: Accepted
Date: 2026-09-15
Type: Domain
Supersedes: —
Related: ADR-0008-submission-events.md, ADR-0009-submission-module-v1.md, ADR-0011-architecture-freeze-v1.0.md, docs/tax/FORM_IMPLEMENTATION_CONVENTION.md, docs/tax/TZ2_ARCHITECTURE.md
```

## Status

**Accepted** — Nova vrsta poreznog dokumenta `TZ2Return` (Obrazac TZ 2) unutar postojeće Tax konvencije. Amendment na freeze (ADR-0011): nova vrsta dokumenta, bez nove domene, bez novog Django appa, bez izmjene PDV jezgre i bez izmjene frozen `SubmissionService`.

## 1. Context

Obrazac TZ 2 (turistička članarina, privatni iznajmljivač) ima službeni XSD u `ePorezna_Schemas.zip` → `TZ 2/` (`ObrazacTZ2-v1-0`). Tenant Alma Čižmić predaje godišnji obrazac. RacunAI već ima obrazac za nove forme: ZP (`tax_forms/zp/` + `ZPReturn` + `SubmissionEvent`).

Governance: nova vrsta dokumenta zahtijeva ADR. `FORM_REGISTRY` sam po sebi nije dovoljan.

`SubmissionDocument.get_period()` je tipiziran kao `VATPeriod`. TZ 2 je **godišnji** obrazac — vezanje na `VATPeriod` bi lockalo PDV razdoblje pri predaji (lekcija PDV-S).

## 2. Decision

1. Uvesti `accounting.TZ2Return` po uzoru na `ZPReturn`: `tenant`, `tax_year`, `version`, `schema_version`, `mapping_version`, `payload_snapshot`, `payload_hash`, `payload_json`, `xml_unsigned`, `xml_submitted`, `unsigned_xml_sha256`, `prepared_by`. **Nema** `status` stupca — audit je `SubmissionEvent`.
2. Period je `tax_year` (1.1.–31.12.), **bez FK na `VATPeriod`**.
3. `get_period()` vraća duck-type adapter s `year` (protokol za `SubmissionService.create_event`). Predaja TZ2 **ne smije** mijenjati `VATPeriod.status`.
4. `TaxDocumentType.TZ2` + `_CONTENT_TYPE_DOCUMENT_TYPE['tz2return']` u `accounting/models.py` (upload path). XML potvrda ide kroz postojeći `SubmissionService.validate()` dispatch (isto kao ZP): validacija živi u `tax_forms/tz2`, `_update_document_workflow` i dalje **ne** dira `VATPeriod`. PDF/screenshot ostaje prihvaćen.
5. Kod: `accounting/services/tax_forms/tz2/` + `accounting/schemas/tz2/v1-0/` prema [`FORM_IMPLEMENTATION_CONVENTION.md`](../tax/FORM_IMPLEMENTATION_CONVENTION.md).
6. Nije `OfficialDocument`. Nije TZ ni TZ-1.

## 3. Consequences

### Prednosti

- Isti lifecycle kao ZP: payload SSOT → XML izveden → `SubmissionService.create_event()`.
- PDV period ostaje nezavisan.

### Rizici / trade-off

- XML confirmation u `validate()` ide kroz `tax_forms/tz2` (XSD + OIB + godina), bez locka `VATPeriod`.
- `Ime`/`Prezime` nisu `CompanySettings.company_name` — eksplicitni split/override.

### Follow-up

- [x] Operativni doc [`TZ2_ARCHITECTURE.md`](../tax/TZ2_ARCHITECTURE.md)
- [x] Registry + ePorezna matrica
- [ ] G2B (izvan v1)

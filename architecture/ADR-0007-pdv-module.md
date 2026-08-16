# ADR-0007 — PDV modul (Obrazac PDV v11.0)

```text
Status: Accepted
Date: 2026-07-05
Supersedes: —
Related: docs/accounting/pdv-obrazac-architecture.md (how)
```

## Status

**Accepted** — jezgreni pipeline implementiran, pokriven testovima i potvrđen na Fine Star referentnom razdoblju (travanj 2026). Promjene jezgre samo kroz novi ADR ili proširenja izvan pipelinea (`tax_forms/` podmoduli, novi boxovi u registryju).

## Context

Fine Star d.o.o. koristi racunAI ERP za knjigovodstvo i eRačun. Mjesečni Obrazac PDV dosad se sastavljao izvan ERP-a (ručni PDV-S XLSX, potpis na ePorezni, arhiva potpisanih XML-ova u `.temp/pdv_obrazac/`). Poziv Porezne uprave i potreba za audit tragom zahtijevaju:

- determinističku agregaciju iz ERP izvora (računi, troškovi, temeljnice),
- generiranje službenog `ObrazacPDV` v11.0 XML-a,
- verzioniranu arhivu predanih obrazaca po tenantu,
- usporedbu ERP izračuna s potpisanim XML-om s ePorezne.

Postojeći modeli `VATPeriod` i `VATLedgerEntry` nisu imali deklarativno mapiranje na polja obrasca niti neutralni poslovni model neovisan o XML-u. Agregacija po stopi PDV-a (`vat_rate`) i if-lanci ne skaliraju na EU stjecanje, uvoz, građevinski prijenos i boxove 610+.

Mini-checkpoint 2.5 (travanj 2026) potvrdio je da ERP agregat za implementirane boxove (201, 202, 203, 303, Podatak400) odgovara referentnom predanom XML-u prije implementacije payload sloja.

## Decision

Ključne arhitektonske odluke:

| Odluka | Sažetak |
|--------|---------|
| `VAT_BOX_REGISTRY` SSOT | Jedini izvor istine u `boxes.py`; `VATBox` enum, `pdv-mapping.md`, agregacija i testovi deriviraju iz registryja |
| `PdvPayload` immutable | `@dataclass(frozen=True)`; `build_pdv_payload()` uvijek kreira novi objekt |
| Arhitekturni gate | `build_pdv_payload(period)` poziva **samo** `aggregate_vat_boxes(period)` jednom; contract test mock-ovima ORM managera |
| Pipeline | `izvori → generate_vat_ledger → VATLedgerEntry → aggregate_vat_boxes → PdvPayload → render/parse/API` |
| Dva lifecycle servisa | `create_vat_return_draft()` (ERP → unsigned XML) i `import_signed_vat_return()` (potpisani → arhiva) |
| Evidencija predaje | `SubmissionEvent` + `SubmissionService` — append-only audit; `mark_vat_return_submitted()` / `mark_pdv_s_submitted()` delegiraju na servis (vidi ADR-0008, ADR-0009) |
| `VATReturn` 1:N | Verzioniranje po `VATPeriod`; immutable audit nakon `submitted`/`imported`; `payload.json` + XML po verziji |
| XSD prije persistencije | `validate_pdv_obrazac_xml()` u draft pipelineu — nema `VATReturn` zapisa ako validacija padne |
| `PdvFieldDifference` + `summary()` | Strukturirani diff s determinističkim sortiranjem za admin, logove i budući API |
| Dva brojača verzija | `PDV_MAPPING_VERSION` (poslovno značenje) i `schema_version` (XSD shema ePorezne) neovisno |
| Canonical Artifact Rule | `payload.json` je jedini kanonski izvor po verziji; `unsigned.xml` je izvedeni artefakt koji mora semantički odgovarati payloadu; ručne izmjene XML-a nisu dopuštene |
| Integritet drafta | `check_vat_return_integrity()` — semantička usporedba polja (I001) i SHA-256 byte hash (I002); admin blokira preuzimanje kad je OUT OF SYNC |

Poslovni iznosi računaju se **jednom** do `PdvPayload`. XML, PDF, HTML, REST i reconciliation koriste isti payload bez ponovnog računanja.

## Consequences

### Prednosti

- Jedno mjesto agregacije — nema divergencije između knjiga, admin pregleda i obrasca.
- Deterministički testovi: snapshot fixturei, round-trip parse/render, contract test na `build_pdv_payload`.
- Proširenja (boxovi 610+, PDV-K, ePorezna API) bez refaktora jezgre — samo registry + mapping ili novi `tax_forms/` podmodul.
- Audit trag predanih obrazaca po tenantu s hash-om i canonical JSON snapshotom.

### Svjesni kompromisi

- Više servisnih slojeva nego „jedan generator XML-a”.
- Unsigned XML se potpisuje izvan ERP-a u v1 (Finin certifikat / ePorezna ručno).
- `VAT_BOX_REGISTRY` mora se održavati pri svakom novom boxu — fail-fast ako `implemented=True` nema `_MAPPING_RULES`.
- Arhivski import (`vat_return=None`) i draft upload imaju različite puteve (namjerno — arhiva preskače usporedbu s draftom).
- Operativna evidencija predaje (`SubmissionEvent`) odvojena je od importa potpisanog XML-a — generator (ERP) ≠ predaja (operator + ePorezna). Vidi [`ADR-0008-submission-events.md`](ADR-0008-submission-events.md).

## Alternatives considered

| Odbačeno | Razlog |
|----------|--------|
| `VATReturn` 1:1 s `VATPeriod` | Nema draftova, ispravaka ni arhive verzija |
| `PdvPayload` kao običan `dict` | Nema type safety; teže refaktorirati i testirati immutability |
| Poslovna logika u XML rendereru | XML nije neutralni model; otežava PDF, API i usporedbu |
| Agregacija po `vat_rate` / if-lanci | Ne skalira na EU/uvoz/građevinski prijenos; krši SSOT registry |
| Ručni `PDV_MAPPING` dict | Drift između koda i docs; `build_mapping_from_registry()` fail-fast |
| `current_return` FK na `VATPeriod` | Rizik nekonzistentnosti; `@property` dovoljan |
| Admin upload bez `import_signed_vat_return()` | Duplicirana logika; bez strukturiranog diffa |

## References

- [`pdv-obrazac-architecture.md`](../accounting/pdv-obrazac-architecture.md) — modeli, workflow, storage, backward compatibility
- [`pdv-mapping.md`](../accounting/pdv-mapping.md) — ERP izvor → VATBox → polje obrasca
- [`pdv-stabilization-runbook.md`](../accounting/pdv-stabilization-runbook.md) — produkcijski tenant, regresija, performanse
- [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md) — planirana proširenja bez promjene jezgre
- [`ADR-0008-submission-events.md`](ADR-0008-submission-events.md) — generička evidencija predaje (SubmissionEvent)

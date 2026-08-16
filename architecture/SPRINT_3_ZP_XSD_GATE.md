# Sprint 3 — ZP XSD gate (prvi korak)

> **Izvor:** Cursor plan `sprint_3_zp_xsd_gate_881333f8` · ažurirano **2026-07-10**  
> **Sprint 3 zatvoren.** XSD gate ✅, ZP lifecycle ✅, PDV 610+/RC/OSS ✅, ADR-0014 ✅ — vidi [`ADR-0014`](ADR-0014-tax-domain-completion.md), [`ROADMAP.md`](ROADMAP.md).

## Checklist napretka

- [x] **Gate A** — Preuzeti `ePorezna_Schemas.zip`, locirati `ZP/` mapu
- [x] **Gate B** — Verifikacija: root XSD, verzija, `xs:include`/`import`, primjer XML
- [x] **Gate C** — Kopirati byte-identično u `accounting/schemas/zp/v1-0/`
- [x] **Gate D** — README manifest + [`ZP_ARCHITECTURE.md`](../tax/ZP_ARCHITECTURE.md) checklist (XSD korak)
- [x] **Gate E** — `test_zp_xsd_present.py` smoke test
- [x] **Faza 1** — `aggregate.py` (fixtures)
- [x] **Faza 2** — `render`, `parse`, `validation`
- [x] **Faza 3** — snapshot + round-trip
- [x] **Faza 4** — `ZPReturn` + `submit.py`
- [x] **Faza 5** — cross-form CI (PDV 101/103)
- [x] **Faza 6** — PDV 610+ → reverse charge → OSS
- [x] **Faza D** — ADR-0014 retrospective

**Ops (završeno):**

- [x] Admin UI — draft, download XML, označi predano
- [x] `verify_zp_period` management command
- [x] Produkcijsko mapiranje I-RA → boxovi 101/103 iz EU izlaznih računa
- [x] Commit + CI green (`pdv-ci.yml`, 12 `test_zp_*.py` modula)

---

## Kontekst

**Infrastructure freeze** vrijedi — vidi [`docs/tax/README.md`](../tax/README.md). Promjene tijekom Sprinta 3 moraju biti opravdane **poslovnom funkcionalnošću**, ne promjenom arhitekture.

**Gate (zatvoren 2026-07-10):** tri uvjeta ispunjena:

1. ZP XSD u [`accounting/schemas/zp/v1-0/`](../../erp/app/accounting/schemas/zp/v1-0/) **bez izmjena**
2. Službeni primjer XML (`examples/Primjer.xml`) spremljen uz sheme
3. Verzija i manifest evidentirani u [`accounting/schemas/zp/README.md`](../../erp/app/accounting/schemas/zp/README.md)

---

## Izvor sheme

PDV-S je importiran iz istog službenog arhiva:

| Stavka | Vrijednost |
|--------|------------|
| Zip | `https://e-porezna.porezna-uprava.hr/Upute/G2B/ePorezna_Schemas.zip` |
| PDV-S mapa | `PDV-S/` — vidi [`accounting/schemas/pdv-s/v1-0/README.md`](../../erp/app/accounting/schemas/pdv-s/v1-0/README.md) |
| ZP mapa | `ZP/` (7 datoteka) |
| Referenca arhiva | [`docs/porezna/upute/2026/README.md`](../porezna/upute/2026/README.md) |

---

## Faza A — Preuzimanje (bez Pythona)

1. Preuzeti `ePorezna_Schemas.zip` u privremeni direktorij (`.temp/schemas/` — ne commitati zip osim ako već nije u `docs/porezna/`).
2. Raspakirati i locirati mapu ZP.
3. Ako mapa ne postoji — provjeriti alternativne nazive; dokumentirati u README.

### Gate A — rezultat (2026-07-10)

| Stavka | Vrijednost |
|--------|------------|
| Zip (radna kopija) | `.temp/schemas/ePorezna_Schemas.zip` |
| Zip (arhiv u repou) | `docs/porezna/upute/2026/ePorezna_Schemas.zip` |
| SHA-256 zipa | `5670c921f1393b08b4ea00657f78f2e814253eda3eb855fcabcb50b9467db8ee` |
| Extract | `.temp/schemas/ePorezna_Schemas_extracted/` |
| **ZP mapa** | `ePorezna_Schemas/ZP/` (7 datoteka) |

Datoteke u `ZP/`:

| Datoteka | Uloga |
|----------|-------|
| `ObrazacZP-v1-0.xsd` | Root XSD (`verzijaSheme` 1.0) |
| `ObrazacZPtipovi-v1-0.xsd` | Tipovi (uključen iz root XSD) |
| `ObrazacZPmetapodaci-v1-0.xsd` | Metapodaci obrasca |
| `ObrazacZPsPotpisom-v1-0.xsd` | Potpis wrapper |
| `TemeljniTipovi-v2-1.xsd` | Shared temeljni tipovi |
| `MetapodaciTipovi-v2-0.xsd` | Shared metapodaci tipovi |
| `Primjer.xml` | Službeni primjer XML |

---

## Faza B — Verifikacija prije kopiranja

| # | Provjera | Rezultat |
|---|----------|----------|
| 1 | Root XSD — točan naziv datoteke | `ObrazacZP-v1-0.xsd` |
| 2 | Verzija sheme (`verzijaSheme`, namespace) | `1.0`, `http://e-porezna.porezna-uprava.hr/sheme/zahtjevi/ObrazacZP/v1-0` |
| 3 | `xs:include` / `xs:import` — potrebni `.xsd` | 5 lokalnih + potpisni wrapper (vanjski XSD-ovi nisu u zipu) |
| 4 | Primjer XML | `Primjer.xml` → `examples/Primjer.xml`; XSD validacija ✅ |
| 5 | Shared tipovi | Cijeli set kopiran; **nije** dijeljen s `pdv-s/v1-0/` (byte razlika) |

**Pravilo:** ne uređivati službene `.xsd` datoteke.

### Gate B — rezultat (2026-07-10)

Ovisnosti unsigned roota:

```text
ObrazacZP-v1-0.xsd
  └─ include → ObrazacZPtipovi-v1-0.xsd
       ├─ import → TemeljniTipovi-v2-1.xsd
       └─ import → ObrazacZPmetapodaci-v1-0.xsd
            └─ include → MetapodaciTipovi-v2-0.xsd
                 └─ import → TemeljniTipovi-v2-1.xsd
```

`ObrazacZPsPotpisom-v1-0.xsd` referencira `xmldsig-core-schema.xsd`, `XAdES.xsd`, `VanjskaOmotnica.xsd` — nisu u zipu (ePorezna potpisna infrastruktura). Lokalna validacija koristi unsigned root (uzorak PDV-S).

---

## Faza C — Spremanje u repo

```text
accounting/schemas/zp/v1-0/
    ObrazacZP-*.xsd
    *.xsd
    examples/Primjer.xml
    README.md                          # manifest u accounting/schemas/zp/README.md
```

**Status:** ✅ importano 2026-07-10, byte-identično iz zip arhiva.

---

## Gate zatvoren → implementacija ZP

Redoslijed (frozen konvencija — [`FORM_IMPLEMENTATION_CONVENTION.md`](../tax/FORM_IMPLEMENTATION_CONVENTION.md)):

| Faza | Modul | Status | Datoteke |
|------|-------|--------|----------|
| 1 | `aggregate.py` | ✅ | [`aggregate.py`](../../erp/app/accounting/services/tax_forms/zp/aggregate.py), fixtures [`tests/fixtures/tax/zp/`](../../erp/app/accounting/tests/fixtures/tax/zp/) |
| 2 | `render`, `parse`, `validation` | ✅ | [`render.py`](../../erp/app/accounting/services/tax_forms/zp/render.py), [`parse.py`](../../erp/app/accounting/services/tax_forms/zp/parse.py), [`validation.py`](../../erp/app/accounting/services/tax_forms/zp/validation.py) |
| 3 | Snapshot + round-trip | ✅ | `test_zp_render_parse_roundtrip`, `test_zp_create_return_draft`, `test_zp_xsd_validation` |
| 4 | `ZPReturn` + `submit.py` | ✅ | model + migracija `0018_zpreturn`, [`submit.py`](../../erp/app/accounting/services/tax_forms/zp/submit.py), `SubmissionService` |
| 5 | Cross-form CI | ✅ | `test_zp_pdv_cross_check`, `.github/workflows/pdv-ci.yml` (10 `test_zp_*.py` modula) |

**Testovi (Docker, `--keepdb`):** 48 testova — OK (2026-07-10).

**Sprint 3 zatvoren:** PDV 610+ → reverse charge → OSS → ADR-0014 ✅.

### Poznati gapovi (post-Sprint 3 backlog)

| Gap | Napomena |
|-----|----------|
| G2B predaja | Manual-only (`mark_zp_submitted`); G2B konektor v2 backlog — vidi ADR-0014 |
| Produkcijska regresija | Fine Star EU razdoblje — `verify_zp_period --xml` |
| VIES cross-check | Hook u `verify.py`; vanjski servis nije integriran |
| Export boxovi 102/104 | Treće zemlje — L1 proširenje |
| `test_zp_build.py` | Nije potreban — `build_zp_payload()` je pass-through na `aggregate_zp_rows()` |

---

## Definition of Done (gate)

- [x] `accounting/schemas/zp/v1-0/` — kompletan XSD set (nepromijenjen)
- [x] Primjer XML u repo (`examples/Primjer.xml`)
- [x] README manifest s verzijom i SHA-256 hashovima
- [x] Smoke test XSD + primjer (`test_zp_xsd_present.py`)
- [x] Nema promjena u `aggregate.py` **prije** zatvaranja gate-a (gate zatvoren prije pipeline commita)

---

## Reference

- [`ZP_ARCHITECTURE.md`](../tax/ZP_ARCHITECTURE.md)
- [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md)
- [`pdv_s/validation.py`](../../erp/app/accounting/services/tax_forms/pdv_s/validation.py) — XSD load uzorak

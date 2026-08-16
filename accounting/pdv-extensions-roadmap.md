# PDV modul — plan proširenja

```text
Status: Planned (nakon stabilizacije)
Prerequisite: pdv-stabilization-runbook.md — kriteriji prijelaza ispunjeni
Architecture: Jezgra zamrznuta — proširenja bez refaktora pipelinea
```

## Gate — proširenja tek nakon stabilizacije

**Ne implementirati** boxove 610+, PDV-K, ePoreznu integraciju ni REST API dok nisu ispunjeni svi kriteriji iz [`pdv-stabilization-runbook.md`](pdv-stabilization-runbook.md):

| Kriterij | Opis |
|----------|------|
| Produkcijska razdoblja | Min. 2 uzastopna mjesečna obračuna (Fine Star) s uspješnim `verify_pdv_period` + upload |
| Regresija | Nema mismatch-a između ERP payloada i predanog XML-a za implementirane boxove |
| Bugovi | Nema otvorenih P1 bugova na PDV workflowu |
| Performanse | Produkcijsko mjerenje (`verify_pdv_period --benchmark`) unutar baselinea iz `test_pdv_performance.py` |

Dok gate nije zatvoren, dopušteni su samo bugfixovi u mapping/ledger sloju i operativna dokumentacija — bez promjene pipelinea `ledger → aggregate → payload → XML`.

## Pravilo

Proširenja **ne refaktoriraju** pipeline:

```text
ledger → aggregate_vat_boxes → build_pdv_payload → render/parse
```

Dodaju se samo:
- novi boxovi u `VAT_BOX_REGISTRY` + `_MAPPING_RULES`,
- novi `tax_forms/` podmoduli za druge obrasce,
- transport sloj (ePorezna, REST API) koji poziva postojeće servise.

Svako proširenje koje mijenja generirani sadržaj → inkrement `PDV_MAPPING_VERSION`.

---

## 1. PDV boxovi 610+ (EU, uvoz, reverse charge)

### Scope

Boxovi 610–615, 620, 630, 640, 650 i povezani input boxovi (304–306, …) za:
- stjecanje dobara/usluga unutar EU,
- uvoz,
- reverse charge (građevinski prijenos, B2B usluge).

### Pristup

1. U `boxes.py`: `active=True`, `implemented=True`, `mapping_rule=...` za svaki box.
2. U `mapping.py`: dodati funkciju u `_MAPPING_RULES` (fail-fast pri importu).
3. U `vat.py` (`generate_vat_ledger`): proširiti izvore (Expense s EU oznakom, JE konta, …).
4. Inkrement `PDV_MAPPING_VERSION`.
5. Ažurirati `pdv-mapping.md`, snapshot fixture, checkpoint test za novo razdoblje.

### Ne dirati

- `build_pdv_payload` — već mapira sve `active` boxove iz agregata.
- `render.py` / `parse.py` — polja već u XSD v11.0.

---

## 2. PDV-K obrazac

### Scope

Godišnja/kvartalna prijava PDV-K (odvojena XSD shema).

### Pristup

Novi podmodul po istom uzorku:

```
accounting/services/tax_forms/pdvk/
  boxes.py       # PDVK_BOX_REGISTRY
  payload.py     # PdvKPayload (frozen)
  build.py       # build_pdvk_payload(period_range)
  render.py
  parse.py
  aggregate.py   # agregacija iz VATLedgerEntry / posebnih izvora
```

Dijeli:
- `canonical.py` (canonical_json, payload_hash),
- `versions.py` pattern za verzioniranje,
- admin workflow pattern (draft → upload → arhiva).

**Ne dijeli** `PdvPayload` — zaseban tip i model `PDVKReturn` (ili generički `TaxFormReturn` kasnije).

---

## 3. ePorezna integracija

**Referentna dokumentacija:** [`docs/tax/EPOREZNA_SUBMISSION_MATRIX.md`](../tax/EPOREZNA_SUBMISSION_MATRIX.md) — master matrica po obrascu (Transport, Auth, Response), extended appendix, glossary i G2B research checklist.

### Submission Capability Roadmap (trenutno stanje)

| Capability | Status |
|------------|--------|
| Generate XML | ✅ |
| XSD Validation | ✅ |
| Import Signed XML | ✅ |
| Submission Audit | ✅ |
| Manual Portal Submit | ✅ |
| G2B Submit | Research |
| Status Polling | Planned |
| Download Confirmation | Planned |

### Scope v2

- XAdES potpis unsigned XML-a unutar ERP-a (Finin cert / HSM),
- direktna predaja na ePoreznu G2B (SOAP),
- povrat `eporezna_identifier` u `VATReturn`.

### Pristup

Novi servis `accounting/services/tax_forms/pdv/eporezna/`:
- `sign_pdv_xml(unsigned_bytes, certificate) → signed_bytes`
- `submit_pdv_return(vat_return) → eporezna_id`

Koristi postojeće:
- `create_vat_return_draft` → unsigned XML,
- `import_signed_vat_return` → arhiva nakon predaje (ili proširenje za auto-import odgovora).

Potpis u v1 ostaje **izvan ERP-a** — integracija ne mijenja jezgreni pipeline. Nema službenog REST kanala PU — vidi matricu §1 (REST = rezervirano/TBD).

---

## 4. Napredni reconciliation

### Scope

- Batch usporedba više razdoblja,
- filtriranje diffa (`is_structural` / `is_value_difference`),
- export diffa u CSV/PDF za računovođu,
- upozorenje kad ERP nakon predaje ima nova knjiženja u razdoblju.

### Pristup

Reuse postojećeg:
- `compare_pdv_payload_fields`,
- `PdvFieldDifference`, `PayloadMismatchError.summary()`.

Novi:
- mgmt command `reconcile_pdv_periods --tenant --from --to`,
- admin dashboard (read-only, bez nove poslovne logike).

---

## 5. REST API

### Scope

Tenant-scoped endpointi za vanjski pristup (npr. mobilni potpis, integracija s računovodstvenim servisom).

### Predloženi endpointi

| Metoda | Put | Servis |
|--------|-----|--------|
| POST | `/api/v1/vat-periods/{id}/generate-ledger/` | `generate_vat_ledger` |
| POST | `/api/v1/vat-periods/{id}/draft/` | `create_vat_return_draft` |
| GET | `/api/v1/vat-periods/{id}/draft/{version}/xml/` | download unsigned |
| POST | `/api/v1/vat-periods/{id}/draft/{version}/submit/` | `import_signed_vat_return` |
| GET | `/api/v1/vat-periods/{id}/reconciliation/` | `compare_pdv_payload_fields` JSON |

API **ne smije** imati vlastitu XML/payload logiku — samo poziv servisa + serializacija `PdvFieldDifference`.

---

## Redoslijed implementacije (preporuka)

| Prioritet | Proširenje | Ovisi o |
|-----------|------------|---------|
| 1 | Boxovi 610+ (EU/uvoz) | Stabilizacija ✅ |
| 2 | Napredni reconciliation | Stabilizacija ✅ |
| 3 | REST API (read + draft + upload) | Stabilizacija ✅ |
| 4 | ePorezna potpis + predaja | Cert infrastruktura |
| 5 | PDV-K | Boxovi 610+ (preklapanje izvora) |

---

## Što je eksplicitno van scopea

- Promjena `VAT_BOX_REGISTRY` SSOT uzorka
- Agregacija po `vat_rate` umjesto `vat_box`
- Poslovna logika u XML rendereru
- `VATReturn` 1:1 s `VATPeriod`
- Potpis u jezgri v1 (ostaje van ERP-a do ePorezna integracije)

## Reference

- [`EPOREZNA_SUBMISSION_MATRIX.md`](../tax/EPOREZNA_SUBMISSION_MATRIX.md)
- [`ADR-0007-pdv-module.md`](../architecture/ADR-0007-pdv-module.md)
- [`pdv-obrazac-architecture.md`](pdv-obrazac-architecture.md)
- [`pdv-stabilization-runbook.md`](pdv-stabilization-runbook.md)

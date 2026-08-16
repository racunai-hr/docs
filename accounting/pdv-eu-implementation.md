# PDV EU boxovi 610–615 — implementacija

Kratak pregled što je u repozitoriju implementirano, kako se testira i što još nije zatvoreno.

Poslovna semantika (SSOT): [`pdv-eu-source-matrix.yaml`](pdv-eu-source-matrix.yaml) (`status: Locked`).

---

## Što je implementirano

### Registry i build (PR-B)

- [`boxes.py`](../../erp/app/accounting/services/tax_forms/pdv/boxes.py): `value_source` uz `field_type`; registry je jedini SSOT (Model A — eksplicitni `value_source=` u svakom `_box(...)`).
- [`build.py`](../../erp/app/accounting/services/tax_forms/pdv/build.py): generički dispatch po `value_source` (nema hardkoda po kodu boxa).
- [`mapping.py`](../../erp/app/accounting/services/tax_forms/pdv/mapping.py): `PDV_MAPPING_VERSION = 2`.

### Mapping i ledger (PR-C)

- Boxovi **610–615** su `implemented=True` s `mapping_rule` u registryju.
- [`mapping.py`](../../erp/app/accounting/services/tax_forms/pdv/mapping.py): `_MAPPING_RULES`, `journal_line_to_box()`, `is_eu_supplier()`, EU konta (`14022`/`24022` dobra, `14032`/`24032` usluge).
- [`vat.py`](../../erp/app/accounting/services/vat.py): proširen `generate_vat_ledger()` — EU goods asset (0373), reverse-charge JE linije, preskakanje EU troškova bez PDV-a; domaći flow (303) netaknut.

### Aktivni boxovi

| Box | Tip | `value_source` | Implementiran | Namjena |
|-----|-----|----------------|---------------|---------|
| 201–203 | pair | PAIR | da | Domaći izlaz po stopi |
| 303 | pair | PAIR | da | Domaći ulaz (troškovi) |
| 610 | scalar | AGGREGATE_BASE | da | EU stjecanje dobara — osnovica |
| 611 | scalar | AGGREGATE_VAT | da | EU stjecanje dobara — PDV |
| 612 | scalar | AGGREGATE_BASE | da | EU usluge — osnovica |
| 613 | scalar | AGGREGATE_VAT | da | EU usluge — PDV |
| 614 | scalar | AGGREGATE_BASE | da | EU dobra — obveza (RC) |
| 615 | scalar | AGGREGATE_VAT | da | EU dobra — pretporez (RC) |
| 400 | scalar | COMPUTED_VAT_DUE | da | Obveza / pretporez razlika |
| 660 | bool | BOOLEAN_NO_OUTPUT | da | Nema isporuka u razdoblju |

Ostali boxovi su u registryju s `value_source=ZERO` (eksplicitno) dok se ne implementiraju.

---

## Poznate granice

- **304** = uvoz, ne EU usluge; **612/613** su EU usluge, ne uvoz.
- EU trošak s `tax_amount = 0` i EU dobavljačem ne ide u 303 — očekuje se manual JE / reverse charge.
- Ledger za 610 koristi osnovicu s konta imovine (npr. 0373); 611/614/615 iz RC knjiženja na 14022/24022.
- Source Matrix (`EU_GOODS_ACQUISITION`, `EU_SERVICES_RECEIVED`) opisuje poslovne događaje; kod mapira na boxove preko `_MAPPING_RULES`.

---

## Checkpoint strategija

| Test | Uloga | Status svibanj 2026 |
|------|-------|---------------------|
| [`test_pdv_checkpoint_april_2026.py`](../../erp/app/accounting/tests/test_pdv_checkpoint_april_2026.py) | Submitted gate (potpisani ePorezna XML) | ✅ travanj |
| [`test_pdv_checkpoint_may_2026_synthetic.py`](../../erp/app/accounting/tests/test_pdv_checkpoint_may_2026_synthetic.py) | **Ključni dokaz** EU logike (610–615 nenulti) | ✅ aktivan gate |
| [`test_pdv_checkpoint_may_2026_reference.py`](../../erp/app/accounting/tests/test_pdv_checkpoint_may_2026_reference.py) | ERP == referentni XML (domaći 303/400) | ✅ aktivan; **nije** EU submitted gate |
| PR-D submitted gate (610–615 nenulti) | ERP == stvarno predani ePorezna XML | ⏳ **otvoreno** |

### Referentni XML — svibanj 2026

| Datoteka | Vrsta | Potpis ePorezna | EU 610–615 |
|----------|-------|-----------------|------------|
| [`PDV_36619131370_20260501-20260531.xml`](../../erp/app/accounting/tests/fixtures/pdv/archive/PDV_36619131370_20260501-20260531.xml) | **Referentni / draft export** iz ERP-a (2026-07-05, bez potpisa) | ne | svi **0.00** |
| (još ne postoji) | **Submitted** s nenultim EU boxovima | da | očekivano nenulti |

**Važno:** arhivski svibanj 2026 potvrđuje da ERP za Fine Star odgovara tom XML-u za domaće boxove (303, 400) i nulte EU boxove. To **ne dokazuje** EU funkcionalnost na stvarnim podacima — za to služi sintetički checkpoint.

### Kada zatvoriti PR-D

1. Prvi mjesec s **stvarno predanim** (potpisanim) ePorezna XML-om gdje su **610–615 nenulti**.
2. Arhivirati taj XML u `fixtures/pdv/archive/`.
3. Dodati `test_pdv_checkpoint_<period>_submitted.py` koji uspoređuje ERP s tim dokumentom.
4. Po potrebi preimenovati/ukloniti referentni svibanj fixture iz submitted semantike (ostaje u archive regression za parse/XSD).

---

## Testovi (pregled)

| Datoteka | Što pokriva |
|----------|-------------|
| `test_pdv_eu_source_matrix.py` | YAML Locked, sync s kodom |
| `test_pdv_eu_mapping.py` | Mapping pravila 610–615 |
| `test_pdv_eu_ledger.py` | `generate_vat_ledger` EU |
| `test_pdv_eu_invariants.py` | 611=614=615, RC invarianti |
| `test_vat_box_registry.py` | Eksplicitni `value_source`, validator |
| `test_pdv_submitted_archive_regression.py` | Parse/XSD/canonical za archive 01–05/2026 |
| `test_pdv_checkpoint_april_2026.py` | Behavior preserving gate (01–04 arhiva) |

---

## Behavior preserving (PR-B)

Za razdoblja **01–04/2026** očekivana je jedina semantička razlika: `PDV_MAPPING_VERSION = 2`. Poslovna polja (`aggregate_vat_boxes`, `PdvPayload`, renderirani XML) ostaju identična referentnim submitted fixtureima.

---

## Povezani dokumenti

- [`pdv-obrazac-architecture.md`](pdv-obrazac-architecture.md) — pipeline i checkpoint pravilo
- [`pdv-stabilization-runbook.md`](pdv-stabilization-runbook.md) — operativni koraci
- [`architecture/ADR-0007-pdv-module.md`](../architecture/ADR-0007-pdv-module.md) — slojevi registry / mapping / ledger

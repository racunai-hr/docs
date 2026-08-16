# Tax forms — testing guide

```text
Status: Living document
Last updated: 2026-07-09
Applies to: all NEW tax forms (ZP, PDV-K, JOPPD, PD, …)
Related: FORM_IMPLEMENTATION_CONVENTION.md, FORM_REGISTRY.md
```

Obvezni testni paket za **svaki novi porezni obrazac**. Cilj: isti standard kvalitete bez redefiniranja po obrascu.

**Napomena:** PDV koristi frozen test suite (`accounting/tests/test_pdv_*`, CI `pdv-ci.yml`) — ne migrirati retroaktivno osim ako ADR-0007 eksplicitno dopušta.

---

## Prije implementacije

| # | Test / artefakt | Datoteka / lokacija | Svrha |
|---|-----------------|---------------------|-------|
| 0 | **Fixtures** | `accounting/tests/fixtures/tax/<form>/` | Scenariji prije koda — vidi [`tax/zp/README.md`](../../erp/app/accounting/tests/fixtures/tax/zp/README.md) |
| 1 | **Fixture contract** | `test_<form>_fixture_contract.py` | JSON struktura, zbrojevi, cross-check metadata |

Contract testovi ne zahtijevaju DB niti implementaciju pipelinea.

---

## Obvezni testovi (nakon implementacije pipelinea)

| # | Test | Što provjerava | Minimalni assert |
|---|------|----------------|------------------|
| 1 | **Fixture contract** | Svi scenariji u `scenarios.yaml` | Loader + struktura |
| 2 | **Aggregate** | `aggregate_*_rows(period)` | Izlaz = `expected_payload.json` po scenariju |
| 3 | **Build gate** | `build_*_payload(period)` | Jedan poziv agregacije; determinizam |
| 4 | **Snapshot** | Canonical payload | `expected_payload_*.json` match |
| 5 | **Round-trip** | payload → render → parse | Polja ≈ original (semantički) |
| 6 | **XSD validation** | `validate_*_xml()` | Generirani unsigned XML prolazi lokalni XSD |
| 7 | **Business validation** | Poslovna pravila | OIB, razdoblje, negativni scenariji iz fixtures |
| 8 | **Cross-form verification** | `verify.py` | npr. ZP ↔ PDV 101/103; fail na mismatch fixtureu |
| 9 | **Submission lifecycle** | `submit.py` + `SubmissionService` | create_event, supersede, payload_hash |
| 10 | **Canonical payload hash** | `canonical.py` | Stabilan hash; event.payload_hash = hash drafta |

---

## Preporučena imena datoteka

```text
accounting/tests/
    fixtures/tax/<form>/
    test_<form>_fixture_contract.py
    test_<form>_aggregate.py
    test_<form>_build.py
    test_<form>_payload_snapshot.py
    test_<form>_render_parse_roundtrip.py
    test_<form>_xsd_validation.py
    test_<form>_business_validation.py
    test_<form>_verify.py              # cross-form / integrity
    test_<form>_submission.py
    test_<form>_canonical_hash.py
```

Manji obrasci mogu spojiti snapshot + round-trip u jednu datoteku — ali svi **tipovi** testova moraju postojati.

---

## Scenariji u fixtures (minimum)

| Scenarij | Obavezno |
|----------|----------|
| Jednostavan slučaj (1 red / 1 partner) | da |
| Više partnera / redova | da |
| Prazno razdoblje | da |
| Ispravak razdoblja (v1 → v2) | ako obrazac podržava supersede |
| Neispravan ulaz (validation error) | da |
| Cross-form mismatch | ako postoji veza s drugim obrazcem |

Indeks u `scenarios.yaml`; loader u `load.py`.

---

## Cross-form verification

Kad obrazac dijeli izvor s PDV-om ili drugim obrazcem:

1. Fixture `pdv_cross_check.json` s `must_match_zp_totals: true/false`
2. Test u `test_<form>_verify.py` poziva `verify_*_against_*()`
3. Mismatch scenarij **mora** failati

Referenca: [`test_zp_verify.py`](../../erp/app/accounting/tests/test_zp_verify.py), [`verify.py`](../../erp/app/accounting/services/tax_forms/zp/verify.py).

---

## Submission testovi

Delegacija na frozen `SubmissionService` (ADR-0009):

- `create_event` s `payload_hash`
- `supersede` lanac (ispravak)
- `attach_confirmation` (XML hash match)
- **Ne** testirati vlastiti audit model po obrascu

Referenca: `test_mark_vat_return_submitted.py`, `test_submission_service.py`.

---

## Canonical hash

```text
build_*_payload → canonical_json → SHA-256 → payload_hash
```

Test:

1. Isti payload → isti hash
2. Promjena jednog polja → drugi hash
3. `SubmissionEvent.payload_hash` = hash predanog sadržaja

---

## CI

| Obrazac | Workflow (preporuka) |
|---------|---------------------|
| PDV | `.github/workflows/pdv-ci.yml` (frozen) |
| ZP | proširiti `pdv-ci.yml` ili `tax-forms-ci.yml` na `tax_forms/zp/` |
| Novi obrazac | dodati path u tax-forms CI |

Pokretanje (Docker, `--keepdb`):

```bash
docker compose exec -T django python manage.py test accounting.tests.test_<form>_fixture_contract --keepdb -v2
docker compose exec -T django python manage.py test accounting.tests.test_<form> --keepdb -v2
```

---

## Checklist pri mergeu novog obrasca

- [ ] Fixtures + contract test (prije ili uz prvi PR pipelinea)
- [ ] Svi obavezni tipovi testova (tablica gore)
- [ ] `FORM_REGISTRY.md` maturity ažuriran
- [ ] `<FORM>_ARCHITECTURE.md` test sekcija + link ovdje
- [ ] Snapshot fixturei u `accounting/tests/fixtures/<form>/`

---

## Reference

- [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md) — layout koda
- [`FORM_REGISTRY.md`](FORM_REGISTRY.md) — SSOT obrazaca
- [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md) — operativni doc (ZP)
- [`README.md`](README.md) — Tax domena + infrastructure freeze

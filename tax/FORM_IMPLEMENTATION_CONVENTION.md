# Tax form implementation convention — racunAI ERP

```text
Status: Active (Sprint 3+)
Applies to: all NEW tax forms (ZP, JOPPD, PD, …)
Does NOT apply to: PDV (frozen ADR-0007), PDV-S (migrate opportunistically)

Infrastructure freeze: Sprint 3 — do not change directory layout or conventions;
  see docs/tax/README.md
Test quality standard: docs/tax/TESTING_GUIDE.md
```

Svaki **novi** porezni obrazac koristi **identičnu** strukturu koda i test fixturea.

---

## 1. Kod — `accounting/services/tax_forms/<form>/`

```text
accounting/services/tax_forms/<form>/
    __init__.py           # public exports
    payload.py            # frozen dataclasses, SCHEMA_VERSION, MAPPING_VERSION
    aggregate.py          # agregacija iz izvora (VATLedger, Builder, …)
    build.py              # build_<form>_payload(period) — arhitekturni gate
    render.py             # render_<form>_xml(payload) → bytes
    parse.py              # parse_<form>_xml(bytes) → payload
    validation.py         # XSD + formalna validacija
    verify.py             # cross-form, integrity, diff s predanim XML-om
    canonical.py          # canonical_json, payload_hash (preporučeno)
    submit.py             # mark_<form>_submitted → SubmissionService
    <form>_returns.py     # create_<form>_return_draft, lifecycle (preporučeno)
    mapping.py            # mapping rules / registry (ako treba)
```

### Obavezni moduli (minimum)

| Modul | Odgovornost |
|-------|-------------|
| `payload.py` | Kanonski poslovni model (`@dataclass(frozen=True)`) |
| `aggregate.py` | Jedan prolaz kroz izvor → raw agregat |
| `build.py` | **Jednom** gradi payload; contract test na gate |
| `render.py` | XML iz payloada (izvedeni artefakt) |
| `parse.py` | Round-trip / import arhive |
| `validation.py` | XSD prije persistencije drafta |
| `verify.py` | Usporedba s PDV / drugim obrascima / predanim XML-om |

### Pravila

1. **`payload.json` je SSOT** — vidi Canonical Artifact Rule u [`FORM_REGISTRY.md`](FORM_REGISTRY.md)
2. **`build_*_payload()`** — jedini ulaz za generiranje; poziva agregaciju **točno jednom**
3. **`submit.py`** — delegira na `SubmissionService` (ADR-0009); nema vlastitog audit modela
4. **`verify.py`** — per-form verification (ne bank/GL reconciliation)
5. XSD u `accounting/schemas/<form>/`

**XSD integritet:** službene sheme **ne uređivati** — original s portala PU; kompatibilnost u Pythonu. Vidi `accounting/schemas/zp/README.md`.

### Legacy iznimke

| Obrazac | Napomena |
|---------|----------|
| PDV | Frozen; `diff.py` + `integrity.py` umjesto `verify.py` |
| PDV-S | Nema `build.py` / `verify.py` — ne refaktorirati u Sprint 3 |

---

## 2. Test fixtures — `accounting/tests/fixtures/tax/<form>/`

```text
accounting/tests/fixtures/tax/<form>/
    README.md
    scenarios.yaml
    load.py
    <scenario_id>/
        ledger_entries.json       # ili builder input
        expected_payload.json
        pdv_cross_check.json      # cross-form (ako relevantno)
        expected_validation.json  # negativni scenariji
```

**Fixtures prije implementacije** — pripremiti scenarije prije pisanja agregacije.

Preporučeni scenariji (minimum):

- jednostavan slučaj (1 partner / 1 red)
- više partnera
- prazno razdoblje
- ispravak razdoblja (v1 / v2)
- neispravan ulaz (validation error)
- cross-form mismatch (verify fail)

Contract test: `test_<form>_fixture_contract.py`

Puni testni paket: [`TESTING_GUIDE.md`](TESTING_GUIDE.md).

---

## 3. Dokumentacija — `docs/tax/<FORM>_ARCHITECTURE.md`

Operativni doc (ne ADR) po uzorku [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md):

- izvori podataka
- payload / mapping
- validation / verify
- render / parse / submission
- test strategija + link na fixtures

Ažurirati [`FORM_REGISTRY.md`](FORM_REGISTRY.md) maturity nakon implementacije.

---

## 4. Implementacijski redoslijed (ZP — referenca)

1. Import službene XSD → `accounting/schemas/zp/`
2. `tax_forms/zp/` — payload → aggregate → build → render → parse
3. validation → verify
4. Snapshot testovi (fixtures)
5. Submission (`ZPReturn` + `SubmissionService`)
6. Cross-form testovi (PDV 101/103)

Tek nakon kompletnog ZP lifecyclea: PDV 610+ → reverse charge → OSS → ADR-0014 ✅ (Sprint 3 zatvoren).

---

## 5. TaxFormEngine

Novi obrazac implementira [`domains/tax/forms/protocol.py`](../../erp/app/domains/tax/forms/protocol.py) konvenciju.

Mapiranje modula → protocol:

| Protocol | Modul |
|----------|-------|
| `generate()` | `build.py` + `<form>_returns.py` |
| `validate()` | `validation.py` |
| `render_xml()` | `render.py` |
| `parse()` | `parse.py` |
| submit / history | `submit.py` → `SubmissionService` |

---

## Reference

- [`TESTING_GUIDE.md`](TESTING_GUIDE.md)
- [`FORM_REGISTRY.md`](FORM_REGISTRY.md)
- [`README.md`](README.md)
- [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md)
- ADR-0007, ADR-0009, ADR-0010

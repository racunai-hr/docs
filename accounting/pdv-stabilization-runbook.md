# PDV modul — stabilizacija (runbook)

Operativni runbook za fazu stabilizacije nakon zamrzavanja jezgre. Arhitektura: [`pdv-obrazac-architecture.md`](pdv-obrazac-architecture.md) · ADR: [`ADR-0007-pdv-module.md`](../architecture/ADR-0007-pdv-module.md).

## Cilj faze

Potvrditi ispravnost modula u stvarnoj uporabi prije proširenja (boxovi 610+, PDV-K, ePorezna API). **Jezgra se ne mijenja** — samo operativno praćenje, regresija, dokumentacija i performanse.

## Produkcijski tenanti

### Fine Star (primarni)

| Stavka | Vrijednost |
|--------|------------|
| Tenant slug | `finestar` |
| OIB | `36619131370` |
| Porezna ispostava | `3566` (Šibenik) |
| Referentna arhiva | `accounting/tests/fixtures/pdv/archive/` (01–04/2026) |
| Gate razdoblje | travanj 2026 ✅ (Mini-checkpoint 2.5) |

### Checklist nakon svakog zatvorenog razdoblja

1. **Knjiženje** — svi računi, troškovi i relevantne temeljnice u razdoblju.
2. **Generiraj knjige** — admin akcija ili `generate_vat_ledger --replace`.
3. **Generiraj draft** — provjeri da XSD validacija prođe.
4. **verify_pdv_period** — usporedba s predanim XML-om (vidi dolje).
5. **Predaja** — potpis + ePorezna + upload potpisani XML.
6. **Reconciliation** — admin view nakon uploada; mora biti „Nema razlika”.
7. **Zabilježi** u tablici praćenja (ispod).

### Tablica praćenja (Fine Star)

| Razdoblje | Knjige | Draft v | Predano | verify OK | Reconciliation | Bilješka |
|-----------|--------|---------|---------|-----------|----------------|----------|
| 2026-01 | — | arhiva | ✅ import | — | — | arhivski import |
| 2026-02 | — | arhiva | ✅ import | — | — | arhivski import |
| 2026-03 | — | arhiva | ✅ import | — | — | arhivski import |
| 2026-04 | ✅ | ✅ | ✅ | ✅ gate | — | referentni gate |
| 2026-05+ | | | | | | popuniti nakon zatvaranja |

### Novi tenant

Prije prvog PDV obrasca provjeri:
- `CompanySettings`: `street`, `house_number`, `postal_code`, `city`, `tax_office`
- `provision_*` command postavlja TaxOffice (ne smije ostati `NULL` bez namjere)
- Probni draft na praznom razdoblju prolazi XSD validaciju

## Regresija nakon mjesečnog obračuna

### Checkpoint pravilo (sintetički → submitted)

Ako referentni submitted XML još ne postoji jer se funkcionalnost prvi put koristi za tekuće razdoblje, koristi se **privremeni sintetički checkpoint** (`test_pdv_checkpoint_<period>_synthetic.py`). Test seed-a reprezentativna knjiženja u testnoj bazi i provjerava očekivane box vrijednosti, invarijante te da travanj (ili drugi postojeći submitted gate) ostaje nepromijenjen.

Nakon prve uspješne predaje:

1. Preuzmi potpisani XML s ePorezne i dodaj u `accounting/tests/fixtures/pdv/archive/`.
2. Zamijeni sintetički gate regresijskim testom (`test_pdv_checkpoint_<period>_submitted.py`) koji uspoređuje ERP izlaz sa stvarno predanim XML-om.
3. Sintetički fixture može ostati kao unit test — nije obavezan gate, ali korisno za izoliranu provjeru poslovne logike.

**Operativni redoslijed za novo razdoblje (npr. svibanj 2026 + boxovi 610–615):**

| Faza | Gate | Akcija |
|------|------|--------|
| Prije predaje | Sintetički | Merge PR-a s `test_pdv_checkpoint_may_2026_synthetic.py`; travanj gate mora prolaziti |
| Nakon ERP exporta (opcionalno) | Referentni | Arhiviraj unsigned XML, dodaj `test_pdv_checkpoint_may_2026_reference.py` — **ne zamjenjuje** sintetički EU gate |
| Nakon stvarne ePorezna predaje s nenultim EU boxovima | **Submitted (PR-D)** | Arhiviraj **potpisani** XML, dodaj `test_pdv_checkpoint_<period>_submitted.py`; sintetički može ostati kao unit |

**Stanje svibanj 2026 (Fine Star):** referentni fixture postoji (`PDV_36619131370_20260501-20260531.xml`, unsigned, EU 610–615 = 0). Sintetički checkpoint je ključni dokaz EU logike. PR-D submitted gate **nije zatvoren** — vidi [`pdv-eu-implementation.md`](pdv-eu-implementation.md).

### Automatski (CI)

Workflow `.github/workflows/pdv-ci.yml` pokreće cijeli PDV test suite na svaku promjenu u `accounting/services/tax_forms/pdv/` i povezanim testovima.

### Ručno — verify_pdv_period

```bash
cd /opt/stacks/racunai.hr/erp

# Usporedba ERP payloada s predanim XML-om
docker compose exec django python manage.py verify_pdv_period \
  --tenant finestar --year 2026 --month 5 \
  --xml /path/to/PDV_36619131370_20260501-20260531.xml

# Regeneriraj knjige prije provjere
docker compose exec django python manage.py verify_pdv_period \
  --tenant finestar --year 2026 --month 5 \
  --regenerate-ledger \
  --xml /path/to/submitted.xml
```

**Exit code 0** = implementirani boxovi (201, 202, 203, 303, 400) podudaraju se.  
**Exit code 1** = razlike — ispis tablice polja; ne predaj dok se ne riješi.

### Usporedba s arhivom u repou

Test `test_pdv_submitted_archive_regression.py` parsira sve XML-ove u `fixtures/pdv/archive/`:
- XSD validacija (signed),
- deterministički canonical JSON,
- konzistentnost implementiranih boxova.

Nakon svakog novog predanog razdoblja **dodaj XML u arhivu** i ažuriraj test ako se očekivani boxovi mijenjaju. Ako je razdoblje prethodno imalo sintetički checkpoint, zamijeni ga submitted testom prema pravilu iznad.

### Kriterij prijelaza na proširenja

- Min. **2 uzastopna produkcijska razdoblja** (Fine Star) s uspješnim verify + upload bez mismatch-a.
- Nema otvorenih P1 bugova na PDV workflowu.
- Performanse unutar baselinea (vidi dolje).

## Performanse

### Baseline (test_pdv_performance.py)

Smoke test na sintetičkom tenantu s ~500 stavki knjige:

| Korak | Cilj (p95) |
|-------|------------|
| `generate_vat_ledger` | < 5 s |
| `aggregate_vat_boxes` | < 0,5 s |
| `build_pdv_payload` | < 0,1 s |
| `render_pdv_obrazac_xml` | < 0,5 s |
| Cijeli pipeline | < 10 s |

Pokretanje:

```bash
docker compose exec django python -m pytest \
  accounting/tests/test_pdv_performance.py -v
```

### Produkcijsko mjerenje

```bash
docker compose exec django python manage.py verify_pdv_period \
  --tenant finestar --year 2026 --month 4 --benchmark
```

Ispisuje trajanje koraka na stvarnom tenantu. Zabilježi u tablicu praćenja ako > 2× baseline.

### Optimizacija (samo ako baseline padne)

Dozvoljeno **bez promjene jezgre**:
- DB indeksi na `VATLedgerEntry(vat_period, vat_box)`
- `select_related` / bulk u `generate_vat_ledger`
- cache TaxOffice / CompanySettings u renderu

Zabranjeno: preskočiti agregaciju, računati boxove u rendereru, duplicirati poslovnu logiku.

## Korisnička dokumentacija

| Dokument | Publika |
|----------|---------|
| [`pdv-accountant-workflow.md`](pdv-accountant-workflow.md) | Računovođe |
| [`pdv-mapping.md`](pdv-mapping.md) | Developeri / revizija |
| [`pdv-obrazac-architecture.md`](pdv-obrazac-architecture.md) | Developeri |

## Bugfixovi iz produkcije

1. Reproduciraj s `verify_pdv_period` + fixture podacima.
2. Fix **samo** u mapping/ledger sloju ili admin workflowu — ne refaktorirati pipeline.
3. Ako fix mijenja izlazni payload → inkrement `PDV_MAPPING_VERSION`, ažuriraj fixture.
4. Dodaj regresijski test.

## Sljedeći korak

Kad su kriteriji ispunjeni → [`pdv-extensions-roadmap.md`](pdv-extensions-roadmap.md).

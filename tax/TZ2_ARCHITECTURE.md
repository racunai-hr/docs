# Obrazac TZ 2 — arhitektura (turistička članarina)

```text
Status: Implemented (L2 — Gen/Parse/Manual Sub)
Version: 0.1
Last updated: 2026-09-15

Registry: docs/tax/FORM_REGISTRY.md
Konvencija: docs/tax/FORM_IMPLEMENTATION_CONVENTION.md
ADR: docs/architecture/ADR-0031-tz2-return.md
XSD: api/app/accounting/schemas/tz2/
```

## Svrha

**Obrazac TZ 2** prijavljuje paušalni iznos turističke članarine za iznajmljivače. Godišnji obrazac (`DatumOd`–`DatumDo` = kalendarska godina). Predaja: unsigned XML + ručni portal ePorezna; audit `SubmissionEvent` (PDF potvrda).

TZ2 **nije** PDV. Izvor **nije** `VATLedger`. UI ePorezne **nije** XML shema.

## Pipeline

```text
CompanySettings + ručni unos (Ime/Prezime, kreveti, stope, plaćanje, EP)
  → aggregate_tz2 / build_tz2_payload
  → Tz2Payload (SSOT)
  → render_tz2_xml / parse_tz2_xml
  → TZ2Return
  → mark_tz2_submitted → SubmissionService.create_event
```

## Mapiranje `Podatak01`–`36`

Aritmetika: službeni `primjer.xml` (ne anotacija „r. br. 1–6“ na `Podatak25`).

| XML | Papir / XSD dokumentacija | Tip |
|-----|---------------------------|-----|
| 01 / 02 / 03 | Krevet u sobi/apartmanu/kući za odmor — BROJ / IZNOS / UKUPNO | int / dec / dec |
| 04 / 05 / 06 | Pomoćni krevet — BROJ / IZNOS / UKUPNO | |
| 07 / 08 / 09 | Kamp — BROJ / IZNOS / UKUPNO | |
| 10 / 11 / 12 | Robinzon — BROJ / IZNOS / UKUPNO | |
| 13 / 14 / 15 | Krevet na OPG-u | |
| 16 / 17 / 18 | Pomoćni na OPG-u | |
| 19 / 20 / 21 | Kamp na OPG-u | |
| 22 / 23 / 24 | Robinzon na OPG-u | |
| 25 | Ukupno obračunani iznos = 03+06+09+12+15+18+21+24 | dec |
| 26–29 | Umanjenja I.–IV. razvojne skupine | dec |
| 30 | Ukupno umanjenje = 26+27+28+29 | dec |
| 31 | Nakon umanjenja = 25 − 30 | dec |
| 32 / 33 | Jednokratna uplata DA/NE (`0\|1`) + iznos | |
| 34 / 35 | Obročno DA/NE + iznos (`31 / 3`) | |
| 36 | Primitak iz Obrasca EP (prethodna godina) | dec |

`UKUPNO = BROJ × IZNOS` (2 decimale). `PodrucniUred` / `Ispostava` su komentirani u XSD — ne šalju se.

**4 apartmana nisu XML polje.**

## Stope (`mapping_version = 1`)

| Kategorija | 2026 |
|------------|------|
| Krevet (nije OPG) | 5.97 |
| Pomoćni krevet | 2.99 |
| Krevet na OPG-u | 3.98 |
| Pomoćni krevet na OPG-u | 1.99 |

Kamp/robinzon default 0.00 dok se ne unese override. Stope žive u Python `mapping.py`, ne u UI.

## Alma 2026

| Polje | Vrijednost |
|-------|------------|
| Ime / Prezime | ALMA / ČIZMIĆ |
| OIB | 20501805574 |
| Šifra općine | 500 |
| Mjesto / Ulica / Broj | VODICE / ULICA BRIBIRSKIH KNEZOVA / 0009 |
| 01 / 04 | 10 / 0 |
| 36 | 17360.00 |
| Plaćanje | obročno (`34=1`, `35=31/3`) |
| Članarina | 10 × 5.97 = 59.70 → rata 19.90 |

Normalizacija `Broj` i `SifraOpcine` je u `build.py`, ne u UI. `company_name` se ne cijepa u `Ime`.

## Submission

`mark_tz2_submitted()` → `SubmissionService.create_event(..., destination=EPOREZNA)`. Potvrda: PDF/JPEG/PNG **ili** predani ObrazacTZ2 XML (`SubmissionService.validate()` → TZ2 XSD u `tax_forms/tz2`). Predaja **ne** locka `VATPeriod`.

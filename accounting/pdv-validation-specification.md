# PDV Validation Specification

```
Status: ACTIVE
Scope: Obrazac PDV-S v1.0 ↔ Obrazac PDV v11.0 (Fine Star, EU stjecanje)
Pravilo dokumenta: samo pravila — bez istraživačkog narativa
Frozen reference: pdv-pu-uputa-2026-citations.md, pdv-source-matrix-review-2026.md
```

Implementacijska specifikacija poslovnih provjera prije izvoza PDV-S / PDV XML-a. Svako pravilo **V00N** mapira se 1:1 na test `test_v00N_…` u `erp/app/accounting/tests/test_pdv_business_validation.py`.

## Severity

| Severity | Ponašanje validatora |
|---|---|
| **Fatal** | XML se ne smije generirati ni izvesti |
| **Warning** | XML se može izvesti; korisnik mora potvrditi |
| **Info** | Zapis u audit log; bez blokiranja |

## Sažetak pravila

| ID | Rule (sažetak) | Level | Severity | Test |
|---|---|---|---|---|
| V001 | Postoji PDV prijava za razdoblje PDV-S | C (+ A) | Fatal | `test_v001_requires_pdv_return` |
| V002 | PDV-S(13) ≤ II.5 + II.6 + II.7 | A + C | Fatal | `test_v002_pdvs_total_not_exceed_pdv` |
| V003 | Valjan EU PDV ID po redu | A | Fatal | `test_v003_valid_eu_vat_id` |
| V004 | Zbroj redaka = zaglavlje ukupno | A | Fatal | `test_v004_header_matches_rows` |
| V005 | PDV-S postoji kad ima EU stjecanja | ERP | Warning | `test_v005_missing_pdvs` |

**Poveznica s konfliktima:** V002 pokriva **C005** (validation risk). **C001–C004** su predmet `erp-fix-610-rule` (Implementation Gate), ne samo ovog validatora.

---

## V001

```yaml
ID: V001
Rule: >
  Za isto porezno razdoblje (godina/mjesec) mora postojati generirani Obrazac PDV
  (VATReturn status ≥ generated) prije izvoza ili predaje Obrazca PDV-S.
Evidence:
  - ePorezna observed (05/2026 Provjeri): "Za razdoblje podnošenja obrasca PDV-S ne postoji knjiženi obrazac PDV."
  - PU A-FAQ-4355 — PDV-S se uspoređuje s Prijavom PDV-a
Level: C (+ A kontekst)
Severity: Fatal
Test: test_v001_requires_pdv_return
```

**Provjera:** `VATReturn` za `(tenant, year, month)` PDV-S razdoblja postoji i nije prazan draft bez payloada.

---

## V002

```yaml
ID: V002
Rule: >
  PDV-S ukupno stečena dobra (polje 13 / IsporukeUkupno.I1) ≤
  zbroj osnovica PDV II.5 + II.6 + II.7 (XML Podatak205 + Podatak206 + Podatak207, vrijednost).
Evidence:
  - PU A-PDV-1488 §4 — "Zbroj vrijednosti stečenih dobara … u polju (13) … može biti jednak ili manji od zbroja … II.5., II.6. i II.7."
  - PU A-PDV-S-PDF poruka 013
  - ePorezna observed (05/2026 Provjeri): upozorenje kad II.7 = 0 a PDV-S I1 = 23.882,35 — vidi [`pdv-korak3-eporezna-test-2026.md`](pdv-korak3-eporezna-test-2026.md)
Level: A + C
Severity: Fatal
Test: test_v002_pdvs_total_not_exceed_pdv
```

**Provjera:** `pdv_s.total_goods ≤ pdv.base_205 + pdv.base_206 + pdv.base_207` (XML `Podatak205`–`207` / Vrijednost; zaokruženo na 2 decimale).

**Napomena implementacije:** usporedba koristi **osnovice** (Vrijednost u paru), ne iznose PDV-a (Porez). Službena semantika redova II.5–II.7 ≠ ERP registry labeli u `boxes.py`.

---

## V003

```yaml
ID: V003
Rule: >
  Svaki red Isporuka u PDV-S mora imati valjan EU PDV identifikator:
  KodDrzave iz službenog popisa EU država, PDVID neprazan (bez koda države),
  duljina PDVID ≤ 12; par (KodDrzave, PDVID) jedinstven unutar obrasca.
Evidence:
  - PU A-XSD-PDVS — tip tKodDrzave (ISO 3166-1 alpha-2 EU), tip tPDVID (maxLength 12)
  - PU A-PDV-S-PDF poruka 012 — isti PDV ID isporučitelja smije se pojaviti samo jednom
  - PU A-FAQ-4355 — usporedba s VIES sustavom
Level: A
Severity: Fatal
Test: test_v003_valid_eu_vat_id
```

**Provjera:** za svaki red — `country_code ∈ _EU_VAT_NUMBER_PREFIXES` (vidi `mapping.py`), `pdv_id` neprazan, `len(pdv_id) ≤ 12`; nema duplikata `(KodDrzave, PDVID)`.

---

## V004

```yaml
ID: V004
Rule: >
  IsporukeUkupno mora odgovarati zbroju redaka Isporuka:
  IsporukeUkupno.I1 = Σ Isporuka.I1;
  IsporukeUkupno.I2 = Σ Isporuka.I2.
Evidence:
  - PU A-XSD-PDVS — sIsporukeUkupno "sumarni podaci o isporukama svih isporučitelja"
  - PU A-XSD-PDVS — sIsporuka po jednom isporučitelju (PDVID, I1, I2)
Level: A
Severity: Fatal
Test: test_v004_header_matches_rows
```

**Provjera:** nakon agregacije ili pri parse/render PDV-S XML-a — totals moraju biti aritmetički točni (2 decimale).

---

## V005

```yaml
ID: V005
Rule: >
  Ako PDV razdoblje sadrži EU stjecanje dobara ili usluga (ledger stavke s
  vat_box ∈ {610, 612} i base_amount > 0, ili ekvivalent nakon erp-fix-610),
  mora postojati generirani Obrazac PDV-S za isto razdoblje prije izvoza PDV XML-a.
Evidence:
  - ERP interna politika konzistentnosti PDV ↔ PDV-S (nije porezni zakon)
  - Poslovna praksa iz A-FAQ-4355 — usklađenost izvještaja za isto razdoblje
Level: ERP
Severity: Warning
Test: test_v005_missing_pdvs
```

**Provjera:** ako `VATLedgerEntry` za razdoblje ima EU stjecanje, a PDV-S payload/XML ne postoji — **Warning** (ne Fatal). Politika tenant-a može eskalirati u **Fatal** (konfiguracija izvan ove specifikacije).

---

## Konvencija proširenja

Novo pravilo **V006** → dodati blok u ovaj dokument + `test_v006_…` u test suite. Severity mora biti Fatal, Warning ili Info. Level: **A–D** (dokazna metodologija) ili **ERP** (interna politika).

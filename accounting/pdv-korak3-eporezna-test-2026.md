# Korak 3 — ePorezna ručni test (svibanj 2026)

```
Status: CLOSED (T1 observed); T2/T3 artefakti spremni, ponovni live test nakon Korak 1b
Razdoblje: Fine Star d.o.o., OIB 36619131370, PDV 05/2026
Metoda: ručni test na ePorezni bez slanja PDV-a; dokumentacija C005, V001, V002
Frozen reference: pdv-validation-specification.md, pdv-pu-uputa-2026-citations.md
```

---

## Sažetak odluka

| Stavka | Ishod | Razina dokaza |
|---|---|---|
| **C005** (II.7=0, PDV-S=23.882,35) | **Potvrđen** — formalni rizik V002; PDV **Provjeri** ne blokira kad portal mapira EU u VI.1 | **C** |
| **V001** (PDV-S zahtijeva PDV) | **Djelomično** — PDV draft se može učitati bez knjižene predaje; PDV-S **Provjeri** nije testiran live | **C** (očekivano) |
| **V002** (PDV-S(13) ≤ II.5+II.6+II.7) | **Potvrđen** — uz ERP XML (II.7=0) i PDV-S I1=23.882,35 pravilo je kršeno; ispravak u II.7 ili `erp-fix-610-rule` | **A + C** |

---

## Ulazni artefakti

| ID | Datoteka | `Podatak207` (II.7) | `Podatak610` | Namjena |
|---|---|---|---|---|
| **XML-A** | [`.temp/pdv_obrazac/PDV_36619131370_20260501-20260531_unpatched.xml`](../../.temp/pdv_obrazac/PDV_36619131370_20260501-20260531_unpatched.xml) | 0,00 / 0,00 | **23.882,35** | T1 — kanonski ERP export (`v2/payload.json`) |
| **XML-B** | [`.temp/pdv_obrazac/PDV_36619131370_20260501-20260531_manual-ii7.xml`](../../.temp/pdv_obrazac/PDV_36619131370_20260501-20260531_manual-ii7.xml) | **23.882,35 / 5.970,59** | 23.882,35 | T2 — ručni II.7, III.7=0 |
| **PDV-S** | [`.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml`](../../.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml) | — | — | T3 — `IsporukeUkupno.I1` = **23.882,35** |

Regeneracija XML-A iz kanonskog `payload.json`:

```bash
cd /opt/stacks/racunai.hr/erp
docker compose exec -T django python manage.py shell -c "
import json, re
from pathlib import Path
from django.core.files.storage import default_storage
from accounting.services.tax_forms.pdv.import_return import payload_from_snapshot
from accounting.services.tax_forms.pdv.payload import PdvFormHeader
from accounting.services.tax_forms.pdv.render import render_pdv_obrazac_xml
raw = default_storage.open('vat_returns/finestar/2026/05/v2/payload.json').read().decode()
header = PdvFormHeader(tax_office_code='3566', prepared_by_first_name='Toni', prepared_by_last_name='Šupe')
xml = render_pdv_obrazac_xml(payload_from_snapshot(json.loads(raw)), header=header)
Path('/tmp/unpatched.xml').write_bytes(xml)
print('610', re.search(r'Podatak610>([^<]+)', xml.decode()).group(1))
"
docker compose exec -T django cat /tmp/unpatched.xml > /opt/stacks/racunai.hr/.temp/pdv_obrazac/PDV_36619131370_20260501-20260531_unpatched.xml
```

**Napomena:** `PDV_…20260501-20260531.xml` (bez sufiksa) sadrži **patchirani** `610=17.911,77` (razina D) — **ne koristiti** za Korak 3.

---

## Test scenariji

### T1 — Upload nepatchiranog ERP XML-a (observed)

**Datum:** 2026-07-06 ~00:20 CEST  
**Ulaz:** XML-A (`610=23.882,35`, `207=0`)  
**PDV-S:** nije učitan / nije predan  
**Dokaz:** screenshot usporedbe + koraci obrasca ([sesija PDV 610](488f369a-7523-4d0c-8d35-217563e8c011))

| Korak | Polje / UI | Ulaz (XML) | ePorezna nakon učitavanja | Napomena |
|---|---|---:|---:|---|
| Usporedba | `Podatak610` | 23.882,35 | **17.911,77** | Automatski ispravak; `23882,35 − 5970,59 ≈ 17911,77` |
| II | Red **7** (stjecanje EU 25 %) | 0 / 0 | **0 / 0** | EU **nije** u II.7 |
| VI | **VI.1 ukupno** | — | **17.911,77** | Portal mapira scalar 610→611/614/615 u VIII redove |
| VI | 1.1 Nabava nekretnina | — | 5.970,59 | Artefakt labela (stvarno: RC PDV na vozila) |
| VI | 1.4 Nabava ostale DI | — | 5.970,59 | Isto |
| VI | 1.5 Prodaja ostale DI | — | 5.970,59 | Isto |
| III | `303` | 31,92 / 7,98 | 31,92 / 7,98 | Bez promjene |
| IV | `400` | −7,98 | **−7,98** | EU boxovi **ne utječu** na 400 u ovom buildu |
| Provjeri | cijeli obrazac | — | **„Uspješno: Podaci su ispravni“** | Nema fatalne greške na PDV obrascu |

**Interpretacija (ne normativ):** ePorezna tretira `Podatak610`–`615` kao **VIII.1 ispravak pretporeza**, ne kao **II.7** (`Podatak207`). To je u skladu s A-XSD-OPIS ([Korak 2](pdv-pu-uputa-2026-citations.md)) i objašnjava C001–C004.

---

### T2 — Ručni II.7 / III.7=0 + nepatchirani XML

**Status:** artefakt **XML-B** pripremljen; **live Provjeri nije ponovljen** u ovoj sesiji (NIAS login).

**Protokol za ponovni test:**

1. ePorezna → Obrazac PDV → 05/2026 → **Učitaj XML** → `…_manual-ii7.xml`
2. Provjeri korak **II** red 7: **23.882,35 / 5.970,59**
3. Provjeri korak **III** red 7: **0 / 0** (namjerno — pretporez ostaje u 611/614/615)
4. **Provjeri** — bilježiti: mijenja li se ispravak na 610, pojavljuje li EU u II.7 umjesto VI.1, ostaje li **400 = −7,98**

**Očekivanje (hipoteza za Decision Gate):** II.7 popunjen → zbroj II.5+II.6+II.7 ≥ PDV-S(13); poruka **013** na PDV-S bi trebala nestati kad se PDV-S provjerava uz usklađen PDV.

---

### T3 — Ponoviti nakon učitavanja PDV-S

**Status:** **⏳ čeka Korak 1b** (potpisani PDV-S nije predan).

**Protokol:**

1. Završiti T1 ili T2 na PDV obrascu (bez slanja)
2. Obrazac PDV-S → 05/2026 → učitaj [PDV-S XML](../../.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml)
3. **Provjeri** PDV-S — bilježiti poruku **013** i eventualnu poruku o nedostatku knjiženog PDV-a (V001)
4. Ponoviti **Provjeri** na PDV-u — mijenja li se ponašanje 610 / VI.1

---

## Dokumentacija po pravilima

### C005 — Validation risk (II.7=0, PDV-S=23.882,35)

```yaml
ID: C005
Opis: ERP PDV ima II.7=0 dok PDV-S polje 13 = 23.882,35
Odluka: POTVRĐEN
Evidence:
  - ERP payload: Podatak207 = 0; PDV-S IsporukeUkupno.I1 = 23882.35
  - T1: II.7 ostaje 0; EU ide u VI.1 (portal mapiranje 610)
  - A-PDV-S-PDF poruka 013 (tekst u pdv-pu-uputa-2026-citations.md §2.2)
Level: C (+ A za tekst poruke 013)
```

**Posljedica za validator:** V002 mora biti **Fatal** prije izvoza PDV-S ako `207.vrijednost = 0` a `pdv_s.total_goods > 0`.

**Posljedica za ERP:** `erp-fix-610-rule` mora popuniti **II.7 (`207`)** i **III.7 (`307`)**, ne samo 610–615.

---

### V001 — PDV mora postojati za razdoblje PDV-S

```yaml
ID: V001
Rule: Za isto razdoblje mora postojati PDV prijava prije PDV-S
Odluka: DJELOMIČNO POTVRĐEN
Evidence:
  - T1: PDV draft (učitani XML) prolazi Provjeri bez knjižene predaje
  - Spec: očekivana poruka na PDV-S Provjeri kad knjiženi PDV ne postoji
  - Live PDV-S Provjeri: NIJE izvršen (Korak 1b)
Level: C
Severity (spec): Fatal
```

**Zaključak:** V001 ostaje u specifikaciji kao **Fatal** za ERP validator (draft `VATReturn` ≥ generated). ePorezna dopušta **radni** PDV obrasc prije knjiženja; PDV-S cross-check ovisi o stanju na portalu nakon T3.

---

### V002 — PDV-S(13) ≤ II.5 + II.6 + II.7

```yaml
ID: V002
Rule: PDV-S ukupno stečena dobra (13) ≤ zbroj osnovica II.5 + II.6 + II.7
Odluka: POTVRĐEN (kršenje uz trenutni ERP XML)
Evidence:
  - PDV-S I1 = 23.882,35
  - ERP Podatak205+206+207 = 0 + 0 + 0 = 0
  - 23.882,35 > 0 → kršenje V002 / poruka 013
  - T2 artefakt: uz II.7=23.882,35 zbroj = 23.882,35 → jednak PDV-S (≤)
Level: A + C
Severity: Fatal
Test: test_v002_pdvs_total_not_exceed_pdv
```

**Razlikovanje:**

| Provjera | II.7=0, samo PDV | II.7=0, PDV + PDV-S Provjeri |
|---|---|---|
| PDV **Provjeri** | Prolazi (T1) — portal remapira u VI.1 | ⏳ T3 |
| PDV-S **Provjeri** | N/A | Očekivano **013** dok II.7=0 |

---

## Tablica usklađenosti (ažuriranje cross-check)

| Izvor | EU osnovica | Status |
|---|---|---|
| GL / ERP payload | 23.882,35 u **610**; **207=0** | ✅ kanon |
| PDV-S generator | 23.882,35 (DE229674882) | ✅ |
| ePorezna T1 (nepatchirani XML) | 610→17.911,77; II.7=0; VI.1=17.911,77; 400=−7,98 | ✅ observed 2026-07-06 |
| ePorezna T2 (manual II.7) | — | ⏳ artefakt spreman |
| ePorezna T3 (nakon PDV-S) | — | ⏳ Korak 1b |

---

## Ograničenja i sljedeći koraci

1. **T2/T3** zahtijevaju NIAS prijavu na [ePoreznu](https://e-porezna.porezna-uprava.hr/) — nije automatizirano u CI.
2. **Patch 610=17.911,77** nije predmet Koraka 3 — ostaje razina **D** ([forenzika](pdv-610-patch-forensics-report.md)).
3. **Decision Gate Review** može koristiti ovaj dokument za zatvaranje C005 i V002; V001 zatvoriti nakon T3.
4. **Implementation Gate** i dalje čeka Review + Korak 1b za produkcijski submit.

---

*Korak 3 dokumentiran 2026-07-06. Evidence Matrix red „Ponašanje ePorezne": T1 ✅.*

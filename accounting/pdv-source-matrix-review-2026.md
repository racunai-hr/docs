# Source Matrix Review — ERP izvor vs službeni element (svibanj 2026)

```
Status: CLOSED (Source Matrix Review)
Decision Gate effect:
  ✓ Konflikt ERP builda vs službeni element (207/307/II.7/VIII.1/610) dokumentiran s dokazima razine A
  ✓ Rasprava o erp-fix-610-rule MOŽE se otvoriti (ne implementacija — još LOCKED)
  ✗ erp-fix-610-rule implementacija — LOCKED (čeka Korak 3 + cijeli Decision Gate)
  ✗ pdv-eu-source-matrix.yaml — nije mijenjan (Locked; računovođa)
Next step:
  PDV Validation Specification → Korak 3 (ePorezna test)
```

**Datum pregleda:** 2026-07-06  
**Predmet:** Fine Star d.o.o., OIB 36619131370, razdoblje **05/2026**  
**Službeni izvori (razina A):** [`pdv-pu-uputa-2026-citations.md`](pdv-pu-uputa-2026-citations.md) (Korak 2, zatvoren)  
**ERP kanonski build:** `erp/media/vat_returns/finestar/2026/05/v2/payload.json`

---

## 1. Referentni ERP build (činjenica, ne presuda)

| Stavka | Vrijednost |
|---|---|
| Datoteka | [`v2/payload.json`](../../media/vat_returns/finestar/2026/05/v2/payload.json) |
| SHA256 | `110037b453d6553b15618007dc781c264ba88ff97a72d375f99ed3a95b756605` |
| `mapping_version` | 2 |
| `schema_version` | 11.0 |
| Generacija | `build_pdv_payload` (Korak 0, 2026-07-06) |

**Napomena:** `v2/unsigned.xml` na disku sadrži ručno patchiran `Podatak610 = 17.911,77` (forenzika razina **D**). Ovaj pregled koristi **isključivo** nepatchirani `payload.json` kao ERP kanon.

### 1.1 Vrijednosti relevantnih polja (05/2026)

| XML element | ERP `payload.json` | Knjigovodstvo (0373 + RC) |
|---|---:|---:|
| `Podatak207` (II.7) | 0,00 / 0,00 | očekivano **23.882,35 / 5.970,59** |
| `Podatak307` (III.7) | 0,00 / 0,00 | očekivano **23.882,35 / 5.970,59** (pretporez dio) |
| `Podatak610` | **23.882,35** | — |
| `Podatak611` | 5.970,59 | RC PDV 25 % |
| `Podatak612`–`613` | 0,00 | — |
| `Podatak614` | 5.970,59 | obveza RC (K 24022) |
| `Podatak615` | 5.970,59 | pretporez RC (D 14022) |
| `Podatak701`–`704` (VI.1 margin) | 0,00 | — |
| PDV-S `I1` (DE229674882) | **23.882,35** (generator) | 23.882,35 |

---

## 2. Glavna matrica — ERP izvor vs službeni element

Legenda statusa:

| Simbol | Značenje |
|---|---|
| ✅ | Usklađeno za analizirani build i službeni izvor razine **A** |
| ❌ | Dokumentirani konflikt (činjenica konkretnog builda, ne trajna arhitekturna presuda) |
| ⏳ | Otvoreno / čeka Korak 3 ili računovođu |
| ◐ | Djelomično — izvor A postoji, ERP ili portal nije potvrđen |

| ERP izvor / ponašanje builda | Službeni element (razina **A**) | Službeni opis (citat) | Build 05/2026 | Status |
|---|---|---|---|---|
| EU stjecanje dobara 25 % — **nije** u paru II.7 | `Podatak207` / **II.7** | „vrijednost stečenih dobara unutar EU … po stopi **25%**" (A-PDV-2398, II 7) | `207` = **0,00 / 0,00** | ❌ **gap** — ERP ne puni službeni kanal glavne prijave |
| EU pretporez od stjecanja dobara 25 % — **nije** u paru III.7 | `Podatak307` / **III.7** | „pretporez … stečenih dobara unutar EU … stopi PDV-a **25%**" (A-PDV-2398, III 7) | `307` = **0,00 / 0,00** | ❌ **gap** — pretporez ide u scalar blok 611/614/615 |
| EU stjecanje dobara — osnovica u scalar **610** | `Podatak207` / **II.7** (glavna prijava) | II.7 je mjesto EU osnovice, ne 610 (A-PDV-2398) | `610` = **23.882,35** | ❌ **konflikt** — analizirani build stavlja EU osnovicu u element čija je službena semantika ispravak DI |
| Službena semantika **610** | `Podatak610` / **VIII.1** zbroj | „**ZA ISPRAVAK PRETPOREZA** (UKUPNO 1.1.+ 1.2.+ 1.3.+ 1.4.+ 1.5.)" (A-XSD-OPIS) | `610` = 23.882,35 (EU osnovica) | ❌ **konflikt** — vrijednost ne odgovara službenoj semantici elementa |
| Nabava ostale DI (614) | `Podatak614` / VIII.1 red **1.4** | „**1.4. Nabava ostale dugotrajne imovine**" (A-XSD-OPIS) | `614` = 5.970,59 (RC obveza, ne neto nabava) | ❌ **konflikt** — ERP koristi 614 za RC obvezu, ne za neto nabavu DI |
| Pretporez na stjecanje (615) | `Podatak615` / VIII.1 red **1.5** | „**1.5. Prodaja ostale dugotrajne imovine**" (A-XSD-OPIS) | `615` = 5.970,59 (RC pretporez) | ❌ **konflikt** — ERP koristi 615 za RC pretporez, ne prodaju DI |
| PDV na stjecanje (611) | `Podatak611` / VIII.1 red **1.1** | „**1.1. Nabava nekretnina**" (A-XSD-OPIS) | `611` = 5.970,59 | ❌ **konflikt** — ERP koristi 611 za izračun 25 % na 610, ne nabavu nekretnina |
| Ispravak DI / VIII.1 (neto nabave) | **VIII.1** + `701`–`704` (VI.1) | „samo **vrijednosni podaci (neto vrijednost)**" (A-PDV-2398, VIII) | svi **0,00** | ⏳ — rent-a-car vozila 0373; čl. 61 u knjigovodstvu ✅, VIII.1 sadržaj ⏳ računovođa |
| PDV-S agregacija po dobavljaču | PDV-S `PDVID` + `I1`, poruka **012** | „Isti PDV ID … smije se pojaviti samo **jednom**" (A-PDV-S-PDF) | 1 red DE229674882 = **23.882,35** | ✅ |
| Usklađenost PDV-S(13) ≤ II.5+II.6+**II.7** | A-PDV-1488, poruka **013** | PDV-S osnovica ≤ zbroj II.5–II.7 | II.7 u ERP = **0** → formalni rizik pri provjeri | ◐ — generator PDV-S ✅; ERP PDV II.7 gap može utjecati na portal provjeru (Korak 3) |
| Uvoz dobara | `Podatak215` / **II.15** | uvoz (A-PDV-2398) | 0,00 | ✅ (nema uvoza u svibnju) |

---

## 3. Usporedba s `pdv-eu-source-matrix.yaml` (Locked — nije mijenjan)

Strojno čitljivi izvor: [`pdv-eu-source-matrix.yaml`](pdv-eu-source-matrix.yaml) (`status: **Locked**`, `version: 1.0.0`, `last_reviewed: 2026-07-05`).

| YAML `event_id` | YAML boxovi | Službeni element (A) | Nesklad |
|---|---|---|---|
| `EU_GOODS_ACQUISITION` | `610`, `611`, `614`, `615` | `207`/`307` (II.7/III.7) + PDV-S `I1` | YAML i implementirani kod (`mapping.py` → `610`) odstupaju od A-XSD-OPIS i A-PDV-2398 |
| `EU_SERVICES_RECEIVED` | `612`, `613` | `210`/`310` (II.10/III.10) + PDV-S `I2` | Nije predmet svibnja 2026; ista klasa problema za buduće usluge |

**ERP invarijanti u YAML-u** (npr. `611 == round(610 × 0.25, 2)`, `611 == 614 == 615`) odražavaju **internu ERP politiku**, ne službenu formulu za `Podatak610` (= zbroj 611–615 za ispravak DI prema A-XSD-OPIS).

**Pravilo plana:** YAML se **ne mijenja** bez odobrenja računovođe i semver bumpa. Ovaj pregled samo dokumentira nesklad između Locked matrixa i službenih izvora razine **A**.

---

## 4. Kod — gdje ERP mapira (referenca za reviziju)

| Datoteka | Što radi |
|---|---|
| [`boxes.py`](../../erp/app/accounting/services/tax_forms/pdv/boxes.py) L209–261 | `610`–`615` označeni kao EU stjecanje; `207`/`307` nisu `implemented` |
| [`mapping.py`](../../erp/app/accounting/services/tax_forms/pdv/mapping.py) L106–140 | `map_eu_goods_base_610()` → konto `0373`; RC na `614`/`615` |
| [`render.py`](../../erp/app/accounting/services/tax_forms/pdv/render.py) L105–116 | `Podatak{code}` = 1:1 s registry kodom (nema preusmjeravanja 610→207) |

Za analizirani build: EU podaci iz knjigovodstva (**23.882,35 / 5.970,59**) ulaze u XML kroz **`Podatak610`–`615`**, dok službeni kanal glavne prijave **`Podatak207`/`307`** ostaje nula.

---

## 5. Auditabilna formulacija (obavezno)

| ❌ Neauditabilno | ✅ Auditabilno (korišteno u ovom dokumentu) |
|---|---|
| „ERP mapira EU stjecanje u 610" | Za build `v2/payload.json` (SHA256 `110037b4…`) ERP generira EU osnovicu stjecanja u element **`Podatak610` = 23.882,35**, dok službeni opis A-XSD-OPIS za `Podatak610` definira **ispravak pretporeza (VIII.1 zbroj)**, a EU stjecanje 25 % ide u **`Podatak207` / II.7** (A-PDV-2398). |
| „ERP je pogrešan" | Za isti build: `Podatak207` = 0,00/0,00; knjigovodstvo i PDV-S generator pokazuju **23.882,35** za EU stjecanje — **nesklad između ERP XML kanala (610) i službenog kanala (207)** je dokumentirana činjenica, ne zaključak o trajnoj ispravnosti sustava. |
| „Portal je u pravu, ERP treba 17.911,77" | Portal je nakon uploada prikazao `610 = 17.911,77` (razina **C**, observed behaviour). Ručni patch na tu vrijednost nema autoritativnu osnovu u izvorima razine A–B ([forenzika](pdv-610-patch-forensics-report.md), razina **D**). |
| „610 = 611 + 614 + 615" | Službeno: `610` = **1.1 + 1.2 + 1.3 + 1.4 + 1.5** (svih pet kategorija, A-XSD-OPIS). U buildu 05/2026: 611 = 614 = 615 = 5.970,59; zbroj triju = 17.911,77 — to je **aritmetika konkretnog builda**, ne citirana PU formula. |

**Konflikt je činjenica konkretnog builda**, ne trajna arhitekturna presuda dok Decision Gate nije zatvoren.

---

## 6. `erp-fix-610-rule` — rasprava vs implementacija

| Faza | Uvjet | Status nakon ovog pregleda |
|---|---|---|
| **Rasprava** o promjeni mapiranja | Dokaz razine **A** (Korak 2 ✅) + Source Matrix Review pokazuje konflikt | **◐ MOŽE SE OTVORITI** — konflikt 610 vs 207/307 dokumentiran gore |
| **Implementacija** `erp-fix-610-rule` | Cijeli Decision Gate (uključujući **Korak 3** ePorezna **C**) | **LOCKED** |
| **Implementacija** `erp-map-ii7-iii7` | Isto + potvrda mapiranja na `207`/`307` | **LOCKED** |
| Promjena `pdv-eu-source-matrix.yaml` | Računovođa + semver + regression | **LOCKED** (nije predmet ovog koraka) |

### Dokazni skup za Decision Gate (stanje 2026-07-06)

| Dokaz | Razina | Status |
|---|---|---|
| Službena uputa PU (citati) | **A** | ✅ Korak 2 |
| XML opis sheme (`Podatak610`–`615`, `207`, `307`) | **A** | ✅ A-XSD-OPIS u arhivu |
| Source Matrix Review (ovaj dokument) | sinteza | ✅ |
| ERP build (`payload.json`) | činjenica | ✅ Korak 0 |
| Forenzika patcha 17.911,77 | **D** | ✅ — ne argumentira fix |
| Ponašanje ePorezne (prije/nakon PDV-S) | **C** | ✅ T1 ([pdv-korak3-eporezna-test-2026.md](pdv-korak3-eporezna-test-2026.md)); T3 ⏳ 1b |
| PDV-S potpisani export | **C** | ⏳ Korak 1b |

**Zašto još ne implementirati:** nedostaje **Korak 3** i potpuni Decision Gate. Rasprava može definirati ciljno mapiranje (`207`/`307` + PDV-S, eventualno VIII.1 `614` za DI), ali kod ostaje zaključan.

---

## 7. Sinteza za Fine Star — svibanj 2026

| Kanal | Službeno očekivanje | ERP build 05/2026 |
|---|---|---|
| Glavna prijava EU stjecanja | **II.7** (`Podatak207`) = 23.882,35 / 5.970,59 | **0** |
| Pretporez EU stjecanja | **III.7** (`Podatak307`) | **0** (pretporez u 611/614/615) |
| PDV-S | DE229674882, I1 = 23.882,35 | ✅ generator |
| Ispravak pretporeza DI | **610** = zbroj kategorija VIII.1; vozila rent-a-car ⏳ računovođa | **610** = puna EU osnovica — **nesukladno A-XSD-OPIS** |

**610 LOCKED za implementaciju:** službeni izvor razine **A** ne opravdava trenutno ERP mapiranje EU osnovice u `Podatak610`. Otključavanje implementacije zahtijeva zatvoreni Decision Gate, ne portal observed behaviour niti patch razina **D**.

**Preporuka do Koraka 3:** upload **nepatchiranog** ERP XML-a (`610` = 23.882,35 iz `payload.json`); bilježiti portal observed behaviour bez tretiranja prikaza kao autoritativnog izvora.

---

## 8. Reference

| Dokument | Svrha |
|---|---|
| [`pdv-pu-uputa-2026-citations.md`](pdv-pu-uputa-2026-citations.md) | Službeni citati razine **A** (Korak 2) |
| [`pdv-may-2026-cross-check.md`](pdv-may-2026-cross-check.md) | Cross-check izvora po iznosima |
| [`pdv-610-patch-forensics-report.md`](pdv-610-patch-forensics-report.md) | Forenzika ručnog patcha (razina **D**) |
| [`pdv-eu-source-matrix.yaml`](pdv-eu-source-matrix.yaml) | Locked ERP poslovna matrica (nije mijenjana) |
| [`pdv-eu-implementation.md`](pdv-eu-implementation.md) | Implementacija boxova 610–615 u kodu |

*Zatvoreno: 2026-07-06 (Source Matrix Review).*

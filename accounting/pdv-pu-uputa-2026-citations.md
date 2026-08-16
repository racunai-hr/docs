# Korak 2 — Službena PU uputa: citati, pravila, pokrivenost (2026)

```
Status: CLOSED (Korak 2)
Decision Gate effect:
  ✓ Službena uputa arhivirana u repou
  ✓ Citati razine A izdvojeni (narativna uputa + ePorezna + XML opis sheme)
  ✓ Tablica pokrivenosti popunjena
  ✗ erp-fix-610-rule ostaje LOCKED — ERP mapira EU stjecanje u 610; službeni izvor definira 610 kao ispravak pretporeza (VIII.1), ne II.7
Next step:
  Decision Gate Review (C005/V001/V002 → [`pdv-korak3-eporezna-test-2026.md`](pdv-korak3-eporezna-test-2026.md) ✅ T1)
  Source Matrix Review: [`pdv-source-matrix-review-2026.md`](pdv-source-matrix-review-2026.md) ✅
```

Arhiv izvora: [`erp/docs/porezna/upute/2026/`](../porezna/upute/2026/README.md) (preuzeto 2026-07-06).

---

## Acceptance Criteria (Korak 2)

| Kriterij | Status |
|---|---|
| Službena uputa PU arhivirana | ✅ HTML + PDF + XSD zip u repou |
| Verzija upute identificirana | ✅ URL + datum na svakoj datoteci (vidi README arhiva) |
| Doslovni citati izdvojeni | ✅ dolje — PDV-S, II.7, VIII.1, 610–615 |
| Svaki citat povezan s lokacijom | ✅ stranica / poglavlje / članak |
| Evidence Matrix ažuriran | ✅ dolje — samo uz citat |

---

## 1. Arhivirani izvori

| ID | Datoteka u repou | Službeni URL | Napomena |
|---|---|---|---|
| **A-PDV-2398** | `Uputa-sastavljanje-prijave-PDV-2019.html` | [Mišljenje 2398](https://porezna-uprava.gov.hr/Misljenja/Detaljno/2398) | Uputa za sastavljanje PDV-a, 04.03.2019. |
| **A-PDV-1488** | `Uputa-popunjavanje-prijave-PDV-2019.html` | [Mišljenje 1488](https://porezna-uprava.gov.hr/Misljenja/Detaljno/1488) | Uputa + usporedba PDV ↔ PDV-S, 17.06.2013. |
| **A-FAQ-4355** | `FAQ-prijave-PDV-a.html` | [FAQ prijave PDV-a](https://porezna-uprava.gov.hr/hr/najcesce-postavljena-pitanja-faq-prijave-pdv-a/4355) | Usklađenost PDV / ZP / PDV-S |
| **A-PDV-S-PDF** | `Korisnicke-upute-Obrazac-PDV-S.pdf` | ePorezna G2B upute | Korisničke upute PDV-S (poruke 012, 013…) |
| **A-XSD-OPIS** | `Opis-elemenata-XML-sheme-PDV-v11.pdf` | iz [ePorezna_Schemas.zip](https://e-porezna.porezna-uprava.hr/Upute/G2B/ePorezna_Schemas.zip) | Semantika `Podatak610`–`615`, `207`, `307` |
| **A-XSD-PDVS** | `ePorezna_Schemas_extracted/.../ObrazacPDVStipovi-v1-0.xsd` | isti zip | Struktura PDV-S reda (`PDVID`, `I1`, `I2`) |
| **A-PRAV-2026** | `Izmjena-Pravilnika-PDV-2026.html` | [Izmjena Pravilnika](https://porezna-uprava.gov.hr/hr/izmjena-i-dopuna-pravilnika-o-porezu-na-dodanu-vrijednost-8156/8156) | PDV-S se **ne mijenja** od 1.1.2026. |
| **A-EVID** | `Poslovne-knjige-i-evidencije.html` | [Poslovne knjige](https://porezna-uprava.gov.hr/hr/poslovne-knjige-i-evidencije/4171) | Rok čuvanja isprava (11 godina) |

---

## 2. Doslovni citati

### 2.1 PDV-S — agregacija po dobavljaču (PDV ID)

**Izvor A-PDV-S-PDF**, poglavlje „Poruke na obrascu PDV-S“, poruka **012**:

> **012** — Isti PDV ID broj isporučitelja smije se pojaviti samo jednom u istom mjesecu.
>
> *Opis rješenja:* Isti PDV ID broj stranog isporučitelja na jednom obrascu PDV-S smije se pojaviti samo jednom. Potrebno je urediti podatke.

**Izvor A-XSD-PDVS**, tip `sIsporuka`:

> *Isporuka, sadrži podatke o vrijednostima isporuka od jednog isporučitelja*

**Interpretacija (ne citat):** više računa istog EU dobavljača u istom mjesecu mora se **zbrojiti u jedan red** (jedan `PDVID` po obrascu). Za Fine Star svibanj 2026: DE229674882, I1 = 8.000 + 15.882,35 = **23.882,35**.

---

### 2.2 PDV-S ↔ PDV — usklađenost iznosa

**Izvor A-PDV-1488**, §4 „Usporedba polja Prijave za stjecanje…“:

> Zbroj vrijednosti stečenih dobara iskazanih u polju **(13)** Prijave za stjecanje može biti **jednak ili manji** od zbroja rednih brojeva **II.5., II.6. i II.7.** PDV obrasca.
>
> Zbroj vrijednost primljenih usluga iskazanih u polju **(14)** Prijave za stjecanje mora bi jednak zbroju rednih brojeva **II.8., II.9. i II.10.** PDV obrasca.

**Izvor A-PDV-S-PDF**, poruka **013**:

> **013** — Ukupna vrijednost stečenih dobara (13) ne smije biti veća od sume iznosa osnovica II.5., II.6. i II.7 na PDV obrascu.

**Izvor A-FAQ-4355**:

> Zbirna prijava i Prijava za stjecanje dobara i primljene usluge iz drugih država članica Europske unije **uspoređivat će se sa Prijavom PDV-a** te putem VIES sustava.

**Pravilo za validator (V002):** `PDV-S.IsporukeUkupno.I1` ≤ `PDV.II.5 + II.6 + II.7` (osnovice).

---

### 2.3 II.7 — EU stjecanje dobara (25 %)

**Izvor A-PDV-2398**, poglavlje II, red **II 7**:

> Ovdje se upisuje vrijednost stečenih dobara unutar EU iz članka 4. stavka 1. točke 2. i članka 9. stavka 4. Zakona i pripadajući iznos PDV-a na stjecanje iz drugih država članica što ga porezni obveznik obračuna po stopi **25%**.

**Izvor A-XSD-OPIS**, element XML (mapiranje na obrazac):

> `<Podatak207>` — **7. STJECANJE DOBARA UNUTAR EU po stopi 25%** (Vrijednost / Porez)

**Izvor A-XSD-OPIS**, pretporez (III. dio):

> `<Podatak307>` — **7. PRETPOREZ OD STJECANJA DOBARA UNUTAR EU po stopi 25%**

**Za svibanj 2026 (Fine Star):** očekivano **II.7 = 23.882,35 / 5.970,59**; pretporez u **III.7** (XML `307`) isti iznos PDV-a.

**Napomena:** U narativnoj uputi 2398 red **II 15** = uvoz (`čl. 76.`); red **III 7** = pretporez od EU stjecanja 25 %. Plan je pogrešno označavao „III.7 = uvoz" — prema uputi i XML opisu **uvoz je II.15** (`Podatak215`).

---

### 2.4 VIII.1 / VI.1 — dugotrajna imovina (ispravak pretporeza)

**Izvor A-PDV-2398**, poglavlje VIII, red **VIII 1**:

> Ovdje se upisuju podaci o nabavi nekretnina, nabavi osobnih automobila i drugih sredstava za osobni prijevoz, prodaji osobnih automobila i drugih sredstava za osobni prijevoz te nabavi i prodaji **ostale dugotrajne imovine**. U podacima o nabavi osobnih automobila… upisuju se isključivo podaci o dijelu vrijednosti (poreznoj osnovici) s osnove koje je **odbijeno 50% pretporeza** iz članka 61. stavka 2. Zakona…

**Izvor A-PDV-2398**, uvod VIII:

> Pod ovom točkom upisuju se ostali podaci… na način da se unose samo **vrijednosni podaci (neto vrijednost)**.

**Za rent-a-car vozila (Fine Star):** čl. 61 iznimka za knjiženje prihvaćena u planu; **sadržaj VIII.1 redova za nabavu vozila na 0373** i dalje zahtijeva potvrdu računovođe (`accountant-viii1`) — uputa ne daje iznimku za flotu za iznajmljivanje u VIII.1.

---

### 2.5 XML polja 610–615 (službena semantika — **A1**)

Narativna uputa (2398/1488) **ne spominje** oznake 610–615. Semantika dolazi iz **A-XSD-OPIS** (službeni opis elemenata sheme v11):

| XML element | Službeni opis (citat) |
|---|---|
| **Podatak610** | **1. ZA ISPRAVAK PRETPOREZA (UKUPNO 1.1.+ 1.2.+ 1.3.+ 1.4.+ 1.5.)** |
| **Podatak611** | **1.1. Nabava nekretnina** |
| **Podatak612** | **1.2. Nabava osobnih automobila i drugih sredstava za osobni prijevoz** |
| **Podatak613** | **1.3. Prodaja osobnih automobila i drugih sredstava za osobni prijevoz** |
| **Podatak614** | **1.4. Nabava ostale dugotrajne imovine** |
| **Podatak615** | **1.5. Prodaja ostale dugotrajne imovine** |

**Ishod Koraka 2:** **A1** — službeni izvor **eksplicitno** definira 610–615 kao **VIII.1 ispravak pretporeza / DI**, ne kao EU stjecanje (II.7).

**Konflikt s ERP-om:** ERP trenutno mapira EU stjecanje dobara u box **610** ([`boxes.py`](../../erp/app/accounting/services/tax_forms/pdv/boxes.py)). Prema A-XSD-OPIS, EU stjecanje ide u **`Podatak207` / II.7**, a **610** je zbroj kategorija nabave/prodaje DI za ispravak pretporeza.

**Portal observed behaviour** (`610 = 17.911,77` ≈ `611+614+615`) **nije** u suprotnosti sa službenom formulom iz A-XSD-OPIS za 610 kao zbroj 611–615 — ali **jest** u suprotnosti s ERP mapiranjem EU osnovice u 610.

---

### 2.6 Dokazi — što čuvati

**Izvor A-FAQ-4355** (dokumentacija uz ZP, analogno za sve EU izvještaje):

> …porezni obveznik mora u svom **knjigovodstvu osigurati sve potrebne podatke** koji omogućuju ispravno i pravovremeno obračunavanje i plaćanje PDV-a te mora imati i **dokumentaciju koja dokazuje** da su ispunjeni uvjeti…

**Izvor A-PDV-2398** (opći izvor podataka):

> Podaci se u Obrazac PDV unose iz **knjigovodstva**… **urednih i vjerodostojnih knjigovodstvenih isprava**…

**Izvor A-EVID** (rok):

> Isprave na temelju kojih su podaci uneseni u dnevnik i glavnu knjigu — **11 godina**

**Operativna lista dokaza (EU stjecanje vozila, svibanj 2026):**

| Dokaz | Svrha | Rok |
|---|---|---|
| EU dobavljačevi računi (T-Cross, Golf) | Osnovica II.7 / PDV-S I1 | 11 god. (knjigovodstvene isprave) |
| VIES / PDV ID provjera DE229674882 | Identifikacija dobavljača | Arhiva provjere u ERP-u |
| Temeljnice (0373 nabava, 14022/24022 RC) | Knjiženje + pretporez | 11 god. |
| Generirani / predani **PDV-S XML** | Podnesak + usklađenost s PDV | 11 god. (porezna dokumentacija) |
| Generirani / predani **PDV XML** | Podnesak | 11 god. |
| Potpisani export s ePorezne (nakon 1b) | Dokaz predaje | 11 god. |

Popratna dokumentacija se **ne prilaže** uz PDV-S pri predaji (A-FAQ); mora postojati u knjigovodstvu.

---

## 3. Poslovna logika (iz citata)

### 3.1 Koje transakcije idu gdje

| Transakcija | Službeno polje | XML (v11) | PDV-S | ERP danas |
|---|---|---|---|---|
| EU stjecanje dobara 25 % (Hadžić vozila) | **II.7** + III.7 pretporez | `207` / `307` | Red po **PDV ID**, polje **I1** | Box **610–615** (scalar) — **odstupanje** |
| EU primljene usluge 25 % | II.10 + III.10 | `210` / `310` | PDV-S **I2** | Box 612–613 (ERP) |
| Uvoz dobara | II.15 | `215` | — (nema u PDV-S) | 0 za svibanj |
| Nabava DI / ispravak pretporeza | **VIII.1** | **610–615** | — | Nije popunjeno u ERP-u za svibanj |
| ZP (izlazne EU isporuke) | I.3 | `103` | — | N/A za svibanj |

**Zaključak:** Za EU nabavu vozila službeni kanal izvještavanja je **II.7 + PDV-S**, ne box 610. Box 610 služi **zbroju kategorija VIII.1** (ispravak pretporeza na DI).

### 3.2 Usklađenost PDV ↔ PDV-S

1. Jedan red PDV-S po **PDV ID-u** isporučitelja (A-PDV-S-PDF 012).
2. `Σ PDV-S.I1` (polje 13) ≤ `PDV II.5 + II.6 + II.7` osnovice (A-PDV-1488, A-PDV-S-PDF 013).
3. PDV-S se uspoređuje s PDV prijavom i VIES-om (A-FAQ-4355).
4. Rok predaje (od 01/2026): zadnji dan u mjesecu za prethodno razdoblje (A-PRAV-2026 kontekst; NN 151/25).

### 3.3 Box 610 — što **ne** ulazi

Prema A-XSD-OPIS, u **610** ne ide osnovica EU stjecanja — ide **zbroj 611+612+613+614+615** za ispravak pretporeza. EU stjecanje Hadžić vozila **ne mapira** se na 610 u smislu službenog izvora.

---

## 4. Tablica pokrivenosti

| Pitanje | Ishod Koraka 2 | Dokazna snaga |
|---|---|---|
| PDV-S agregacija po PDV ID | **Riješeno** — jedan red po isporučitelju | **A** (A-PDV-S-PDF 012, A-XSD-PDVS) |
| II.7 (EU 25 %) | **Riješeno** — osnovica + PDV; XML `207`/`307` | **A** (A-PDV-2398, A-XSD-OPIS) |
| VIII.1 / VI.1 (DI) | **Riješeno** — neto nabave po kategorijama; 50 % OA u VIII.1 | **A** (A-PDV-2398 VIII.1); rent-a-car iznimka u VIII.1 **⏳ računovođa** |
| Koje transakcije ulaze u box **610** | **Riješeno** — ispravak pretporeza DI (zbroj 611–615), **ne** EU II.7 | **A** (A-XSD-OPIS); konflikt s ERP mapiranjem dokumentiran |
| Usklađenost PDV (II.5–7) ↔ PDV-S (13) | **Riješeno** — PDV-S ≤ PDV | **A** + **C** (uputa + ePorezna 013) |
| Koji dokazi se čuvati | **Riješeno** — knjigovodstvo + isprave 11 god. | **A** (A-FAQ, A-EVID, A-PDV-2398) |
| Semantika XML **610–615** | **A1** — eksplicitno u opisu sheme | **A** (A-XSD-OPIS) |
| Formula `610 = 611+614+615` | **Djelomično** — 610 = **1.1+1.2+1.3+1.4+1.5** (svih pet) | **A** za zbroj; portalovo 17.911,77 **nije** citirano u uputi |
| III.7 = uvoz | **Odbijeno** — III.7 je pretporez EU 25 %; uvoz je II.15 | **A** (A-PDV-2398, A-XSD-OPIS) |

---

## 5. Ažuriranje Evidence Matrix (Faza 1)

| Tvrdnja | Stari | Novi | Dokaz |
|---|---|---|---|
| PDV-S agregacija po dobavljaču (PDV ID) | ◐ | **✅** | A-PDV-S-PDF 012 |
| II.7/III.7 treba dopuniti (EU u II.7) | ⏳ | **✅** (semantika); ERP gap ostaje | A-PDV-2398, A-XSD-OPIS; ERP ne puni `207` |
| Semantika polja 610–615 (službena) | ⏳ | **✅** A1 — ispravak DI, ne EU | A-XSD-OPIS |
| `610 = 611 + 614 + 615` | ❌ | **❌** (službeno: +612+613) | A-XSD-OPIS |
| `610` treba biti `17.911,77` | ❌ | **❌** | — |
| VIII.1 redovi za rent-a-car vozila 0373 | — | **⏳** | A-PDV-2398; čl. 61 iznimka u knjigovodstvu ✅, VIII.1 sadržaj ⏳ računovođa |

---

## 6. Implikacije za Fine Star — svibanj 2026

| Izvor | Očekivano (službeno) | ERP / stanje |
|---|---|---|
| Knjigovodstvo (0373 + RC) | 23.882,35 / 5.970,59 | ✅ |
| **II.7** (XML `207`) | 23.882,35 / 5.970,59 | **0** u ERP XML-u (gap) |
| **PDV-S** DE229674882 | 23.882,35 (1 red) | ✅ generirano |
| **610** (XML) | Ispravak DI (VIII.1) — za vozila vjerojatno **614** nabava ostale DI, ne ukupna EU osnovica | ERP šalje **23.882,35** u 610 — **nesukladno A-XSD-OPIS** |

**610 LOCKED:** Službeni izvor **ne** opravdava ERP mapiranje EU stjecanja u 610. Otključavanje `erp-fix-610-rule` zahtijeva Decision Gate + mapiranje na **II.7 (`207`)** / VIII.1 (`614`), ne korekciju prema portalu.

**Preporuka do Koraka 3:** upload **nepatchiranog** ERP XML-a radi observed behaviour; ne tretirati portalov `610` kao autoritativan bez testa nakon PDV-S.

---

## 7. Reference u repou

- **Source Matrix Review (ERP vs službeni element):** [`pdv-source-matrix-review-2026.md`](pdv-source-matrix-review-2026.md) ✅ 2026-07-06
- Cross-check svibanj: [`pdv-may-2026-cross-check.md`](pdv-may-2026-cross-check.md)
- Forenzika patcha: [`pdv-610-patch-forensics-report.md`](pdv-610-patch-forensics-report.md)
- ERP EU boxovi: [`pdv-eu-implementation.md`](pdv-eu-implementation.md)
- Arhiv PU: [`erp/docs/porezna/upute/2026/`](../porezna/upute/2026/README.md)

*Zatvoreno: 2026-07-06 (Korak 2).*

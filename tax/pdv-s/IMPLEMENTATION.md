# PDV-S — implementacija ADR-0034

```text
Status: Living document
Datum: 2026-11-28
ADR: ../../architecture/ADR-0034-pdv-s-workflow.md
R0: PARTIAL (Q-01, Q-02, Q-07 otvorena)
```

Svi primjeri su sintetički.

## 1. Rad u aplikaciji

Put: **Porezi i obrasci → PDV-S**.

1. **Pregled godine.** Svih 12 mjeseci, rok, status i iznosi. „Potrebna provjera” znači da evidencija ne dokazuje ni obvezu ni njezino nepostojanje, i nakon roka. „Rok istekao” prikazuje se kad postoji dokazani doprinos. Pregled ništa ne stvara u bazi.
2. **Porezne činjenice.** Za svaki dokument EU dobavljača računovođa potvrđuje vrstu (dobra, usluge, izvan obrasca), porezni postupak, stopu, datum isporuke i porezni datum. Sustav predlaže porezni datum (R0-03), ali ništa ne zaključuje iz države, opisa ili VIN-a. Dok činjenica nema, mjesec je blokiran stavkom `EU_TAX_FACTS_MISSING`.
3. **Kreiraj PDV-S.** Radi bez PDV prijave. Ako mjesečno razdoblje ne postoji, stvara se samo ono. Postoji najviše jedna aktivna verzija.
4. **Provjeri retke i izvore.** Svaki redak otvara dokumente i korekcije s pojedinačnim doprinosom.
5. **Validiraj** (uključuje VIES), **finaliziraj**, preuzmi XML i ručno ga predaj na ePoreznoj. Priprema i predaja su odvojene (vidi §1.1). Obrazac za pregled i ispis daje **PDF obrazac** (§1.2).
6. **Evidentiraj predaju:** portalni UUID i vrijeme zaprimanja. PDF potvrda i predani XML mogu doći kasnije; do tada je status „Predano – nedostaje dokumentacija”.
7. **Ispravak:** nakon promjene izvora prikazuju se razlike; ispravak se izrađuje samo izričitom radnjom i uvijek je cjelovit obrazac.

Povrat finaliziranog obrasca u pripremu traži razlog i potvrdu da XML nije predan. Stari XML ostaje u povijesti kao neaktivan.

### 1.1 Priprema tijekom tekućeg mjeseca

Obrazac se može validirati, finalizirati i preuzeti (XML, PDF, XLSX, ZIP) i dok mjesec traje. „Razdoblje još nije završilo” je upozorenje. Takva verzija prikazuje se kao **Finalizirana priprema**, a datoteke nose `_PRIPREMA`.

Evidentiranje predaje moguće je tek od prvog dana nakon razdoblja, s vremenom zaprimanja nakon njegova kraja. Prije predaje provjerite jesu li uključeni svi dokumenti: ako su se izvori promijenili, vratite obrazac u pripremu, osvježite, validirajte i ponovno finalizirajte.

### 1.2 PDF obrazac

`domains/tax/pdv_s/form_pdf.py` crta Obrazac PDV-S (ADR-0034 §10) iz zamrznute verzije, apsolutnim koordinatama na stranici 595 × 842 pt (ReportLab). Ne koristi vanjske resurse, a isti ulaz daje iste bajtove. Font je Liberation Sans 2.1.5 (SIL OFL 1.1, metrike Ariala) u `domains/tax/pdv_s/fonts/`.

| Polje | Izvor |
|---|---|
| (1), (2) | `print_header.tax_office`: šifrarnik `TaxOffice` prema šifri iz `CompanySettings`. Nepoznata šifra: ispisuje se šifra u (2), (1) prazno |
| (3)–(5), (7)–(14) | Snimka verzije: obveznik, razdoblje, retci, zbrojevi; valuta određuje naslov stupca (eura ili kuna) |
| (6) | Uvezeni XML (`Zastupnik/OIB`); inače prazno |
| (15), (17) | `prepared_by` iz snimke; kod uvoza kontakt iz izvornog XML-a |
| (16) | `print_header.signatory` (`accounting/services/pdv_s/print_header.py`); samo ispisano ime |

Dulji nazivi i adrese prelamaju se unutar polja i po potrebi smanjuju; tekst se ne odsijeca. Prva stranica prima 15 redaka uz zbrojeve, odnosno 22 bez njih.

Postavke: potpisnika određuju **odgovorne osobe** tvrtke (ovlaštena osoba → zakonski zastupnik → direktor). Nakon izmjene postavki obrazac u pripremi treba osvježiti; finalizirana verzija ostaje ista.

## 2. Odluke D01–D10

| Odluka | Provedba |
|---|---|
| D01 | Automatika iz poreznih činjenica; ručna korekcija s dokumentom, razlogom, autorom; isključenje temeljnice; zaštita od dvostrukog uključivanja; zatvaranje vezom na dokument |
| D02 | UUID i vrijeme obvezni; PDF i XML naknadno, neovisno; neusklađen XML se čuva, ali ne zatvara status |
| D03 | Drill-down iz zamrznute verzije: dokument, temeljnica, klasifikacija, porezni datum, valuta, tečaj, EUR |
| D04 | Godišnji pregled bez upisa; ručno kreiranje; potvrda „nema obveze” koja istječe novim izvorom; rokovi s neradnim danima |
| D05 | Razlike po državi, PDV ID-u, vrsti i izvoru; ručni ispravak |
| D06 | Blokirajuće greške, upozorenja, informacije; VIES s odlukom za negativan rezultat; validacija vezana uz točan sadržaj |
| D07 | XML, PDF obrazac (§1.2), XLSX, PDF validacije, originalni dokazi, ZIP s manifestom i hashovima |
| D08 | Finalizacija zamrzava snimku, hash i XML; povrat u pripremu s auditom; `VATPeriod` se ne zaključava |
| D09 | Dva statusa (portalni ≤ zadnje zaprimljeni; interni = zadnji predani); pokreće se nakon predaje; obrazloženja |
| D10 | Pojedinačni uvoz uz pregled; zaštićeni parser; konflikt UUID-a i starije predaje blokira; razdoblja prije 2023. u kunama |

## 3. REVIEW REQUIRED (finalizacija zahvaćenog obrasca blokirana)

| Pitanje | Okidač u sustavu |
|---|---|
| Q-01 | odobrenje, povrat, storno; negativan doprinos; storno temeljnica na EU kontima |
| Q-02 | negativan redak |
| Q-07 | tečaj koji nije objavljen na porezni datum; ostatak zaokruživanja koji mijenja redak |
| Q-03 do Q-06 | postupci „bez PDV ID-a”, „oslobođeno stjecanje”, „vlastita dobra”, „kontinuirana usluga” |

Ostali mjeseci i obrasci nisu time pogođeni.

## 4. Arhitektura

```text
Ulazni dokument ─► ExpenseTaxFact (revizije) ─┐
PDVSCorrection (dokaz, razlog) ───────────────┤
Temeljnice na EU kontima (samo provjera) ─────┤
                                              ▼
                    TaxClassificationEngine (jedini klasifikator)
                      ├─► PDV projekcija (II.5–II.10 / III.5–III.10)
                      └─► PDV-S kandidat (domains/tax/pdv_s, bez ORM-a)
                                              ▼
 PDVSReturn (anchor) ─► PDVSReturnVersion (snimka, XML, validacija, hash)
                                              ▼
                     SubmissionService.create_event (nepromijenjen)
                                              ▼
                     PDVSEvidenceAttachment · PDVSReconciliation
```

| Sloj | Putanja (`api/app`) |
|---|---|
| Čista pravila (prenosiva) | `domains/tax/facts/`, `domains/tax/pdv_s/` (uključujući `form_pdf.py`) |
| ORM servisi | `accounting/services/pdv_s/`, `accounting/services/tax_facts/` |
| Modeli | `accounting/models_pdv_s.py` |
| API | `domains/tax/api/pdv_s_views.py`, `pdv_s_schema.py` |
| Sučelje (`app`) | `src/lib/pdvs.ts`, `src/components/tax/pdvs/` |

- Dokazi, korekcije i XML pohranjuju se u bazi, ne pod javno posluživanim `media`.
- `content_hash` koristi isti algoritam kao ranije predaje (`pdvs-v1`); `integrity_hash` pokriva cijelu snimku.
- Zapisi činjenica, dokaza, VIES-a, usklađenja i povijesti ne brišu se.
- `SubmissionService` i `SubmissionEvent` nisu mijenjani; usklađenje pokreće Django signal nakon commita.

## 5. API

Sve rute pod `/api/tax/`, JWT, tenant iz hosta. Čitanje i izvoz: sve uloge. Izmjene: vlasnik i računovođa.

`pdv-s/years/{year}/`, `pdv-s/months/{period}/` (`draft/`, `diff/`, `corrections/`, `reconciliation/`, `no-obligation/`), `pdv-s/versions/{id}/` (`refresh/`, `validate/`, `finalize/`, `revert/`, `submit/`, `export/{kind}/`), `pdv-s/submissions/{uuid}/evidence/`, `pdv-s/evidence/{id}/`, `pdv-s/corrections/{id}/…`, `pdv-s/vies/{id}/decision/`, `pdv-s/reconciliations/{id}/notes/`, `pdv-s/import/preview|confirm/`, `expenses/{id}/tax-facts/`, `tax-date-suggestion/`.

Raniji putovi `pdv-s/periods/{period}/…` zadržani su: XML se izdaje samo iz zamrznute verzije, a predaja samo nad finaliziranom. Admin evidencija predaje PDV-S isključena je.

## 6. Promjene porezne jezgre (mapiranje v16)

| Bilo | Sada |
|---|---|
| 204 prodaja na daljinu | 204 tuzemni prijenos porezne obveze (ranije 209) |
| 209 tuzemni prijenos | 209 primljene EU usluge 13 % |
| EU usluge 5/13 % u 210 | 208 / 209 / 210 prema stopi |
| EU dobra 5/13 % u 207 | 205 / 206 / 207 prema stopi |
| pretporez EU usluga u 306 | 308 / 309 / 310 |
| pretporez tuzemnog prijenosa 309 | 304 |
| OSS → 215, elektroničko sučelje → 214, IOSS → 308 | REVIEW REQUIRED (nema polja) |
| EU trošak bez VIN-a → usluga | REVIEW REQUIRED dok nema poreznih činjenica |
| svako knjiženje na 0373 → 207 | službeni dokument i temeljnica s tuzemnim pretporezom isključeni; 207 samo uz prijenos porezne obveze u istoj temeljnici ili odgovarajućeg iznosa u razdoblju, inače pregled |
| 660 uvijek „da” | `CompanySettings.vat_cash_accounting`; prazno zadržava „da” |

Predane i uvezene PDV prijave nisu mijenjane. Za razdoblja čiji retci nastaju iz EU nabava, ponovno generiranje PDV knjiga (otvoreno razdoblje ili ispravak) traži potvrđene porezne činjenice.

## 7. Ograničenja

- Polja 301, 302, 211–215, 311–315, 630–650 ne popunjavaju se automatski.
- Stari generator knjiga (prekidač `vat_projection_write` isključen) ne koristi porezne činjenice; PDV-S ih koristi neovisno o prekidaču. Razlika se vidi u usklađenju.
- Format PDV ID-a provjerava se prema ograničenjima sheme, bez kontrolnih znamenki po državama (Q-11).
- Rokovi za razdoblja s krajem prije 2020. označeni su približnima.
- VIES odgovor vrijedi za trenutak upita; `GB` nije pokriven.
- Za predaje evidentirane prije ove verzije sadržaj nije dostupan; usporedba i snimka ne rekonstruiraju se.
- Usklađenje koristi zbrojeve PDV prijave; razrada po dobavljaču postoji samo za PDV-S.
- Predaja na ePoreznu ostaje ručna; G2B nije dio opsega.
- Šifrarnik poreznih ureda ima jedno mjesto po šifri, pa ono imenuje i područni ured i ispostavu. Ispostava izvan sjedišta područnog ureda traži zaseban podatak u šifrarniku.
- Nema postavke za poreznog zastupnika ni broj faksa.
- Verzije zamrznute prije PDVS-HOTFIX-03 uzimaju nazive ureda i potpisnika iz postavki u trenutku ispisa; njihov PDF može se promijeniti s postavkama.

## 8. Vraćanje na prethodnu verziju

Migracije `accounting.0046` i `settings.0014` samo dodaju tablice i jedan neobavezni stupac. Stari kod radi uz njih, pa povratak koda **ne traži** vraćanje baze:

1. vratiti prethodni commit `api` i prethodnu sliku `app`;
2. ponovno pokrenuti servise.

Nakon povratka stari kod ne vidi nove verzije; evidentirane predaje ostaju kao `SubmissionEvent`. PDV knjige generirane novim mapiranjem u otvorenim razdobljima treba ponovno generirati. Unmigracija briše nove tablice zajedno s dokazima i dopuštena je samo dok su prazne.

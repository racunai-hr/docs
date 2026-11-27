# PDV-S — R0 arhitekturna revizija ADR-0034

```text
Status: R0 PARTIAL — preporuka: ADR-0034 ostaje Proposed
Datum: 2026-11-27
Predmet: architecture/ADR-0034-pdv-s-workflow.md (docs develop @ 77f7228)
```

Povezano: [regulatorna analiza](R0-REGULATORY-ANALYSIS.md) · [matrica](R0-SCENARIO-MATRIX.md) · [audit](R0-EXISTING-IMPLEMENTATION-AUDIT.md) · [završni izvještaj](R0-FINAL-REPORT.md)

## 1. Zaključak

Svih deset poslovnih odluka međusobno je dosljedno i nijedna ne proturječi propisima utvrđenima u R0. **Nije potrebno ponovno otvarati nijednu odluku.** Nisu provedive samo unutar PDV-S modula: tri preduvjeta pripadaju poreznoj jezgri (ADR-0019 / ADR-0007), a dvije odluke (D02, D10) traže zaštitni sloj ili vlastite modele uz zamrznuti `SubmissionService` (ADR-0009).

## 2. Deset poslovnih odluka

| Odluka | Regulatorna provjera | Arhitekturna provedivost | Uvjet |
|---|---|---|---|
| D01 B — automatika + dokumentirane korekcije | dopušteno; obrazac mora sadržavati točne podatke | **uvjetno** | pravilo odnosa korekcije i PDV knjige (K-3); stabilan identitet izvora za sprječavanje dvostrukog uključivanja |
| D02 B — UUID/datum odmah, PDF i XML naknadno | portal izdaje XML/PDF potvrde i dokument | **uvjetno** | prilozi u modelu u vlasništvu PDV-S, uz FK na event; ne mijenjati `SubmissionEvent` (K-4) |
| D03 C — puni pregled izvora | — | **uvjetno** | porezne činjenice po dokumentu (K-1) moraju postojati i biti snimljene |
| D04 B — detekcija, ručno kreiranje | rokovi i „nema obveze” po R0-01 | da, nakon K-1 | read-only kandidat za 12 mjeseci; nepotpunost → „Potrebna provjera” |
| D05 B — diff, ručni ispravak | ispravak je cjeloviti obrazac (P 182/7) | da | snimka predane verzije; razdoblje odobrenja Q-01 |
| D06 B — validacija i VIES | pravila 005/009/011/012 nisu u XSD-u; VIES nema povijesni upit | da | VIES klijent je nov connector (governance: RFC + Issue) |
| D07 C — arhiva | — | da | pohrana artefakata s kontrolom pristupa po tenantu; provjera prije implementacije |
| D08 B — zaključavanje verzije | — | da | event dobiva hash zamrznute verzije preko postojećeg parametra `payload_hash` |
| D09 C — rekoncilijacija | portalna: II.5–10 zadnje **zaprimljenog** PDV-a, manje ili jednako | **uvjetno** | ispraviti mapiranje PDV polja (K-2); okidanje bez izmjene `SubmissionService` |
| D10 B — pojedinačni uvoz | razdoblja do 2022. u kunama; `GB` u shemi | **uvjetno** | pravilo redoslijeda (K-5); valuta po razdoblju (K-6) |

### Preciznosti koje ne mijenjaju odluke

- **D09:** portalna kontrola uspoređuje sa zadnje zaprimljenim PDV obrascem bez obzira na status; interna sa zadnje predanim nepromjenjivim verzijama. To su dva statusa nad moguće različitim verzijama (`api` već razlikuje stanje „u obradi”).
- **D09, periodičnost:** obveznik s EU transakcijama mora biti mjesečni (R0-01 1.7); drukčiji slučaj je iznimka koju treba prikazati kao neusporedivu.
- **§4.1 ADR-a:** „VAT-ID-only obveznik” obuhvaća usluge uvijek, a dobra tek iznad praga stjecanja ili uz odustajanje (R0-01 1.3).
- **§1 ADR-a:** „sva zakonski relevantna nabava” isključuje, među ostalim, oslobođene usluge, posebna pravila mjesta, maržu i uvoz (R0-02).

## 3. Konzistentnost s postojećim ADR-ovima i dokumentima

| Dokument | Odnos | Ocjena |
|---|---|---|
| ADR-0007 (PDV) | PDV-S ne mijenja PDV payload; ali porezni datum i mapiranje polja zahvaćaju jezgru | **konflikt u implementaciji** (K-1, K-2); promjena traži PDV gate (snapshot testovi, `PDV_MAPPING_VERSION`) |
| ADR-0008 | jedan generički append-only audit; „verzioniranje obrasca” odvojeno od predaje | usklađeno; ADR-0034 dodaje `PDVSReturnVersion` ispod anchora |
| ADR-0009 (freeze) | `SubmissionService` jedini ulaz; core se ne mijenja bez ADR-a | usklađeno uz zaštitni sloj; K-4, K-5 |
| ADR-0010 | PDV-S pripada domeni Tax | usklađeno; novi modeli u Tax domeni, bez novog Django appa |
| ADR-0011 / governance | „Promjena PDV/banking/submission jezgre → ADR + Architecture Review” | K-1 i K-2 traže zaseban ADR amendment i review |
| ADR-0019 (**Proposed**) | jedinstveni klasifikacijski engine; kod postoji uz legacy putanju | **konflikt s invarijantom „jedan sustav”** dok postoje dvije putanje (K-7) |
| ADR-0023 | `country_code` je SSOT; porezni sloj još koristi nazive država | otvoren follow-up; potreban prije PDV-S klasifikacije |
| ADR-0027 | ispravak PDV-a može ponovno graditi knjigu predanog razdoblja | PDV-S snimka mora biti neovisna o knjizi; D09 ponavlja kontrolu |
| ADR-0033 devizni računi | porezni preračun odvojen i potvrđen | usklađeno s Z čl. 36.; ciljni datum mora biti porezni datum |
| `FORM_REGISTRY` | `payload.json` kanon | ADR-0034 ga uvodi; do tada registry opisuje stanje koje ne postoji |
| `FORM_IMPLEMENTATION_CONVENTION` | PDV-S „migrate opportunistically”; fixtures `fixtures/tax/<form>/` | preporuka: PDVS-02/03 uskladiti s konvencijom (`payload.py`, `build.py`, `canonical.py`, `verify.py`) |
| `EPOREZNA_SUBMISSION_MATRIX` | PDV-S „Quarterly” | netočno; ispraviti u zasebnoj dokumentacijskoj izmjeni |

### Nalaz o numeraciji ADR-ova

Broj **ADR-0033** koriste dva dokumenta: `ADR-0033-incoming-invoice-foreign-currency.md` na grani `docs/main` ([poveznica](https://github.com/racunai-hr/docs/blob/main/architecture/ADR-0033-incoming-invoice-foreign-currency.md), na njega se poziva kod) i `ADR-0033-mcp-gateway.md` na `docs/develop`. Grane `main` i `develop` repozitorija `docs` su se razišle. Treba odluka o renumeraciji prije sljedećeg spajanja; ADR-0034 time nije izravno pogođen.

## 4. Arhitekturni konflikti

| ID | Konflikt | Posljedica | Preporuka |
|---|---|---|---|
| **K-1** | Porezni datum, vrsta isporuke, posebni režimi i snimka dobavljača ne postoje u izvorima ni klasifikaciji; razdoblje bira datum troška | D01, D03, D04 nisu izvedivi; PDV i PDV-S moraju dijeliti isto razdoblje | Amendment ADR-0019: porezne činjenice kao ulaz jedinstvenog klasifikatora. **Ne** rješavati posebnom logikom unutar PDV-S (bio bi drugi porezni engine) |
| **K-2** | Registar PDV polja ne odgovara službenom obrascu (audit A-02) | D09 uspoređuje pogrešna polja; PDV XML može biti netočan za pogođene transakcije | Zaseban hitni zadatak pod ADR-0007; D09 tek nakon njega |
| **K-3** | PDV-S korekcija koja ne ulazi u PDV knjigu stvara drugu poreznu istinu | napetost D01 ↔ „jedinstveni sustav” | Korekcija = porezna činjenica Tax domene s vezom na dokument; PDV-S je uključuje odmah, PDV je vidi kao otvorenu razliku (D09). Kada izvor bude proknjižen, korekcija se zatvara vezom, ne briše |
| **K-4** | Jedan prilog po eventu | D02 | `PDVSEvidenceAttachment` (uloga PDF/XML, hash, autor, vrijeme) s FK na `SubmissionEvent`; postojeći `confirmation_attachment` ostaje kompatibilan. Generički `SubmissionAttachment` tražio bi ADR (backlog ADR-0009) |
| **K-5** | `current_submission` = najveći `submission_no` | uvoz starije predaje postaje „aktualan” | Bez izmjene servisa: uvoz starije predaje kad postoji novija **blokirati uz pregled** (D10 to dopušta). Ako je takav uvoz poslovno nužan, potreban je amendment ADR-0009 |
| **K-6** | Sustav pretpostavlja EUR; razdoblja prije 2023. su u kunama | D10 i ispravci povijesnih razdoblja | Valuta kao polje verzije, izvedena iz razdoblja; bez konverzije pri uvozu |
| **K-7** | Legacy i engine projekcija daju različit PDV-S | invarijanta kanonskog izvora | PDV-S graditi samo iz jedinstvenog klasifikatora; stanje prekidača po obvezniku provjeriti prije PDVS-03 |
| **K-8** | PDV predaja zaključava `VATPeriod`; kasni račun ne može u knjigu, a PDV-S ispravak ga mora sadržavati | ADR §4.1 „read-only kandidat” | `prepare_vat_projection()` već radi bez upisa; PDV-S kandidat graditi tim putem, uz greške **po dokumentu** (danas jedan problem odbija cijeli kandidat) |
| **K-9** | `VATPeriod` rasponi i `CASCADE` na `PDVSReturn` | pogrešno razdoblje, gubitak anchora | PDV-S dopustiti samo na mjesečnom retku; za nove verzije `on_delete=PROTECT` |
| **K-10** | Hash ovisi o redoslijedu i o živom agregatu | usporedba dokaza, D08 | Verzija algoritma; legacy-kompatibilan hash računati iz zamrznute, kanonski sortirane snimke; puni integrity hash zasebno |

## 5. Provjera invarijanti

| Invarijanta | ADR-0034 tekst | Trenutni kod |
|---|---|---|
| Neovisnost PDV-S i PDV | očuvana | očuvana |
| Jedan porezni klasifikacijski sustav | tražena | **nije** (K-7) |
| Kanonski računovodstveni izvori | tražena | djelomično (regex opisa, ručna mapa) |
| Nepromjenjive predane verzije | tražena | **ne postoje** |
| Povijest `SubmissionEvent` | očuvana | očuvana |
| Tenant izolacija | tražena | API da; prilozi i pomoćna sučelja provjeriti |
| Postojeća zaključavanja | očuvana | PDV-S ih ne dira |
| Frozen `SubmissionService` | očuvan uz K-4, K-5 | — |
| Bez paralelnog poreznog enginea | tražena | ugrožena ako se K-1 rješava unutar PDV-S |

## 6. Predložena ciljna struktura (nije odobrena)

```text
Izvorni dokumenti ──► porezne činjenice (datum, vrsta, režim, snimka dobavljača)   [ADR-0019 amendment]
                            │
                   TaxClassificationEngine (jedan)
                     ├──► PDV projekcija / Obrazac PDV            [ADR-0007]
                     └──► PDV-S kandidat (read-only, greške po dokumentu)
                              + dokumentirane korekcije
                                   │
                    PDVSReturn (anchor) ── PDVSReturnVersion (snimka, XML, validacija, hash)
                                   │              └── PDVSEvidenceAttachment
                    PDV-S guard ──► SubmissionService.create_event(payload_hash=…)  [nepromijenjen]
```

## 7. Preporučena dopuna redoslijeda gateova

Postojeći gateovi PDVS-01 do PDVS-08 ostaju. Preporučuje se ispred njih:

| Gate | Opseg | Kriterij izlaska |
|---|---|---|
| R0.1 | Odgovori na Q-01, Q-02, Q-07 (i po mogućnosti ostala) | scenariji O → V/D |
| TAX-0 | Ispravak registra PDV polja (K-2) | snapshot testovi, provjera utjecaja na predane prijave |
| TAX-1 | Amendment ADR-0019: porezne činjenice i odobrenja (K-1, K-7) | Architecture Review; ledger diff |
| zatim | PDVS-01 … PDVS-08 | PDVS-01/02 mogu paralelno s TAX-1; PDVS-03 ovisi o njemu |

Ažuriranje plana u §5 ADR-0034 nije napravljeno; čeka potvrdu.

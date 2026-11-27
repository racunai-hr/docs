# PDV-S — R0 audit postojeće implementacije

```text
Status: R0 PARTIAL
Datum: 2026-11-27
Pregledano: api origin/develop @ 180dcb9 (stablo identično main @ c4902ae)
            app origin/develop @ b91a1c8 (stablo identično main @ e6e9b91)
            docs origin/develop @ 77f7228
Metoda: statički read-only pregled koda i testova; validacija sintetičkih primjera na službenom XSD-u.
Nije izvedeno: Django testovi, upiti nad bazom, provjera produkcijskog sučelja i konfiguracije.
```

Putanje su relativne na `api/app/` odnosno `app/src/`. Nalazi o ponašanju izvedeni su čitanjem koda i treba ih potvrditi testom prije popravka. Koja se projekcijska putanja koristi za pojedinog obveznika (postavka `vat_projection_write`, zadano isključeno — `accounting/services/tax_projection/switch.py`) nije provjereno.

Povezano: [regulatorna analiza](R0-REGULATORY-ANALYSIS.md) · [matrica](R0-SCENARIO-MATRIX.md) · [arhitekturna revizija](R0-ARCHITECTURE-REVIEW.md) · [završni izvještaj](R0-FINAL-REPORT.md)

## 1. Tok podataka danas

```text
Invoice / Expense / JournalEntry
  → generate_vat_ledger()  (legacy, zadano)   ili   TaxClassificationEngine → projekcija (switch on)
  → VATLedgerEntry (box 207, 210)
  → aggregate_pdv_s_rows()        računa se uživo pri svakom čitanju
  → render_pdv_s_xml()            nepotpisan XML, ne sprema se
  → mark_pdv_s_submitted() → SubmissionService.create_event()   hash = ponovni živi izračun
```

## 2. Nalazi

Ozbiljnost: **K** kritično (netočan sadržaj porezne prijave), **V** visoko, **S** srednje, **N** nisko.

### A-01 (K) EU nabave evidentirane kao trošak mogu izostati ili biti krivo razvrstane

Legacy putanja (`accounting/services/vat.py`):

- grana za EU trošak bez PDV-a (`vat.py:434`) stvara samo box **614** i to kad nema temeljnice; nikad ne stvara 207 ni 210;
- retke temeljnica kojima je izvor trošak preskače i za konta EU dobara i EU usluga (`_RC_SKIP_ACCOUNTS`);
- zaključak: u toj putanji nabava evidentirana kao `Expense` ne dolazi u PDV-S; 207/210 nastaju samo iz temeljnica kojima izvor nije trošak.

Engine putanja (`domains/tax/classification/engine.py:355` i dalje):

- `adapt_expense` uvijek postavlja `supply_kind='unknown'` (`accounting/services/tax_shadow/adapters.py:193`);
- roba se prepoznaje samo po VIN-u u opisu (`is_eu_goods_acquisition`, `mapping.py:402`);
- EU roba bez VIN-a bez temeljnice → klasificira se kao **usluga** (210);
- EU roba bez VIN-a s temeljnicom → `eu_expense_posted_via_journal` (`engine.py:428`), a retci konta 0373/1402x/2402x preskaču se (`tax_projection/snapshot.py:60`) → nabava **izostaje**, ili, ako je automatsko knjiženje stvorilo 24032, ulazi kao usluga;
- isti kriterij koristi automatsko knjiženje: EU trošak profila `opex` bez VIN-a knjiži se kao RC usluga (`accounting/services/posting.py:865`).

Posljedica: zahtjev „sve EU nabave, ne samo vozila” nije ispunjen.

### A-02 (K, modul PDV) Registar PDV polja ne odgovara službenom Obrascu PDV

`accounting/services/tax_forms/pdv/boxes.py` i `mapping.py` šifre polja izravno zapisuju kao `Podatak<šifra>` (`pdv/render.py`). Usporedba sa službenim opisom elemenata i P čl. 175.–176.:

| Polje | Službeno značenje | Lokalna oznaka / izvor |
|---|---|---|
| 204 | tuzemni prijenos porezne obveze | prodaja na daljinu |
| 205 / 206 | stjecanje dobara 5 % / 13 % | NPS / montaža |
| 208 / 209 | primljene EU usluge 5 % / 13 % | trostrani / **tuzemni prijenos** (konta 2401x) |
| 214 / 215 | naknadno oslobođenje izvoza / uvoz | e-sučelje / OSS |
| 304 | pretporez tuzemnog prijenosa | uvoz |
| 306 | pretporez stjecanja dobara 13 % | **pretporez EU usluga** (konto 14032) |
| 308 / 309 | pretporez EU usluga 5 % / 13 % | IOSS / pretporez tuzemnog prijenosa |

Polja 207, 210, 303 i 307 odgovaraju. EU usluge po 5 % i 13 % mapiraju se u 210 / 306, a dobra po nižim stopama u 207.

Posljedica za PDV-S: osnova za kontrole 013/014 (II.5–10) iz lokalnih podataka nije pouzdana; II.9 može sadržavati tuzemni prijenos. Posljedica za PDV XML prelazi opseg ADR-0034 i traži zaseban zadatak pod ADR-0007. Je li ijedna predana prijava pogođena **nije provjereno**.

### A-03 (K) Domaća knjiženja na konto 0373 tretiraju se kao EU stjecanje

Svaki dugovni redak na 0373 iz temeljnice kojoj izvor nije trošak daje box 207 (`vat.py:544` i dalje; `domains/tax/classification/accounts.py:63`), osim ako opis temeljnice sadrži „PPMV”. Pravila za službene dokumente knjiže na 0373 profile `ppmv_vehicle_acquisition`, `fzoeu_vehicle_acquisition_fee` i `fzoeu_tire_acquisition_fee` (`posting.py:210`–`247`), a opis temeljnice je `[official_document_posted] <broj> — <iznos>` (`posting.py:1173`), pa riječ „PPMV” nije zajamčena. `OfficialDocument` nema polje `supplier`, stoga razrješenje PDV ID-a ne uspijeva i **cijeli PDV-S za mjesec vraća grešku**; ujedno iznos ulazi u PDV II.7. Isto vrijedi za svaku ručnu temeljnicu dugotrajne imovine u pripremi koja nije EU stjecanje.

### A-04 (K) Odobrenja i storna

- `aggregate_pdv_s_rows()` preskače `base_amount <= 0` (`pdv_s/aggregate.py:144`);
- engine upisuje storno s negativnim predznakom (`tax_projection/apply.py:320`) → agregator ga ignorira, a PDV zbraja s predznakom;
- `Expense.amount` ima `MinValueValidator(0.01)` (`expenses/models.py:342`) i nema modela odobrenja → odobrenje dobavljača ne može se evidentirati kao ulazni dokument.

Posljedica: PDV-S može biti precijenjen i veći od PDV II.5–10.

### A-05 (K) Nema poreznog datuma

Razdoblje se bira po `expense_date` (`tax_shadow/selection.py:27`) odnosno datumu temeljnice; engine to bilježi kao `period_date_reason='document_date'` (`engine.py:119`, `:146`). `Expense` nema datum isporuke, datum primitka računa ni porezni datum. Parser eRačuna čita datum isporuke (`ubl/parser/invoice.py:361`), ali se ne sprema na trošak. Pravila R0-03 ne mogu se provesti.

### A-06 (V) Nema isključenja i posebnih slučajeva

Nema podatka o marži, oslobođenju EU usluge, posebnom mjestu obavljanja, državi otpreme ni pragu stjecanja. Posljedice: račun marže s VIN-om → 207; oslobođena EU usluga bez PDV-a → 210 (provjera kategorije `E` dolazi tek iza EU grane, `engine.py:471`); usluga iz `XI` → 210 jer je `XI` među EU prefiksima (`mapping.py:36`); `GB` se ne može proizvesti ni za razdoblja do 2020.

### A-07 (V) Identifikacija dobavljača

- redoslijed: `partner_oib` s retka → izvor temeljnice → regex „Račun:” iz opisa → `MANUAL_SUPPLIER_MAP` (`pdv_s/aggregate.py:58`–`113`);
- engine retci iz temeljnica nemaju `partner_oib` (`tax_projection/apply.py:358`);
- koristi se trenutni partner, nema snimke po transakciji;
- `is_eu_supplier` oslanja se na nazive država i prefikse, ne na `country_code` (`mapping.py:373`; otvoreni follow-up ADR-0023);
- `split_eu_vat_id` prihvaća `HR` i ne provjerava format ni duljinu;
- jedna neriješena stavka baca `ValueError` i ruši pregled, XML i evidentiranje predaje za cijelo razdoblje (nema grešaka po dokumentu).

### A-08 (V) Nema verzije, snimke ni nepromjenjivog sadržaja

`PDVSReturn` (`accounting/models.py:1105`) ima samo `tenant`, `vat_period`, `version`, `created_at`. `get_payload_hash()` poziva živi agregat (`submission/protocol.py:52`). `GET .../pdv-s/periods/{period}/` i `/xml/` računaju uživo (`domains/tax/read/service.py:135`, `domains/tax/write/service.py:317`). XML pri svakom renderu dobiva novi `Identifikator` i vrijeme (`pdv_s/render.py:61`). `payload.json` ne postoji.

### A-09 (V) Evidentiranje predaje bez provjera

`submit_pdv_s_period()` (`write/service.py:208`): `version_confirmed=True` uvijek; nema XSD provjere pri predaji; prazan prvi obrazac se prihvaća; ne provjerava se PDV ID ni status obveznika; sadržaj koji se hashira može se razlikovati od XML-a koji je korisnik ranije preuzeo i predao.

### A-10 (V) Ograničenja `SubmissionService` (zamrznut, ADR-0009)

| Činjenica | Referenca | Utječe na |
|---|---|---|
| Jedan `confirmation_attachment`, postavlja se jednom | `models.py:1240`; `service.py:535` | D02 (zasebni PDF i XML) |
| `current_submission` = najveći `submission_no` | `service.py:364` | D10 (kasniji uvoz starije predaje postao bi aktualan) |
| `submission_type` izveden iz broja | `models.py` `SubmissionEvent.submission_type` | D10 (uvoz starije predaje prikazan kao ispravak) |
| Neusklađen hash XML priloga je upozorenje | `service.py` `validate()` | D02 |
| Za PDV-S provjerava samo početak razdoblja i OIB | `service.py:177` | dokaz predaje |
| `select_for_update().filter(...)` ne izvršava upit | `service.py:459` | zaključavanje se mora osigurati na ulazu (PDV-S to danas radi zaključavanjem `VATPeriod`, `pdv_s/submit.py`) |
| PDV-S ne zaključava `VATPeriod` | `service.py:530` | D08 — **ispunjeno** |

### A-11 (S) Naredba za arhiviranje nije uporabiva kao uvoz

`archive_pdv_s_submission`: traži potpis (`:72`), a portalni XML često dolazi bez potpisa; zahtijeva jednakost sa živim agregatom; kao vrijeme predaje upisuje `timezone.now()` (`:145`); ne sprema XML uz event.

### A-12 (S) Razdoblje i referencijalni integritet

`VATPeriod` (`models.py:567`) dopušta višemjesečne raspone (`period_to`), a `pdv_s/aggregate.py` uzima granice samo prvog mjeseca dok filtrira sve retke tog razdoblja. `PDVSReturn.vat_period` je `on_delete=CASCADE`, a `SubmissionEvent` ga referencira generički (bez FK) — brisanje razdoblja ostavilo bi evente bez dokumenta.

### A-13 (S) Zaglavlje obrasca

Bez odgovorne osobe upisuje se `NA` / `NA` (`aggregate.py:133`); prazna ispostava i telefon izvan uzorka `+…` ruše XSD; porezni zastupnik nije podržan; OIB se čita iz `CompanySettings.vat_number`.

### A-14 (S) Devize

Porezni preračun postoji (`ExpenseFxConfirmation`, svrha `tax`; HNB pa ESB — `fx_rates/lookup.py:27`) i bez potvrde stavka traži pregled — usklađeno s Z čl. 36. Zadani ciljni datum je `expense_date` (`domains/finance/services/fx_confirmation.py:222`), ne porezni datum. Raniji tečaj nije dopušten (`config/settings.py:307`).

### A-15 (S) Kontrola prema PDV-u

`validate_v002_pdvs_total_not_exceed_pdv` (`pdv/business_validation.py:39`): samo dobra, ozbiljnost `fatal` (portal: upozorenje), nema kontrole usluga; poziva se samo iz testa. Radi nad živim podacima, ne nad predanim verzijama.

### A-16 (S) Izvoz i sigurnosni zahtjevi

Postojeća admin radnja „Export PDV-S (XLSX)” (`vat.py:647`) izvozi zbrojeve PDV knjiga, ne Obrazac PDV-S ni njegove izvore, i ne smije se ponovno upotrijebiti za D07 bez prerade. Uzor za siguran XLSX postoji u `domains/reporting/documents/export.py`. Za uvoz (D10) parser treba izričito konfigurirati bez razrješavanja entiteta i mrežnih dohvata (`pdv_s/parse.py:42` koristi zadani parser). PDF obrazac, PDF validacije, ZIP manifest: ne postoje.

### A-17 (S) Sučelje (`app`)

- `components/tax/PdvSPeriodList.tsx`: popis iz PDV razdoblja (bez mjeseci bez razdoblja), jedan HTTP poziv po razdoblju, greška se prikazuje kao prazan iznos, zbrajaju se sva razdoblja; nema CTA, rokova ni statusa obveze.
- `components/tax/PdvSWorkflow.tsx`: XML, UUID i datum predaje, jedna datoteka potvrde; nema drafta, validacije, pregleda izvora, razlika ni arhive.
- Uloge: `canWriteTax` (owner, accountant) odgovara API-ju.

### A-18 (N) Testovi, CI i dokumentacija

- Testovi: `accounting/tests/test_pdv_s_eu_services.py` (engine putanja; usluge, dobra, isključenje 612), `domains/tax/tests/test_pdv_workflow_api.py` (neovisnost, povijest predaja), `test_submission_events.py`, `test_pdv_business_validation.py`. Hash agregata dotiče `test_mark_vat_return_submitted.py`; nema XSD ni round-trip testova PDV-S niti fixtura `fixtures/tax/pdv_s/` po `TESTING_GUIDE.md`.
- U `api/.github/workflows/` postoji samo `openapi.yml`; workflow za PDV regresiju koji spominju upute ne nalazi se tamo.
- `FORM_REGISTRY.md` navodi `payload.json` kao kanon za PDV-S — ne postoji.
- `EPOREZNA_SUBMISSION_MATRIX.md` navodi PDV-S kao tromjesečni — netočno (mjesečni, R0-01 1.7). `Schema Version` je `TBD`, službeno `1.0`.

## 3. Pokrivenost ADR-0034

| Zahtjev | Status | Referenca / napomena |
|---|---|---|
| Neovisnost o PDV prijavi (predaja ne zaključava razdoblje) | **IMPLEMENTED** | `service.py:530`; `test_pdv_workflow_api.py` `test_pdv_s_xml_and_submit_without_attachment` |
| Kreiranje obrasca za odabrani mjesec bez postojećeg razdoblja (§4.1) | **MISSING** | `get_vat_period` vraća 404; nema endpointa ni CTA |
| Obuhvat svih EU dobara i usluga (§1) | **CONFLICT** | A-01, A-03, A-06 |
| Porezni datum i razdoblje | **MISSING** | A-05 |
| Jedinstveni klasifikacijski sustav | **CONFLICT** | dvije putanje (legacy i engine) s različitim rezultatom; ADR-0019 Proposed |
| D01 B automatski unos | **PARTIAL** | agregacija postoji; A-01–A-07 |
| D01 B dokumentirane ručne korekcije | **MISSING** | ručni redak (`VATLedgerEntry.is_manual`) ušao bi u agregat bez razloga, dokumenta i autora |
| Zaštita od dvostrukog uključivanja | **MISSING** | nema veze korekcija–izvor |
| D02 B evidencija UUID + datum | **IMPLEMENTED** | `submit.py`, `PdvSSubmitView` |
| D02 B naknadni PDF i XML, status dokumentiranosti | **PARTIAL / CONFLICT** | jedan prilog (A-10) |
| D03 C drill-down | **MISSING** | retci ledgera nemaju porezni datum, valutu, tečaj, snimku partnera |
| D04 B detekcija obveze, rokovi | **MISSING** | — |
| D05 B razlike i ručni ispravak | **PARTIAL** | postoji lanac predaja i „Nova predaja”; nema snimke ni diffa |
| D06 B validacija | **PARTIAL** | XSD pri preuzimanju XML-a |
| D06 B VIES | **MISSING** | u kodu nema klijenta; postoje samo pomoćne provjere oblika (`shared/vat`) |
| D07 C arhiva | **MISSING** | A-16 |
| D08 B zaključavanje verzije | **MISSING** | A-08 |
| D09 C rekoncilijacija | **MISSING** | A-15, A-02 |
| D10 B pojedinačni uvoz | **PARTIAL** | parser i CLI (A-11); nema API-ja, pregleda, provjere duplikata sadržaja |
| Pravne praznine za odobrenja, devizne dane | **BLOCKED** | Q-01, Q-02, Q-07 |
| Tenant izolacija i uloge (porezni API) | **IMPLEMENTED** | `get_vat_period(tenant, …)`; `domains/tax/api/permissions.py`; pisanje owner/accountant (`tenants/permissions.py:45`), čitanje i izvoz sve uloge |
| Tenant izolacija novih dokaza i priloga | **MISSING** | provjeriti pristup datotekama prije D02/D07 |
| Povijest `SubmissionEvent` (append-only) | **IMPLEMENTED** | `SubmissionEvent.save()` |
| Immutable predane verzije | **MISSING** | A-08 |

### Pregled komponenti

| Komponenta | Ocjena |
|---|---|
| `VATPeriod` | upotrebljiv kao mjesečni ključ; rasponi i `CASCADE` (A-12) |
| `VATLedgerEntry` | nedovoljno činjenica za PDV-S (A-05, A-07) |
| `PDVSReturn` | samo anchor — kako ADR predviđa |
| `VATReturn` | uzor za verzije i `payload_snapshot` (ADR-0007/0027) |
| `SubmissionEvent` / `SubmissionService` | ponovno upotrebljivi uz zaštitni sloj (A-10) |
| `aggregate_pdv_s_rows()` | zamijeniti ili bitno proširiti |
| renderer | strukturno usklađen sa shemom; deterministički artefakt treba spremati |
| parser | proširiti (zaglavlje, metapodaci, sigurna konfiguracija) |
| canonical hash | ovisi o redoslijedu redaka; verzionirati algoritam |
| VIES | ne postoji |
| PDF / XLSX / ZIP | ne postoji za PDV-S |
| zaključavanja | `VATPeriod` i fiskalna razdoblja postoje; PDV-S ih ne dira |
| RBAC / tenant | postoji na API-ju |

## 4. Ponašanje koda prema ključnim scenarijima (statička procjena)

| Scenarij | Očekivano | Kod danas |
|---|---|---|
| G02 stroj bez VIN-a | I1 | usluga ili izostaje (A-01) |
| G04 marža s VIN-om | isključeno | engine: uključuje se kao dobro |
| G06 novo vozilo s VIN-om | I1 | engine: I1; legacy: ovisi o temeljnici |
| G16 domaća naknada na 0373 | isključeno | box 207 / greška cijelog mjeseca (A-03) |
| U01–U04 EU usluge | I2 | engine: I2; legacy: trošak izostaje |
| U08 oslobođena usluga | isključeno | engine: uključuje se |
| U12 usluga XI | isključeno | engine: uključuje se |
| T04, T07 porezni datum | po obvezi | po datumu troška |
| N01 odobrenje | neto | bruto (A-04) |
| F03 prazna prva predaja | blokada | prihvaća se |
| F05 jedan redak po ID-u | da | da |
| V06 HR prefiks | blokada po stavci | greška XSD za cijeli obrazac |

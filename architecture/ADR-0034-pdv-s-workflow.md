# ADR-0034 — Samostalni PDV-S obrazac i životni ciklus predaje

Status: Proposed  
Date: 2026-11-27  
Type: Business  
Supersedes: —  
Related: [ADR-0007](ADR-0007-pdv-module.md), [ADR-0008](ADR-0008-submission-events.md), [ADR-0009](ADR-0009-submission-module-v1.md), [ADR-0019](ADR-0019-tax-classification-engine.md), [ADR-0023](ADR-0023-partner-tax-jurisdiction.md), [ADR-0027](ADR-0027-pdv-correction-lifecycle.md), [FORM_REGISTRY](../tax/FORM_REGISTRY.md).

## 1. Cilj i status

Proposed; nije odobrenje implementacije ili deploymenta. Korisnik mora moći kreirati PDV-S za odabrani mjesec bez prethodnog kreiranja ili predaje PDV prijave. Obuhvat je sva zakonski relevantna nabava robe i usluga iz EU, ne samo vozila. PDV prijava nastaje kasnije; usporedba nakon njezine predaje ne smije blokirati PDV-S.

## 2. Postojeće stanje

Read-only pregled develop: api 180dcb9, app b91a1c8, docs 95f3385 (ponovno provjeriti HEAD prije implementacije).
- app/src/components/tax/PdvSPeriodList.tsx učitava postojeće PDV periode i njihove agregate; nema CTA za kreiranje. PdvSWorkflow.tsx nudi XML, evidentiranje predaje, potvrdu i novu predaju; nema spremljeni draft.
- api/app/accounting/services/tax_forms/pdv_s/{aggregate,render,parse,validation,submit}.py: agregacija iz VATLedgerEntry box 207/210; preskače osnovice <=0. Provjeriti sve relevantne stope i režime. Legacy fallback preko opisa temeljnice i MANUAL_SUPPLIER_MAP je rizik identifikacije.
- GET /api/tax/pdv-s/periods/{period}/ i /xml/ računaju uživo; POST /submit/ evidentira predaju. PDVSReturn je anchor 1:1 po periodu. SubmissionEvent čuva hash tadašnjeg agregata, ne puni snapshot. XML mismatch je upozorenje.
- FORM_REGISTRY navodi payload.json kao kanon, što postojeći live workflow ne ostvaruje u potpunosti.
- Postoje testovi povijesti predaja i neovisnosti PDV perioda. Testovi nisu izvršeni; produkcijski UI nije provjeren.

## 3. R0 — regulatorni gate prije izmjene koda

Izvori:
- [Zakon o izmjenama Zakona o PDV-u NN 151/2025](https://narodne-novine.nn.hr/clanci/sluzbeni/2025_12_151_2262.html), izmjena čl. 86. st. 2. i prijelazne odredbe.
- [Pravilnik NN 11/2026](https://narodne-novine.nn.hr/clanci/sluzbeni/full/2026_01_11_90.html); za potpuna pravila koristiti važeći konsolidirani zakon i pravilnik.
- [ePorezna PDV-S korisničke upute](https://e-porezna.porezna-uprava.hr/Upute/G2B/Korisni%C4%8Dke%20upute%20-%20Obrazac%20PDV-S.pdf), str. 13–15, poruke 004–014 (provjereno 27.11.2026).
- [Službeni XSD paket](https://e-porezna.porezna-uprava.hr/Upute/G2B/ePorezna_Schemas.zip); postojeća lokalna shema api/app/accounting/schemas/pdv-s/v1-0/.

Potvrđeno: mjesečna prijava; od 2026. rok do kraja idućeg mjeseca (posebno provjeriti povijesna razdoblja, prijelaz i neradne dane). Važeći PDV ID podnositelja za razdoblje; jedan red po PDV ID-u dobavljača, barem jedna nenulta vrijednost. Prva prazna prijava nije dopuštena (005); nova prijava za isto razdoblje zamjenjuje prethodnu (008); prazna zamjenska prijava može poništiti prethodnu. Nedostatak knjiženog PDV obrasca je upozorenje (004), ne zabrana. Kontrole 013/014: PDV-S dobra uspoređuju se sa zbrojem osnovica II.5–II.7, usluge II.8–II.10 zadnjeg zaprimljenog PDV obrasca; iznos PDV-S može biti manji ili jednak. Interno objasniti razliku, ne nametati jednakost kao zakonski uvjet.

Lokalni XSD I1/I2 koristi tDecimal15-2 bez minInclusive: negativni iznosi tehnički su dopušteni. To nije dokaz pravilnog poreznog razdoblja odobrenja. R0 mora utvrditi nastanak porezne obveze za dobra/usluge, odobrenja, povrate, storno, negativni neto po dobavljaču, naknadno evidentirane račune, predujmove, oslobođene transakcije, maržu, nova prijevozna sredstva, XI/GB povijesne iznimke, tečaj ne-EUR računa. Ne koristiti datum unosa kao porezni datum. Provjeriti aktualnu službenu shemu i portalna pravila. Dokumentirati pravni izvor, datum, primjere i očekivane rezultate; ne mijenjati službeni XSD. Nejasni scenariji = BLOCKED, ne nagađati niti tiho odbacivati negativne osnovice.

## 4. Predložena odluka

### 4.1 Kreiranje i izvor

Ruta /t/{slug}/porezi/pdv-s ostaje. CTA Kreiraj PDV-S, godina i mjesec. Postojeći VATPeriod zajednički je porezni period; ako nedostaje, idempotentno kreirati kroz domenski servis. Ne kreirati VATReturn, ne predavati PDV, ne zatvarati period. PDV ledger zajednički izvor poreznih činjenica; izgraditi ako nedostaje uz postojeće guardove. Ne otvarati zaključani period niti prepisivati predani PDV. Ako aktualna projekcija zahtijeva izmjenu zaključane knjige, read-only kandidat kroz isti klasifikacijski servis kao zaseban gate, bez drugog poreznog enginea. Miješani račun dobra/usluge klasificirati po poreznoj činjenici. Nedostajući PDV ID ili klasifikacija = pregled, ne nagađanje iz države/opisa/kategorije. Uključiti obveznike s PDV ID-om izvan redovnog sustava PDV-a kada je primjenjivo.

### 4.2 Verzije i audit

Zadržati PDVSReturn anchor 1:1 i postojeći SubmissionEvent lanac na istom dokumentu. Predložiti podređeni PDVSReturnVersion (tehnički naziv nije konačan) s jedinstvenim brojem verzije, payload.json, hashom, schema/mapping verzijom, izvorima, datumom/autorom i XML artefaktom. Verziju točno povezati s eventom; ne mijenjati generic SubmissionEvent ni core servis bez zasebnog ADR-a. Pri evidentiranju predaje proslijediti hash zamrznutog snapshota kroz postojeći servis, ne izračunavati ponovno iz žive knjige. Sačuvati prepoznatljivost legacy hash algoritma (verzija algoritma; zaseban puni integrity hash po potrebi). Najviše jedan aktivni draft; atomic + lock + unique constraint, idempotentno kreiranje. Predana verzija immutable. Ispravak = novi cjeloviti snapshot, ne delta. Prazan zamjenski obrazac samo uz eksplicitnu potvrdu i dokaz prethodne predaje; ne briše povijest.

### 4.3 UI

Pregled obrazaca s CTA, filtrima godine/statusa, iznosima i rokovima. Detalj: zaglavlje, država/PDV ID/naziv dobavljača (naziv interni), dobra/usluge, zbrojevi. Klik na red otvara račune, odobrenja, porezni datum, osnovicu EUR i klasifikaciju. Greške po dokumentu, bez skrivenog preskakanja.

Draft → validacija → spremno za predaju (zamrznuti snapshot) → XML → ručna predaja ePoreznoj → evidencija portal UUID-a i datuma → potvrda → povijest. Uređivanje izvora kroz računovodstveni workflow pa eksplicitno osvježenje drafta. **Potvrđeno 2026-11-27 — opcija A:** PDV-S se popunjava isključivo automatski iz kanonske porezne evidencije. Nema ručnog dodavanja, izmjene ili brisanja redaka ni ručnih korekcija izravno u obrascu, uključujući API i admin. Nedostajući ili pogrešni dokumenti ispravljaju se u izvornom računovodstvenom workflowu, uz postojeća pravila knjiženja i zaključavanja, pa se draft eksplicitno osvježava. Prazan zamjenski obrazac dopušten je samo kad ga kanonski izvori opravdavaju i uz postojeće guardove. Prikaz Predano + ispravak u pripremi. Ako se izvori promijene, prikaz upozorenja i diff; nikada ne prikazivati aktualni agregat kao prethodno predani obrazac.

### 4.4 Dokazi predaje

Razlikovati evidentirano slanje, zaprimanje i potvrđenu obradu prema raspoloživim dokazima; bez izmišljanja statusa portala. Portal UUID nije XML Metapodaci/Identifikator. Predaja mora referencirati konkretan zamrznuti snapshot. PDF potvrda i XML imaju različitu dokaznu ulogu. Za nove verzije ne prihvatiti sadržajno neusklađeni XML kao dokaz istog snapshota; provjeriti OIB, cijelo razdoblje, tip i canonical hash, uz kompatibilnost s portalnim normalizacijama. Ne mijenjati zajednički SubmissionService bez odobrenog odstupanja; PDV-S specifični guard na ulazu. Predloženi default: UUID i datum omogućuju ručnu evidenciju, nedostajuća potvrda ostaje jasno označena; vlasnik može odabrati strože pravilo.

### 4.5 Naknadna usklađenost s PDV

Prije PDV predaje status: PDV još nije predan / usporedba nije dostupna. Nakon predaje usporediti konkretne predane verzije (PDV-S I1/I2 prema relevantnim osnovicama II.5–7 i II.8–10); prikaz iznosa, razlike, verzije, datuma i statusa. Odvojiti portalnu kontrolu <= od interne potpune rekoncilijacije; razliku obrazložiti. Usporedba read-only; ne mijenja nijedan obrazac, ne podnosi ispravak automatski. Ne čekati PDV za kreiranje ili predaju PDV-S.

### 4.6 Povijesni podaci

Legacy evente i hashove ostaviti netaknute. Ako postoji valjan arhivirani XML, rekonstruirati snapshot samo uz dokaz podudarnosti i označenu provenijenciju; inače označiti povijesni sadržaj nedostupnim. Ne rekonstruirati staru predaju iz današnjeg ledgera. Migracija additive, dry-run/report, tenant izolacija, bez produkcijskog upisa bez posebnog GO.

## 5. Implementacijski plan (nakon Accept/GO)

| Gate | Opseg | Kriterij |
|---|---|---|
| R0 | Regulatorna analiza | Izvori i test matrica za sve scenarije iz §3 |
| PDVS-01 | Kreiranje perioda/obrasca | Kreiranje bez VATReturn; idempotentno; RBAC/tenant |
| PDVS-02 | Snapshot i migracija | Immutable verzije, legacy evidence očuvan |
| PDVS-03 | Agregacija i provjera izvora | Sva relevantna dobra/usluge, sve stope, negativni iznosi prema R0 |
| PDVS-04 | API/OpenAPI | Draft, refresh, validate, XML, submit, history; očuvati postojeći contract ili verzionirati uz odluku |
| PDVS-05 | App | CTA, detalj, drill-down, greške, status, ispravci |
| PDVS-06 | Evidencija predaje | Hash zamrznute verzije, portal UUID, potvrda, audit |
| PDVS-07 | Cross-form kontrola | PDV-S radi prije PDV; naknadna usporedba read-only |
| PDVS-08 | Regression/CI | Lokalni CI, GitHub Actions na kontrolnoj točki, browser smoke; bez deploya bez GO |

Acceptance: ručni write pokušaji nad agregiranim retcima odbijeni u UI/API/admin; ispravak izvora i osvježenje drafta; prvi obrazac bez PDV; postojeći period bez knjige; zaključani period bez mutacije; roba, usluge i miješani račun; više računa istog VAT ID-a; različite stope; VAT-ID-only obveznik; odobrenje/storno/late invoice prema R0; nedostajući ID; prazna prva prijava blokirana, prazna zamjenska dopuštena uz guard; XSD round-trip; dvostruki klik i concurrency; izmjena ledgera nakon exporta/predaje ne mijenja snapshot/hash; ispravak v2 čuva v1; neusklađeni XML; cross-tenant/RBAC; usporedba nakon PDV predaje i nakon PDV ispravka. Koristiti sintetičke podatke u javnom repozitoriju, bez stvarnih OIB-a, VIN-a, računa ili tajni. Django test na serveru s --keepdb.

## 6. Granice i posljedice

Nema G2B automatizacije, novog poreznog enginea, izmjene PDV prijave, automatskog knjiženja radi kreiranja PDV-S, ručnog uređivanja XML/payload.json ili promjene produkcije. Prednost: samostalni workflow i ponovljiv audit. Trošak: nova persistence verzija, backfill analiza i API/UI proširenje. R0 ostaje otvoren za detaljnu pravnu matricu; implementacija ne smije početi prije njegova zatvaranja i odobrenja ADR-a. Otvorene produktne odluke: strogoća obvezne potvrde (preporuka označiti nedostajuću), prikaz internih izvora. Status maturity ne povisivati prije stvarne isporuke.

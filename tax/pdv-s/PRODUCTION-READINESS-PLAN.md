# PDV-S — program produkcijske spremnosti (PDVS-PRODUCTION-01)

```text
Status: Living document
Zadnja izmjena: 2026-11-27
Cilj: produkcijski spreman PDV-S modul prema ADR-0034
ADR-0034: Proposed
R0: PARTIAL
Aktivna kontrolna točka: G01 (zaustavljeno, čeka odluke vlasnika)
Radna grana: develop (api, app, docs)
```

Povezano: [ADR-0034](../../architecture/ADR-0034-pdv-s-workflow.md) · [R0 završni izvještaj](R0-FINAL-REPORT.md) · [regulatorna analiza](R0-REGULATORY-ANALYSIS.md) · [matrica scenarija](R0-SCENARIO-MATRIX.md) · [audit implementacije](R0-EXISTING-IMPLEMENTATION-AUDIT.md) · [arhitekturna revizija](R0-ARCHITECTURE-REVIEW.md) · [FORM_REGISTRY](../FORM_REGISTRY.md)

Repozitorij je javan. Stvarni poslovni podaci, produkcijski detalji i sigurnosni nalazi vode se u privatnim izvještajima izvan Gita i ovdje se ne navode.

## 1. Pravila programa

1. Gateovi se izvršavaju redom. Ne radi se na svim fazama istodobno.
2. Svaki gate završava kontrolnom točkom: rezultat se dokumentira ovdje; sljedeći gate traži navedenu autorizaciju.
3. Materijalni bloker zaustavlja program; ne zaobilazi se pretpostavkom.
4. Poslovne odluke D01–D10 iz ADR-0034 potvrđene su i ne mijenjaju se.
5. Autorizacija za jedan gate ne prenosi se na sljedeći, na produkciju niti na `main`.

## 2. PDVS-HOTFIX-01

**PDVS-HOTFIX-01: CLOSED / NO FILING REQUIRED** (2026-11-27)

PDV-S za listopad 2026. ne predaje se. Temelj su audit izvora i potvrde vlasnika projekta; dokazi su u privatnom izvještaju. Nije kreiran prazan obrazac, nije evidentirana predaja i nije mijenjano porezno razdoblje.

Zaključak vrijedi samo za provjereno razdoblje. Do otklanjanja nalaza A-01 do A-05 iz [audita](R0-EXISTING-IMPLEMENTATION-AUDIT.md) XML iz postojećeg toka ne predavati bez ručne usporedbe sa svim EU računima i odobrenjima; to vrijedi i za studeni 2026. (rok 31.12.2026.).

## 3. Pregled gateova

| Gate | Opseg | Ovisi o | Status | Commit SHA | Autorizacija |
|---|---|---|---|---|---|
| G00 | Preflight i zatvaranje listopada | — | **DONE**, otvorene stavke prenesene | `b45dbbc` (plan) | dana 2026-11-27 |
| G01 | SEC-AUDIT-01 | G00 | **AUDIT DONE**, gate otvoren | `b45dbbc` (plan) | audit dan; sanacija nije |
| G02 | TAX-CORE-01 | G01 | PREPARED, nije pokrenut | — | potrebna |
| G03 | TAX-CORE-02 | G02 | NOT STARTED | — | potrebna |
| G04 | Regulatorni R0 closure | G03; odgovori na pravna pitanja | NOT STARTED, blokiran Q-01/02/07 | — | potrebna |
| G05 | Prihvaćanje arhitekture ADR-0034 | G04 | NOT STARTED | — | potrebna |
| G06 | Backend i persistence | G05 | NOT STARTED | — | potrebna |
| G07 | Frontend | G06 | NOT STARTED | — | potrebna |
| G08 | Validacija, VIES, export, import | G06, G07 | NOT STARTED | — | potrebna |
| G09 | Rekoncilijacija | G02, G08 | NOT STARTED | — | potrebna |
| G10 | CI, regresija, stage | G06–G09 | NOT STARTED | — | potrebna |
| G11 | Produkcijska spremnost i deployment gate | G10 | NOT STARTED | — | zaseban izričit GO |

Mapiranje na plan u ADR-0034 §5 (plan se ne mijenja): R0 → G04; PDVS-01 do PDVS-04 i PDVS-06 → G06; PDVS-05 → G07; PDVS-07 → G09; PDVS-08 → G10.

## 4. Produkcijski standard (svi implementacijski gateovi)

| Zahtjev | Dokaz za izlaz iz gatea |
|---|---|
| Tenant izolacija | negativni testovi preko granice tenanta za svaki novi ulaz (API, admin, datoteke) |
| RBAC | matrica uloga × radnja, uključujući anonimnog korisnika |
| Transakcijska sigurnost | atomarnost i testovi konkurentnosti |
| Idempotentnost | ponovljeni zahtjev ne stvara duplikat |
| Immutable predani snapshoti | pokušaj izmjene odbijen; hash stabilan nakon promjene izvora |
| Audit trail | autor, vrijeme, razlog; povijest se ne briše |
| Kompatibilnost API ugovora | postojeći contract testovi; promjena ugovora uz odluku |
| OpenAPI i TypeScript | regeneracija bez razlike; typecheck |
| Testovi | unit, integration, migration, security |
| Browser smoke i stage | zabilježen rezultat |
| Rollback | isproban na kopiji, uključujući bazu |

Django testovi na serveru uz `--keepdb`, kroz Docker, na izoliranoj bazi. Javni fixturei samo sintetički. Razvoj u zasebnim Git worktreeima, ne u checkoutu koji koristi produkcija.

## 5. Gateovi

### G00 — Preflight i zatvaranje listopada

| | |
|---|---|
| Opseg | Commitovi, verzije, Git status, migracije, necommitane izmjene, backupovi, lokalni CI, GitHub Actions, deploy i rollback; zatvaranje PDVS-HOTFIX-01 |
| Ovisnosti | — |
| Kriteriji | Svaka stavka provjerena ili izričito označena nepotvrđenom |
| Testovi | Provjera migracija i modela bez upisa; provjera integriteta postojećeg dumpa |
| Rizici | Stanje se mijenja nakon preflighta — ponoviti prije G11 |
| Rollback | Nije primjenjivo (samo čitanje) |
| Status | DONE, 2026-11-27 |
| Commit SHA | plan: `b45dbbc`; stanje koda: api `main` `c4902ae` / `develop` `180dcb9`; app `main` `e6e9b91` / `develop` `b91a1c8`; docs `develop` `8cd1f3f` (prije ovog dokumenta); intermediary i www `ff7288a`, `08d9922` |
| Autorizacija | dana |

Zabilježeno: migracije primijenjene do `accounting.0045`, `expenses.0028`, bez razlike modela i migracija. GitHub Actions izvršavaju samo provjere OpenAPI sheme i TypeScript tipova; zadnja izvršavanja na navedenim commitovima uspješna. PDV regresijski workflow koji spominju upute ne nalazi se u repozitoriju koji ga može pokrenuti. Grane `docs/main` i `docs/develop` razišle su se.

Otvorene stavke prenesene dalje:

| ID | Stavka | Rješava se |
|---|---|---|
| WIP-01 | Necommitane izmjene u produkcijskom checkoutu `api`; ne uklanjati, ne prepisivati, ne commitati automatski; vlasnik odlučuje | zaseban zadatak, prije prvog deploya |
| OPS-01 | Sigurnosne kopije: raspored, obuhvat i probni povrat za bazu i datoteke | G11 (preporuka ranije) |
| OPS-02 | Dostupnost i verzije stage okruženja | G10 |
| OPS-03 | Testovi u CI, uključujući PDV regresiju | G10 (preporuka prije G02) |
| OPS-04 | Ponovljiv deploy i rollback; verzionirana konfiguracija; oznaka commita u slikama | G11 |
| DOC-01 | Dva ADR dokumenta s brojem 0033 | G05 |

### G01 — SEC-AUDIT-01

| | |
|---|---|
| Opseg | SEC-01 pristup datotekama; SEC-02 admin, RBAC i tenant izolacija; SEC-03 sigurnost izvoza; SEC-04 sadržaj javnih repozitorija; SEC-05 plan sanacije |
| Ovisnosti | G00 |
| Kriteriji audita | Svaki nalaz ima ozbiljnost, dokaz, komponentu, najmanju korekciju, regresijski test, utjecaj na korisnike, deploy i rollback; potvrđeni i nepotvrđeni odvojeni |
| Kriteriji zatvaranja gatea | Sanacija odobrena, isporučena i provjerena regresijskim testovima, ili vlasnik za pojedini nalaz pisano prihvati rizik |
| Testovi | Negativni testovi autorizacije i izolacije; testovi izvoza i uploada |
| Rizici | Sanacija može privremeno ograničiti postojeće radnje korisnika |
| Rollback | Po valu sanacije, definiran u privatnom planu |
| Status | AUDIT DONE 2026-11-27; gate **otvoren** |
| Commit SHA | audit nad api `c4902ae`; plan: `b45dbbc` |
| Autorizacija | read-only audit dan; izmjene koda, konfiguracije, produkcije i Git povijesti nisu |

Rezultati su privatni. Javni opis pojedinih nalaza objavljuje se tek nakon sanacije.

### G02 — TAX-CORE-01: službena polja Obrasca PDV

Pripremljeno za kontrolnu točku; ne izvršava se usporedno s G01.

| | |
|---|---|
| Opseg | Usporedba svih službenih polja s aktualnim registrom; sve stope PDV-a; tuzemni prijenos porezne obveze; EU stjecanja dobara; EU primljene usluge; ostale relevantne kategorije; utjecaj na PDV-S i na predane PDV prijave |
| Ovisnosti | G01; zaseban zadatak pod [ADR-0007](../../architecture/ADR-0007-pdv-module.md) (arhitekturna revizija K-2) |
| Redoslijed | (1) matrica polje → službeni opis → izvor → trenutno mapiranje → status; (2) read-only provjera utjecaja na predane prijave; (3) prijedlog ispravka; (4) implementacija uz zaseban GO |
| Kriteriji | Svako polje ima službeni izvor i odluku; popis pogođenih predanih prijava u privatnom izvještaju; `PDV_MAPPING_VERSION` i `pdv-mapping.md` usklađeni |
| Testovi | Snapshot testovi PDV-a, contract test arhitekturnog gatea, ledger diff, regresija referentnih razdoblja |
| Rizici | Promjena mapiranja mijenja payload i XML; moguća potreba za ispravkom predanih prijava |
| Rollback | Vraćanje verzije mapiranja; predane verzije ostaju netaknute |
| Status | PREPARED |
| Commit SHA | — |
| Autorizacija | potrebna za analizu; posebna za izmjenu koda |

Povijesne prijave pregledavaju se samo čitanjem. Ne ispravljaju se automatski.

### G03 — TAX-CORE-02: porezne činjenice i jedinstvena klasifikacija

| | |
|---|---|
| Opseg | Dopuna [ADR-0019](../../architecture/ADR-0019-tax-classification-engine.md) prije PDV-S klasifikacije: porezni datum, vrsta isporuke, posebni režimi, snimka dobavljača, odobrenja; jedna klasifikacijska putanja (K-1, K-7); dokumentirane ručne korekcije kao sljedive porezne činjenice (K-3); `country_code` iz ADR-0023 |
| Ovisnosti | G02 |
| Kriteriji | Architecture Review dopune; plan migracije i cutovera; svaka korekcija ima dokument, razlog, autora i vezu na kasniji izvor |
| Testovi | Scenariji R0 matrice sa statusom V i D; za status O samo blokada `REVIEW`; ledger diff; migracijski testovi |
| Rizici | Drugi porezni engine ako se pravila ugrade u PDV-S; pretpostavke o odobrenjima prije G04 |
| Rollback | Shadow način uz postojeću projekciju; additive migracije |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

Dijelovi koji ovise o Q-01 i Q-07 ostaju blokirajući do G04.

### G04 — Regulatorni R0 closure

| | |
|---|---|
| Opseg | Pisana potvrda Q-01 (razdoblje odobrenja), Q-02 (negativni iznosi na portalu), Q-07 (tečaj bez tečajne liste, odobrenja, zaokruživanje); po mogućnosti Q-03 do Q-11; ponovna provjera izvora i XSD-a |
| Ovisnosti | G03; odgovor poreznog savjetnika ili Porezne uprave |
| Kriteriji | R0 = PASS, ili vlasnik izričito prihvati da navedeni scenariji ostaju blokirani u prvoj verziji |
| Testovi | Scenariji O dobivaju očekivane vrijednosti samo uz izvor |
| Rizici | Dugo čekanje odgovora; promjena propisa |
| Rollback | Dokumentacijski |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna; preporuka: upit poslati odmah jer ne mijenja kod |

### G05 — Prihvaćanje arhitekture ADR-0034

| | |
|---|---|
| Opseg | Architecture Review ADR-0034; model `PDVSReturnVersion`; zasebni PDV-S prilozi predaje uz frozen `SubmissionService` (K-4); blokada uvoza starije predaje kad postoji novija, uz računovodstveni pregled (K-5); povijesna razdoblja u kunama inicijalno samo uvoz i prikaz (K-6); samo mjesečno razdoblje i `PROTECT` (K-9); verzionirani hash (K-10); kandidat za zaključana razdoblja (K-8); RFC za VIES connector; renumeracija jednog ADR-0033 bez gubitka sadržaja i povijesti i usklađivanje grana `docs` |
| Ovisnosti | G04 |
| Kriteriji | ADR-0034 Accepted; svaka stavka K ima odluku; ADR_INDEX bez duplog broja |
| Testovi | Provjera poveznica nakon renumeracije, uključujući reference iz koda |
| Rizici | Renumeracija prekida reference |
| Rollback | Revert dokumentacijskih commitova |
| Status | NOT STARTED — ADR-0034 ostaje Proposed |
| Commit SHA | — |
| Autorizacija | potrebna |

### G06 — Backend i persistence

| | |
|---|---|
| Opseg | Idempotentno kreiranje razdoblja i obrasca; verzije i snapshot; agregacija iz kanonskog izvora s greškama po dokumentu; korekcije; detekcija obveze i rokovi (D04); diff (D05); zaključavanje (D08); evidencija predaje i prilozi (D02); API i OpenAPI |
| Ovisnosti | G05; zatvoren G01 |
| Kriteriji | Kriteriji ADR-0034 §5 za backend; legacy eventi i hashevi netaknuti |
| Testovi | Unit, integration, migracijski (naprijed i natrag na kopiji), concurrency, tenant/RBAC, contract |
| Rizici | Migracije se primjenjuju pri startu servisa; neobavezni novi stupci i zadane vrijednosti moraju podnijeti povratak starog koda |
| Rollback | Additive migracije; isprobani postupak s bazom prije deploya |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

### G07 — Frontend

| | |
|---|---|
| Opseg | Godišnji pregled 12 mjeseci, CTA, detalj, drill-down (D03), statusi, razlike, ispravci |
| Ovisnosti | G06 |
| Kriteriji | Povijesna predaja prikazuje snapshot; nedostupni podaci označeni; uloge poštovane i na serveru |
| Testovi | Komponentni testovi, typecheck, generirani tipovi, browser smoke |
| Rizici | Prikaz živog agregata kao predanog; N+1 pozivi |
| Rollback | Prethodna slika aplikacije; API kompatibilan unatrag |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

### G08 — Validacija, VIES, export i import

| | |
|---|---|
| Opseg | D06 validacija i VIES; D07 arhiva (XML, PDF, XLSX, ZIP s manifestom); D10 pojedinačni uvoz |
| Ovisnosti | G06, G07 |
| Kriteriji | Poslovna pravila izvan XSD-a provedena; export samo iz snapshota; uvoz atomaran i siguran |
| Testovi | VIES valid/invalid/unavailable; sigurnost XML parsera; formula injection; sigurnost priloga; duplikati i konflikti |
| Rizici | Nedostupnost VIES-a; nesiguran sadržaj datoteka |
| Rollback | Isključivanje novih ulaza bez brisanja evidencije |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

### G09 — Rekoncilijacija

| | |
|---|---|
| Opseg | D09: portalna kontrola i interna rekoncilijacija nad predanim verzijama; obrazloženja razlika |
| Ovisnosti | G02 (ispravno mapiranje), G08 |
| Kriteriji | Dva odvojena statusa; bez mutacije obrazaca; bez blokade zbog nepredanog PDV-a |
| Testovi | Scenariji F10–F15, ispravak bilo kojeg obrasca, idempotentno osvježavanje |
| Rizici | Usporedba pogrešnih polja |
| Rollback | Rezultati kontrole su dodatni zapisi; isključivanje okidanja |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

### G10 — CI, regresija i stage

| | |
|---|---|
| Opseg | Testovi u CI-u i lokalno, uključujući PDV regresiju; stage s istim commitovima; browser smoke; proba migracija i rollbacka |
| Ovisnosti | G06–G09 |
| Kriteriji | Zeleni CI na točnim commitovima; stage verzije zabilježene; svi kriteriji ADR-0034 §5 pokriveni |
| Testovi | Cijeli paket |
| Rizici | Stage se razlikuje od produkcije |
| Rollback | Stage reset |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | potrebna |

### G11 — Produkcijska spremnost i deployment gate

| | |
|---|---|
| Opseg | Ponovljeni preflight; sigurnosne kopije baze i datoteka s probnim povratom; WIP-01 riješen; runbook; kriteriji prekida; provjera nakon deploya; backfill samo uz dry-run i poseban GO |
| Ovisnosti | G10; zatvoren G01 |
| Kriteriji | Checklist potpisan od vlasnika; rollback isproban |
| Testovi | Smoke i regresija nakon deploya bez izmjene poslovnih podataka |
| Rizici | Deploy mijenja živi checkout; automatske migracije |
| Rollback | Runbook s bazom, kodom i slikama |
| Status | NOT STARTED |
| Commit SHA | — |
| Autorizacija | zaseban izričit GO za merge `develop` → `main` i za deploy |

## 6. Trenutne granice autorizacije (2026-11-27)

Dopušteno: G00, ovaj plan, read-only SEC-AUDIT-01, dokumentacija i planovi sanacije, commit i push dokumentacije na `develop`.

Nije dopušteno: izmjene produkcijskog aplikacijskog koda, produkcijske migracije, promjene zaključanih razdoblja, izmjene predanih obrazaca, prepisivanje Git povijesti, produkcijski deployment, merge `develop` → `main`.

## 7. Dnevnik

| Datum | Događaj | Commit |
|---|---|---|
| 2026-11-27 | G00 izvršen; PDVS-HOTFIX-01 zatvoren; G01 audit izvršen; plan uspostavljen; zaustavljeno na kontrolnoj točki G01 | `b45dbbc` |

Pri svakoj izmjeni ažurirati status, commit SHA i dnevnik.
